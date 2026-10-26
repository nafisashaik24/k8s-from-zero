# 01 - Pods

## What is a Pod?
The smallest unit you can deploy in Kubernetes. A Pod wraps one or more containers that share the same network (IP address) and storage.

## What I did
- Wrote a Pod manifest for nginx (`pod.yaml`)
- Created it on a local kind cluster
- Accessed it from my browser using port-forward
- Inspected logs, Pod IP, node placement, and events
- Deleted it

## Commands
| Command | Purpose |
|---|---|
| `kubectl apply -f pod.yaml` | Create the Pod |
| `kubectl get pods` | List Pods and their status |
| `kubectl get pod my-first-pod -o wide` | Show Pod IP and node |
| `kubectl describe pod my-first-pod` | Full details and events |
| `kubectl logs my-first-pod` | Container output |
| `kubectl port-forward pod/my-first-pod 8080:80` | Tunnel localhost:8080 to Pod port 80 |
| `kubectl delete -f pod.yaml` | Delete the Pod |

## How traffic reached nginx
Browser → localhost:8080 → kubectl port-forward → API server → node (learning-control-plane) → Pod (10.244.0.5:80) → nginx

## Events I observed
Scheduled → Pulling → Pulled → Created → Started

## What I learned
- Every manifest has 4 parts: apiVersion, kind, metadata, spec
- Pods get a private IP that is not reachable from outside the cluster
- port-forward is a temporary debugging tunnel, not a permanent way to expose an app
- The scheduler picks the node; the kubelet on that node starts the container
- nginx logged my browser request (200) and a missing favicon (404)
- Bare Pods are not restarted if deleted — Deployments solve this (next lesson)