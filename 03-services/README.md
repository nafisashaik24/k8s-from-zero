# 03 - Services

## Why does this exist?
In [Lesson 02](../02-deployments/), we saw that Pods are constantly replaced (self-healing, scaling, rolling updates), and **every new Pod gets a new IP address**. So how does one part of an app reliably talk to another?

Imagine a food delivery app. When you place an order, the **order-service** must ask the **restaurant-service** "is this restaurant open?". But restaurant-service runs as 3 Pods with 3 IPs that keep changing. Which IP should order-service call?

A **Service** solves this. It gives a group of Pods **one stable name and IP address** that never changes, and spreads requests across whichever Pods are healthy.

## Analogy
A Service is like a restaurant's **phone number**. Staff change shifts and new people are hired, but customers always call the same number, and whoever is available picks up.

## Key concepts
- A Service has a **stable name** (e.g., `web-service`) and a **stable IP** (ClusterIP)
- It finds its Pods **by label**, the same way a Deployment does
- It **load-balances** requests across all matching Pods
- When Pods are replaced, the Service **updates its list automatically** (EndpointSlices)
- Kubernetes has built-in **DNS**: other Pods reach a Service just by its name

```
caller ──► http://web-service  (stable name and IP, never changes)
                 │  finds Pods by label: app=web
                 ├──► Pod 10.244.0.21
                 ├──► Pod 10.244.0.22
                 └──► Pod 10.244.0.23
```

## The YAML, line by line
This lesson uses two files:
- [`deployment.yaml`](./deployment.yaml): the same nginx Deployment from Lesson 02 (3 replicas), so the Service has Pods to send traffic to
- [`service.yaml`](./service.yaml): the Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: ClusterIP
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 80
```

| Line | Meaning | Required? |
|---|---|---|
| `apiVersion: v1` | Services are core Kubernetes objects, like Pods | Yes |
| `kind: Service` | The type of object | Yes |
| `metadata.name` | **The stable name.** Other Pods reach it at `http://web-service`. Kubernetes registers it in the cluster's DNS | Yes |
| `spec.type: ClusterIP` | **Who can reach it.** ClusterIP = only from inside the cluster | No (ClusterIP is the default) |
| `spec.selector` | **Which Pods get the traffic:** every Pod labeled `app: web` | Yes, in practice |
| `ports.port` | The port callers use to reach the Service | Yes |
| `ports.targetPort` | The port on the Pods that traffic is forwarded to (where nginx listens) | No (defaults to `port`) |

### `port` vs. `targetPort`
```
caller ──► web-service:80 ──► Pod:80 (nginx)
              ↑ port            ↑ targetPort
```
They're often the same, but don't have to be. If an app listened on `8080`, you could set `port: 80` and `targetPort: 8080`. Callers use the simple port 80, and the Service forwards to 8080.

### Labels connect everything
```
Deployment ──creates Pods with──► app: web ◄──selects── Service
```
The Deployment and the Service never talk to each other directly. Both find the same Pods through the label `app: web`.

## Hands-on

### 1. Create the Deployment and the Service
```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl get pods -o wide
kubectl get service web-service
kubectl get endpointslices -l kubernetes.io/service-name=web-service
```

Expected:
```
NAME                             READY   STATUS    IP
web-deployment-6d8567df8-4xmnh   1/1     Running   10.244.0.21
web-deployment-6d8567df8-bg9s4   1/1     Running   10.244.0.23
web-deployment-6d8567df8-jd9mh   1/1     Running   10.244.0.22

NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)
web-service   ClusterIP   10.96.208.153   <none>        80/TCP

NAME                ADDRESSTYPE   PORTS   ENDPOINTS
web-service-cxlnw   IPv4          80      10.244.0.21,10.244.0.22,10.244.0.23
```

- **CLUSTER-IP** (`10.96.x.x`) is the Service's own address, from a different range than Pods (`10.244.x.x`). It stays the same for as long as the Service exists
- **ENDPOINTS** is the live list of Pods behind the Service. It matches the Pod IPs exactly
- **EXTERNAL-IP `<none>`** is expected: ClusterIP is internal only

> You may also see `kubectl get endpoints`. It still works but is deprecated. EndpointSlices are the modern replacement.

### 2. Call the Service by name from inside the cluster
A ClusterIP Service is only reachable inside the cluster, so start a temporary test Pod:
```bash
kubectl run test-client --rm -it --image=curlimages/curl -- sh
```

| Part | Meaning |
|---|---|
| `kubectl run test-client` | Start a Pod named `test-client` |
| `--image=curlimages/curl` | A tiny image containing `curl`, a tool for making web requests |
| `-it` | Interactive: gives you a shell inside the Pod |
| `--rm` | Delete the Pod automatically when you exit |
| `-- sh` | Run a shell inside it |

