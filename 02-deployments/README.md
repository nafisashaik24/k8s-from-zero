# 02 - Deployments

## Why does this exist?
In [Lesson 01](../01-pods/), when a Pod was deleted, it was gone forever. Nobody brought it back. Real apps can't work like that: Pods crash, servers fail, and new versions need to be released.

A **Deployment** solves this. You tell it *"keep 3 copies of this Pod running"*, and Kubernetes makes sure that stays true — replacing Pods that die, scaling up or down, and upgrading to new versions without downtime.

## Analogy
A Deployment is like a **store manager** who must keep 3 cashiers working at all times. If one cashier leaves, the manager immediately brings in a replacement. If the store gets busy, the manager adds more cashiers. When training changes, the manager swaps cashiers one at a time so the store never closes.

## Key concepts
- You write the **Deployment**; Kubernetes creates a **ReplicaSet**, which creates the **Pods**
- **`replicas`** is your *desired state* — the number of Pods that should be running
- The Deployment finds its Pods **by label**, not by name
- Changing the Pod template triggers a **rolling update** with zero downtime
- Old ReplicaSets are kept (at 0 Pods) so you can **roll back** instantly

```
Deployment  (you create this)
   │ creates and manages
   ▼
ReplicaSet  (keeps the Pod count correct)
   │ creates
   ▼
Pod   Pod   Pod
```

## The YAML, line by line
[`deployment.yaml`](./deployment.yaml):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
  labels:
    app: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: nginx
          image: nginx:1.28
          ports:
            - containerPort: 80
```

The file has **two layers**, which is why `metadata` and `spec` each appear twice:

```
Deployment (manager)
├── metadata  → who the Deployment is
└── spec      → what the Deployment should do
    └── template → blueprint for each Pod
        ├── metadata → labels stamped on each Pod
        └── spec     → what each Pod runs
```

| Line | Meaning | Required? |
|---|---|---|
| `apiVersion: apps/v1` | Deployments live in the `apps` API group (Pods use core `v1`) | Yes |
| `kind: Deployment` | The type of object | Yes |
| `metadata.name` | The Deployment's name. Also the prefix for its ReplicaSet and Pod names | Yes |
| `metadata.labels` | Tags on the Deployment itself (for organizing) | No |
| `spec.replicas` | How many Pods to keep running | No (defaults to 1) |
| `spec.selector.matchLabels` | **Which Pods this Deployment owns** — every Pod with `app: web` | Yes |
| `spec.template` | The blueprint for each Pod. It's the Pod from Lesson 01, minus `apiVersion` and `kind` | Yes |
| `template.metadata.labels` | Labels stamped on every Pod. **Must match the selector**, or Kubernetes rejects the file | Yes |
| `template.spec.containers` | What runs inside each Pod | Yes |

**Why must the labels match?** The Deployment *creates* Pods labeled `app: web`, then *looks for* Pods labeled `app: web` to count them. If they didn't match, it would never recognize its own Pods and would keep creating more forever.

**Why no `name` in the template?** With 3 copies, they can't share one name, so Kubernetes generates a unique name for each Pod.

## Hands-on

### 1. Create the Deployment
```bash
kubectl apply -f deployment.yaml
kubectl get deployments
kubectl get replicasets
kubectl get pods
```

Expected:
```
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
web-deployment   3/3     3            3           28s

NAME                        DESIRED   CURRENT   READY   AGE
web-deployment-557577df75   3         3         3       36s

