# 01 - Pods

## Why does this exist?
Kubernetes never runs a container on its own. It always wraps it in a **Pod**, which gives the container an IP address, networking, and storage, and lets Kubernetes track whether it's healthy.

## Analogy
If a container is a **shopkeeper**, a Pod is the **shop space**: it has its own address, and the shopkeeper works inside it.

## Key concepts
- A Pod is the **smallest unit** Kubernetes runs
- Usually **one container per Pod**. Multiple containers only when they must work closely together (e.g., an app plus a helper "sidecar" that ships its logs)
- Containers in the same Pod share one IP address
- Pods are **temporary**. If a Pod is deleted or dies, it is not repaired. A replacement would be a brand-new Pod with a new name and IP

## The YAML, line by line
[`pod.yaml`](./pod.yaml):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-first-pod
  labels:
    app: web
spec:
  containers:
    - name: nginx
      image: nginx:1.27
      ports:
        - containerPort: 80
```

| Line | Meaning | Required? |
|---|---|---|
| `apiVersion: v1` | Pods are part of the core Kubernetes API, version 1 | Yes |
| `kind: Pod` | The type of object we're creating | Yes |
| `metadata.name` | Unique name, used in commands like `kubectl logs my-first-pod` | Yes |
| `metadata.labels` | `key: value` tags. Other objects use them to find this Pod | Optional, but almost always used |
| `spec` | What the Pod should do | Yes |
| `containers` | List of containers to run (`-` starts a list item) | Yes |
| `name: nginx` | The container's name inside the Pod | Yes |
| `image: nginx:1.27` | The image to run, pinned to version `1.27`. Without a tag, Kubernetes uses `latest`, which can change without warning | Yes |
| `containerPort: 80` | Documents that the app listens on port 80 | Optional, but good practice |

**What is nginx?** A popular, lightweight **web server**: a program that waits for requests from browsers and sends back web pages. It's used here as a simple stand-in app because it starts in seconds and shows a page you can see in your browser.

## Hands-on

### 1. Create the Pod
```bash
kubectl apply -f pod.yaml
kubectl get pods
```

Expected:
```
NAME           READY   STATUS    RESTARTS   AGE
my-first-pod   1/1     Running   0          9s
```

| Column | Meaning |
|---|---|
| READY | `1/1` = 1 of 1 containers ready |
| STATUS | `Running` = container started and is still running |
| RESTARTS | How many times the container crashed and restarted |
| AGE | How long ago the Pod was created |

### 2. See it in your browser
```bash
kubectl port-forward pod/my-first-pod 8080:80
```
Open http://localhost:8080. You'll see **Welcome to nginx!** Press `Ctrl+C` to stop.

**How the request reached nginx:**
```
Browser → localhost:8080 → kubectl port-forward → API server
        → node (learning-control-plane) → Pod (10.244.0.5:80) → nginx
```
- The Pod has a **private IP** your computer can't reach directly
- `8080:80` means "port 8080 on my computer → port 80 in the Pod"
- `port-forward` is a **temporary tunnel** for testing. It stops when you press `Ctrl+C`. Real apps use Services and Ingress (later lessons)

### 3. Inspect it
```bash
kubectl logs my-first-pod
```
At the bottom you'll see your browser's requests:
```
"GET / HTTP/1.1" 200 ...            ← the page loaded (200 = success)
"GET /favicon.ico HTTP/1.1" 404 ... ← browser asked for a tab icon nginx doesn't have (harmless)
```

```bash
kubectl get pod my-first-pod -o wide
```
Shows the Pod's **IP** and which **node** it runs on:
```
NAME           READY   STATUS    IP           NODE
my-first-pod   1/1     Running   10.244.0.5   learning-control-plane
```

```bash
kubectl describe pod my-first-pod
```
Scroll to **Events** to see what Kubernetes did, in order:
```
Scheduled → Pulling → Pulled → Created → Started
```
- `default-scheduler` picked the node
- `kubelet` (the agent on the node) pulled the image and started the container

## Break it on purpose
Change the image to one that doesn't exist:
```yaml
      image: nginx:doesnotexist
```
Then:
```bash
kubectl delete -f pod.yaml
kubectl apply -f pod.yaml
kubectl get pods
kubectl describe pod my-first-pod
```
You'll see `ErrImagePull`, then `ImagePullBackOff`. The Events section explains why. Change the image back to `nginx:1.27` afterwards.

## Common errors

| Error | Cause | Fix |
|---|---|---|
| `ErrImagePull` / `ImagePullBackOff` | Image name or tag is wrong | Fix the spelling or tag and re-apply |
| `error converting YAML to JSON` | Indentation mistake or a tab character | Use 2 spaces; check the line number in the error |
| `AlreadyExists` | A Pod with that name already exists | Delete it first, or use `kubectl apply` instead of `kubectl create` |
| `port-forward` stops working | The command was stopped or the Pod was deleted | Re-run it while the Pod is `Running` |

## Cleanup
```bash
kubectl delete -f pod.yaml
```

## Key takeaways
- Every manifest has 4 parts: `apiVersion`, `kind`, `metadata`, `spec`
- A Pod wraps a container and gives it a private IP inside the cluster
- `port-forward` is a temporary testing tunnel
- The **scheduler** picks the node; the **kubelet** starts the container
- Bare Pods are **not recreated** when deleted. Deployments solve this

## Next
👉 [02 - Deployments](../02-deployments/)
