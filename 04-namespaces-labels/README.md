# 04 - Namespaces & Labels

## Why does this exist?
So far, everything lived in one place (`default`). Real clusters are shared by many teams and environments, and that quickly becomes a mess.

Imagine a food delivery company with one cluster:
- The **orders team** runs `order-service`, the **payments team** runs `payment-service`
- Each team wants a **dev** copy for testing and a **prod** copy for real customers

Without organization, names clash (two teams can't both have a `web-service`), it's easy to delete the wrong thing, and nobody can tell who owns what.

Kubernetes gives you two tools:
- **Namespaces** split one cluster into separate spaces
- **Labels** tag objects so you can find and group them in many ways

## Analogy
- A **namespace** is like an **apartment number** in a building. Every apartment can have a "kitchen" without confusion, because the apartment number tells them apart.
- **Labels** are like **hashtags on a photo**. One photo can be `#vacation #beach #family`, and you can search by any of them.

## Key concepts
- A namespace is a **folder inside the cluster**. The same name can exist in different namespaces
- Without `-n`, `kubectl` always works in your current namespace (usually `default`)
- Apps call Services in **their own namespace** by short name, and in **another namespace** by adding `.namespace`
- Namespaces separate **names**, not network traffic: dev can still reach prod unless you add NetworkPolicies (later lesson)
- Labels are `key: value` tags. One object can have many
- Deployments and Services find Pods **by label**, so changing a label changes who manages a Pod and who sends it traffic

```
Cluster
├── namespace: orders-dev    → web-deployment, web-service
├── namespace: orders-prod   → web-deployment, web-service   ← same names, no clash
└── namespace: kube-system   → Kubernetes' own components
```

## The namespaces you already have

```bash
kubectl get namespaces
```

```
NAME                 STATUS   AGE
default              Active   26h
kube-node-lease      Active   26h
kube-public          Active   26h
kube-system          Active   26h
local-path-storage   Active   26h
```

| Namespace | What it's for |
|---|---|
| `default` | Where things go when you don't name a namespace (Lessons 01–03 lived here) |
| `kube-system` | Kubernetes' own components. Don't touch it |
| `kube-public` | Info readable by anyone. Mostly empty |
| `kube-node-lease` | Nodes send "I'm alive" heartbeats here |
| `local-path-storage` | Added by kind, provides disk storage (Storage lesson) |

Look inside `kube-system`:
```bash
kubectl get pods -n kube-system
```

```
NAME                                             READY   STATUS    RESTARTS       AGE
coredns-559f6c778d-jwnb2                         1/1     Running   1 (3m9s ago)   26h
coredns-559f6c778d-wptsb                         1/1     Running   1 (3m9s ago)   26h
etcd-learning-control-plane                      1/1     Running   1 (3m9s ago)   26h
kindnet-2lqmf                                    1/1     Running   1 (3m9s ago)   26h
kube-apiserver-learning-control-plane            1/1     Running   1 (3m9s ago)   26h
kube-controller-manager-learning-control-plane   1/1     Running   1 (3m9s ago)   26h
kube-proxy-76469                                 1/1     Running   1 (3m9s ago)   26h
kube-scheduler-learning-control-plane            1/1     Running   1 (3m9s ago)   26h
```

The control plane from the [concepts guide](../docs/concepts.md) is right here, running as ordinary Pods:

| Pod | Role | Where you saw it |
|---|---|---|
| `kube-apiserver` | The front door for every `kubectl` command | Concepts guide |
| `etcd` | The database storing everything you create | Concepts guide |
| `kube-scheduler` | Picks a node for each Pod | `describe` Events in Lesson 01 |
| `kube-controller-manager` | Runs the ReplicaSet logic that replaced deleted Pods | Self-healing in Lesson 02 |
| `coredns` | The cluster's DNS: turns `web-service` into an IP | `curl web-service` in Lesson 03 |
| `kube-proxy` | Makes Service IPs forward traffic to Pods | Load balancing in Lesson 03 |
| `kindnet` | Networking that gives each Pod its own IP | Every Pod IP so far |

`RESTARTS: 1` appears because Docker Desktop was quit and reopened: the whole cluster stopped and came back on its own.

## The YAML, line by line

### [`namespaces.yaml`](./namespaces.yaml)
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: orders-dev
---
apiVersion: v1
kind: Namespace
metadata:
  name: orders-prod
```

| Line | Meaning |
|---|---|
| `kind: Namespace` | The simplest object in Kubernetes: no `spec`, just a name |
| `name: orders-dev` | Lowercase letters, numbers, and dashes only |
| `---` | **Separates objects in one file.** One YAML file can hold many objects |

### [`app.yaml`](./app.yaml)
A Deployment and a Service in one file:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
        team: orders
        tier: frontend
    spec:
      containers:
        - name: nginx
          image: nginx:1.28
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 80
```

| What | Why |
|---|---|
| **No `namespace` in the file** | The namespace is chosen when applying (`-n`), so the same file works for dev and prod |
| `team: orders`, `tier: frontend` | Extra labels for finding and grouping Pods |
| `selector` is still only `app: web` | A selector only needs enough labels to find its Pods. Also, **a Deployment's selector can't be changed after creation** (`field is immutable`) |

### Common labels in real teams

| Label | Answers | Example values |
|---|---|---|
| `app` | What app is this? | `web`, `order-service` |
| `team` | Who owns it? | `orders`, `payments` |
| `tier` | Which layer? | `frontend`, `backend`, `database` |
| `env` | Which environment? | `dev`, `prod` |
| `version` | Which release? | `v1`, `v2` |

## Hands-on

### 1. Create the namespaces
```bash
kubectl apply -f namespaces.yaml
kubectl get namespaces
```
`orders-dev` and `orders-prod` appear in the list.

### 2. Run the same app in both
```bash
kubectl apply -f app.yaml -n orders-dev
kubectl apply -f app.yaml -n orders-prod
kubectl get deployments,services -n orders-dev
kubectl get deployments,services -n orders-prod
kubectl get deployments,services
```

```
orders-dev:   deployment.apps/web-deployment 2/2   service/web-service 10.96.161.48
orders-prod:  deployment.apps/web-deployment 2/2   service/web-service 10.96.218.241
default:      service/kubernetes 10.96.0.1   ← your app isn't here
```
- Same names in both namespaces, each with its **own** ClusterIP
- The built-in `kubernetes` Service in `default` is how Pods reach the API server

### 3. Call Services across namespaces
Start a test Pod **inside `orders-dev`**:
```bash
kubectl run test-client --rm -it --image=curlimages/curl -n orders-dev -- sh
```
```sh
curl -sv -o /dev/null web-service 2>&1 | grep -E "Trying|onnect"
curl -sv -o /dev/null web-service.orders-prod 2>&1 | grep -E "Trying|onnect"
cat /etc/resolv.conf
exit
```

```
*   Trying 10.96.161.48:80...                 ← "web-service" reached DEV
*   Trying 10.96.218.241:80...                ← "web-service.orders-prod" reached PROD

search orders-dev.svc.cluster.local svc.cluster.local cluster.local
nameserver 10.96.0.10
options ndots:5
```

**How short names work (`/etc/resolv.conf`):**

| Line | Meaning |
|---|---|
| `search ...` | Endings tried on short names, in order. `web-service` becomes `web-service.orders-dev.svc.cluster.local` first: your own namespace is assumed, like dialing a phone number without the area code |
| `nameserver 10.96.0.10` | Who answers DNS questions: the Service in front of the `coredns` Pods in `kube-system` |
| `options ndots:5` | Names with fewer than 5 dots get the `search` endings tried first |

| Calling from | You type | You reach |
|---|---|---|
| Same namespace | `web-service` | Your namespace's Service |
| Another namespace | `web-service.orders-prod` | That namespace's Service |
| Anywhere | `web-service.orders-prod.svc.cluster.local` | The exact Service |

### 4. Work across namespaces without typing `-n`
See everything in the cluster:
```bash
kubectl get pods -A
```
`-A` = all namespaces, with a NAMESPACE column showing where each Pod lives.

Change your default namespace:
```bash
kubectl config set-context --current --namespace=orders-dev
kubectl get pods                                        # shows dev Pods, no -n needed
kubectl config view --minify | grep namespace           # namespace: orders-dev
kubectl config set-context --current --namespace=default
```
A **context** is a saved combination of *which cluster* and *which namespace*.

> ⚠️ Forgetting which namespace you're in is a classic real-world mistake, like deleting in prod when you thought you were in dev. Always check before deleting, and switch back to `default` after experimenting.

### 5. See and filter labels
After adding `team` and `tier` to the template and re-applying (this triggers a rolling update):
```bash
kubectl get pods -A -l app=web --show-labels
```

```
NAMESPACE     NAME                              LABELS
orders-dev    web-deployment-55f786dc9b-6czf6   app=web,pod-template-hash=55f786dc9b,team=orders,tier=frontend
orders-dev    web-deployment-55f786dc9b-xwjz6   app=web,pod-template-hash=55f786dc9b,team=orders,tier=frontend
orders-prod   web-deployment-55f786dc9b-499nj   app=web,pod-template-hash=55f786dc9b,team=orders,tier=frontend
orders-prod   web-deployment-55f786dc9b-t7c8s   app=web,pod-template-hash=55f786dc9b,team=orders,tier=frontend
```
- **`pod-template-hash`** is added by Kubernetes, not by you. It's the same hash that appears in Pod names, and it lets each ReplicaSet tell its own Pods apart
- The hash changed from `6d8567df8` to `55f786dc9b` because new labels mean a new template
- `service/web-service unchanged`: `apply` only changes what's different

Label filters:

| Command | Matches |
|---|---|
| `kubectl get pods -A -l app=web -L team,tier` | Shows labels as columns |
| `kubectl get pods -A -l tier=frontend` | `tier` equals `frontend` |
| `kubectl get pods -A -l team=orders,tier=frontend` | Both conditions (comma = AND) |
| `kubectl get pods -A -l 'team in (orders,payments)'` | Any of several values |
| `kubectl get pods -A -l 'tier,tier!=backend'` | Has a `tier` label, and it isn't `backend` |
| `kubectl get pods -A -l tier=backend` | Nothing: `No resources found` |

### 6. Pull a Pod out of its Deployment with one label
This shows how labels connect everything.

```bash
kubectl label pod web-deployment-55f786dc9b-6czf6 -n orders-dev app=debug --overwrite
kubectl get pods -n orders-dev -o wide --show-labels
kubectl get endpointslices -n orders-dev -l kubernetes.io/service-name=web-service
```

```
Before:     6czf6 (10.244.0.10, app=web)    xwjz6 (10.244.0.11)
            Endpoints: 10.244.0.10, 10.244.0.11

After:      6czf6 (10.244.0.10, app=debug)  ← still running, but on its own
            mqmnw (10.244.0.14, app=web)    ← new Pod created by the ReplicaSet
            xwjz6 (10.244.0.11, app=web)
            Endpoints: 10.244.0.11, 10.244.0.14   ← .10 dropped automatically
```

One label change, two separate reactions:
- The **ReplicaSet** looks for `app=web`, found only 1 Pod, and created a replacement
- The **Service** looks for `app=web` and stopped sending traffic to `6czf6`

They never talk to each other. Each just watches labels.

**Real-world use:** if one production Pod behaves strangely, relabel it. Customers stop reaching it, a healthy replacement starts, and the odd Pod keeps running so you can investigate with `logs` and `exec`.

Clean up the orphan:
```bash
kubectl delete pod web-deployment-55f786dc9b-6czf6 -n orders-dev
kubectl get pods -n orders-dev
```
Back to 2 Pods, and **no replacement** is created, because the Deployment no longer considers it its own.

## Break it on purpose
Try changing the Deployment's selector in `app.yaml`:
```yaml
  selector:
    matchLabels:
      app: web
      team: orders
```
Then:
```bash
kubectl apply -f app.yaml -n orders-dev
```
Kubernetes rejects it with `field is immutable`. A Deployment's selector is fixed once it's created. Change it back to just `app: web`.

## Common errors

| Error | Cause | Fix |
|---|---|---|
| `No resources found in default namespace` | Your objects are in another namespace | Add `-n <namespace>`, or use `-A` to search everywhere |
| `-l 'tier!=backend'` matches Pods with no `tier` label | `!=` also matches Pods missing the label | Use `-l 'tier,tier!=backend'` ("has tier, and not backend") |
| A filter matches system Pods you didn't expect | Kubernetes labels its own Pods too (e.g., `tier=control-plane`) | Filter on labels only your app has, like `app=web` or `team=orders` |
| `field is immutable` | Tried to change a Deployment's selector | Delete and recreate the Deployment, or keep the selector unchanged |
| `curl: Could not resolve host` across namespaces | Used a short name for a Service in another namespace | Use `<service>.<namespace>` |
| Commands affect the wrong namespace | Default namespace was changed and not switched back | `kubectl config view --minify \| grep namespace` to check |

## Cleanup
```bash
kubectl delete namespace orders-dev orders-prod
```
**Deleting a namespace deletes everything inside it**: Deployments, Services, and Pods. Powerful, so use it carefully.

## Key takeaways
- **Namespaces** split one cluster into separate spaces; the same name can exist in each
- Short Service names resolve in **your own namespace**; use `<service>.<namespace>` for others
- `-n` picks a namespace, `-A` shows all, and **contexts** set your default
- **Labels** are tags; one object can have many, and `-l` filters by them
- `!=` also matches objects **without** the label
- Deployments and Services both follow labels, so **changing a label changes who manages a Pod and who sends it traffic**

## Next
👉 05 - ConfigMaps & Secrets