NAME                              READY   STATUS    RESTARTS   AGE
web-deployment-557577df75-glsdc   1/1     Running   0          43s
web-deployment-557577df75-qbjvc   1/1     Running   0          43s
web-deployment-557577df75-r79zp   1/1     Running   0          43s
```

**The names form a chain**, so you can always tell who created what:
```
web-deployment                      ← Deployment (the name you chose)
web-deployment-557577df75           ← ReplicaSet (+ hash of the Pod template)
web-deployment-557577df75-glsdc     ← Pod (+ random ID)
```

### 2. Self-healing
Open two terminals side by side. In one, watch Pods live:
```bash
kubectl get pods -w
```
In the other, delete a Pod (use a name from your own output):
```bash
kubectl delete pod web-deployment-557577df75-glsdc
```

Result:
```
web-deployment-557577df75-qbjvc   Running   2m30s   ← original
web-deployment-557577df75-r79zp   Running   2m30s   ← original
web-deployment-557577df75-rpqg5   Running   8s      ← automatic replacement
```
The ReplicaSet saw *"desired 3, current 2"* and created a replacement within seconds.

### 3. Scaling
```bash
kubectl scale deployment web-deployment --replicas=5
kubectl get pods
kubectl scale deployment web-deployment --replicas=3
kubectl get pods
```
When scaling down, Kubernetes removes the **newest** Pods first, keeping the ones that have proven stable.

> **Note:** `kubectl scale` changes the cluster directly, but the YAML still says `replicas: 3`. In real teams, the **YAML file is the source of truth** — edit the file and re-apply instead.

### 4. Rolling update
Change the image in `deployment.yaml` from `nginx:1.27` to `nginx:1.28`, then:
```bash
kubectl apply -f deployment.yaml
kubectl rollout status deployment web-deployment
kubectl get replicasets
```

Expected:
```
NAME                        DESIRED   CURRENT   READY   AGE
web-deployment-557577df75   0         0         0       7m17s   ← old (nginx:1.27)
web-deployment-6d8567df8    3         3         3       107s    ← new (nginx:1.28)
```

What happened:
- Changing the image changed the template, so its hash changed (`557577df75` → `6d8567df8`)
- Kubernetes created a **new ReplicaSet** and replaced Pods **one at a time** — notice the different Pod ages (101s, 93s, 93s)
- Some Pods were always running, so there was **zero downtime**
- The old ReplicaSet was kept at 0 for rollbacks

### 5. Rollback
```bash
kubectl rollout history deployment web-deployment
kubectl rollout undo deployment web-deployment
kubectl get replicasets
kubectl describe deployment web-deployment | grep Image
```

Expected:
```
NAME                        DESIRED   CURRENT   READY   AGE
web-deployment-557577df75   3         3         3       9m11s   ← old one brought back
web-deployment-6d8567df8    0         0         0       3m41s

    Image:         nginx:1.27
```
Kubernetes simply scaled the old ReplicaSet back up, which is why rollbacks are fast.

> **Drift warning:** after `rollout undo`, the cluster runs `1.27` but the YAML still says `1.28`. When the file and the cluster don't match, it's called **drift**. In real teams you would revert the change in Git instead. To resync here: `kubectl apply -f deployment.yaml`.

## Break it on purpose
Make the template label different from the selector:
```yaml
  template:
    metadata:
      labels:
        app: website
```
Then:
```bash
kubectl apply -f deployment.yaml
```
Kubernetes rejects it with an error saying the selector does not match the template labels. Change it back to `app: web`.

## Common errors

| Error | Cause | Fix |
|---|---|---|
| `selector does not match template labels` | `selector.matchLabels` and `template.metadata.labels` differ | Make them identical |
| `no matches for kind "Deployment" in version "v1"` | Used `apiVersion: v1` | Use `apiVersion: apps/v1` |
| `field is immutable` on the selector | Tried to change the selector of an existing Deployment | Delete the Deployment and re-create it |
| Pods stuck in `ImagePullBackOff` after an update | Wrong image name or tag | Fix the tag and re-apply, or `kubectl rollout undo` |

## Cleanup
```bash
kubectl delete -f deployment.yaml
```
This deletes the Deployment **and** its ReplicaSets and Pods.

## Key takeaways
- Never run bare Pods in real apps — use a Deployment
- Deployment → ReplicaSet → Pods
- **Labels** connect the Deployment to its Pods
- Deleted or crashed Pods are replaced automatically (**self-healing**)
- Changing the template triggers a **rolling update** with zero downtime
- `kubectl rollout undo` rolls back instantly using the old ReplicaSet
- Keep the **YAML file as the source of truth** to avoid drift

## Next
👉 03 - Services: how to give these changing Pods one stable address