Inside the Pod (your prompt changes to `~ $`), try three ways to reach nginx:
```sh
curl web-service                              # by name (how real apps do it)
curl 10.96.208.153                            # by ClusterIP
curl web-service.default.svc.cluster.local    # by full DNS name
exit
```
All three return **Welcome to nginx!**

The full DNS name format is `<service>.<namespace>.svc.cluster.local`. The short name works because the test Pod is in the same namespace (`default`).

> **Tip:** if your prompt is `~ $`, you're inside a Pod. `kubectl` only exists on your computer, not inside Pods.

### 3. See load balancing
Send 6 requests from a test Pod:
```bash
kubectl run test-client --rm -it --image=curlimages/curl -- sh
```
```sh
for i in 1 2 3 4 5 6; do curl -s -o /dev/null web-service; echo "request $i sent"; done
exit
```

Then check which Pods handled them:
```bash
kubectl logs -l app=web --prefix --tail=3
```

Result (2-2-2 split):
```
[pod/web-deployment-6d8567df8-jd9mh/nginx] 10.244.0.25 - - "GET / HTTP/1.1" 200 ... "curl/8.23.0"
[pod/web-deployment-6d8567df8-jd9mh/nginx] 10.244.0.25 - - "GET / HTTP/1.1" 200 ... "curl/8.23.0"
[pod/web-deployment-6d8567df8-4xmnh/nginx] 10.244.0.25 - - "GET / HTTP/1.1" 200 ... "curl/8.23.0"
[pod/web-deployment-6d8567df8-4xmnh/nginx] 10.244.0.25 - - "GET / HTTP/1.1" 200 ... "curl/8.23.0"
[pod/web-deployment-6d8567df8-bg9s4/nginx] 10.244.0.25 - - "GET / HTTP/1.1" 200 ... "curl/8.23.0"
[pod/web-deployment-6d8567df8-bg9s4/nginx] 10.244.0.25 - - "GET / HTTP/1.1" 200 ... "curl/8.23.0"
```
- `-l app=web` gets logs from every Pod with that label; `--prefix` shows which Pod wrote each line
- `10.244.0.25` is the test Pod's IP (the caller)
- The split is random, not strictly in turn, so you may see 3-2-1 instead

### 4. The Service survives Pod replacement
```bash
kubectl delete pod web-deployment-6d8567df8-4xmnh
kubectl get pods -o wide
kubectl get endpointslices -l kubernetes.io/service-name=web-service
kubectl get service web-service
```

Result:
```
Endpoints before:  10.244.0.21, 10.244.0.22, 10.244.0.23
Endpoints after:   10.244.0.22, 10.244.0.23, 10.244.0.26   ← .21 removed, .26 added automatically
Service IP:        10.96.208.153                           ← unchanged
```
The Deployment replaced the Pod, Kubernetes updated the endpoints, and anyone calling `web-service` never noticed.

## Other Service types (preview)

| Type | Reachable from | Typical use |
|---|---|---|
| **ClusterIP** | Inside the cluster only | Apps talking to each other (this lesson) |
| **NodePort** | Outside, via a port on every node | Quick testing |
| **LoadBalancer** | Outside, via a cloud load balancer | Production apps in the cloud |

Opening apps to the outside world properly is covered in the Ingress lesson.

## Break it on purpose
Change the Service's selector to a label no Pod has:
```yaml
  selector:
    app: website
```
Then:
```bash
kubectl apply -f service.yaml
kubectl get endpointslices -l kubernetes.io/service-name=web-service
```
ENDPOINTS shows `<unset>`: the Service exists but finds no Pods, so requests to it fail. This is the most common Service mistake. Change it back to `app: web` and re-apply.

## Common errors

| Error | Cause | Fix |
|---|---|---|
| Endpoints show `<unset>` or are empty | Selector doesn't match any Pod labels | Make `spec.selector` match the Pods' labels |
| `curl: Could not resolve host` | Wrong Service name, or the Service is in another namespace | Check `kubectl get svc`; use `<service>.<namespace>` |
| `Connection refused` | `targetPort` doesn't match the port the app listens on | Set `targetPort` to the container's port |
| `AlreadyExists ... object is being deleted` | The previous test Pod is still shutting down | Wait ~30 seconds or use a different name |
| `kubectl: not found` | Running kubectl inside a Pod | Exit the Pod first |

## Cleanup
```bash
kubectl delete -f service.yaml
kubectl delete -f deployment.yaml
```

## Key takeaways
- Pod IPs change; **Services give a stable name and IP**
- Services find Pods **by label**, just like Deployments
- Built-in **DNS** lets Pods call each other by Service name
- Services **load-balance** across all healthy Pods
- **EndpointSlices** update automatically as Pods come and go
- **ClusterIP** is internal only; NodePort, LoadBalancer, and Ingress expose apps outside

## Next
👉 04 - Namespaces & Labels
