# Kubernetes Concepts: The Big Picture

Read this before starting the lessons. No commands here, just ideas.

## What is Kubernetes?
Kubernetes (often written **K8s**) is a system that **runs and manages containers for you**. You tell it *what you want* ("keep 3 copies of my app running"), and it makes that happen, and keeps it that way even when things crash.

## The shopping mall analogy

| Shopping mall | Kubernetes | What it is |
|---|---|---|
| The mall building | **Cluster** | The whole system |
| Each floor | **Node** | A machine that runs your apps |
| Each shop space | **Pod** | A wrapper with its own address where a container lives |
| The shopkeeper inside | **Container** | The actual program (e.g., a web server) |
| Mall management office | **Control plane** | Decides what runs where and keeps things in order |
| A brand with several shops | **Your application** | Many Pods working together |

## A real example: a messaging app
A big app like a messaging app is not one giant program. It's split into small pieces (**microservices**), and each piece runs in its own containers and Pods:

```
Cluster
├── Pod → message-service   (×3 copies)  sends and receives messages
├── Pod → user-service      (×3 copies)  login and profiles
├── Pod → media-service     (×2 copies)  photos and videos
├── Pod → website           (×2 copies)  the web version
└── Pod → database                       stores everything
```

Each piece scales on its own. New Year's Eve and everyone is texting? Run 20 copies of `message-service` without touching anything else.

## One cluster, many apps
A cluster usually runs several apps, kept separate with **namespaces** (like folders):

```
Cluster
├── namespace: messaging   → message-service, user-service...
├── namespace: payments    → payment-api...
└── namespace: monitoring  → dashboards, alerts...
```

## The key idea: desired state
You never tell Kubernetes *how* to do something step by step. You describe **what you want** (the desired state), like "3 copies of my app should be running". Kubernetes constantly compares that with **what's actually running** and fixes any difference. If a Pod crashes, Kubernetes notices "2 running, 3 wanted" and starts a new one.

## Anatomy of every Kubernetes file
Almost every Kubernetes YAML file has the same four parts:

```yaml
apiVersion: v1        # WHICH version of the API this follows
kind: Pod             # WHAT type of object
metadata:             # WHO it is (name, labels)
  name: example
spec:                 # WHAT it should do
  ...
```

Learn this shape once, and every new object is just a different `kind` with a different `spec`.

## What happens when you run `kubectl apply`

```
You ──kubectl──► API server ──► etcd (saves what you want)
                     │
                     ▼
                 Scheduler (picks a node)
                     │
                     ▼
                 kubelet on that node (pulls the image, starts the container)
```

## Glossary

| Term | Meaning |
|---|---|
| **Image** | A packaged app, ready to run (e.g., `nginx:1.27`). Like an installer file |
| **Container** | A running copy of an image |
| **Pod** | The smallest unit Kubernetes runs. Wraps a container and gives it an IP address |
| **Node** | A machine that runs Pods |
| **Cluster** | A group of nodes managed by Kubernetes |
| **Control plane** | The "brain" of the cluster |
| **API server** | The front door. Every `kubectl` command goes here |
| **etcd** | The database storing the cluster's state |
| **Scheduler** | Decides which node each Pod runs on |
| **Controller manager** | Keeps what's running matching what you asked for |
| **kubelet** | An agent on every node that starts the containers |
| **kubectl** | The command-line tool you use to talk to the cluster |
| **Manifest** | A YAML file describing what you want Kubernetes to create |
| **Label** | A `key: value` tag used to find and group objects |
| **Deployment** | Keeps a set number of Pod copies running and handles updates |
| **ReplicaSet** | Created by a Deployment to keep the exact number of Pods |
| **Service** | A stable name and address to reach a group of Pods |
| **Namespace** | A folder-like way to separate things in one cluster |

## Next
👉 [00 - Setup](../00-setup/)
