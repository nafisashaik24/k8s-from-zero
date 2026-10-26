# 00 - Setup

## Why does this exist?
Before learning Kubernetes, you need a cluster to practice on. Real clusters run in the cloud and cost money. **kind** creates a real Kubernetes cluster on your own computer, for free.

## What we're installing

| Tool | What it does |
|---|---|
| **Docker Desktop** | Runs containers on your computer |
| **kubectl** | The command-line tool to talk to Kubernetes |
| **kind** | "Kubernetes IN Docker": runs a whole cluster inside a Docker container |
| **VS Code** | Editor for writing YAML |
| **Git** | Saves your work |

## 1. Install Docker Desktop

**Mac:** check your chip first:
```bash
uname -m
```
- `arm64` → download **Docker Desktop for Apple Silicon**
- `x86_64` → download **Docker Desktop for Intel**

Download from [docker.com](https://www.docker.com/products/docker-desktop/), open the `.dmg`, drag Docker into Applications, and launch it. Wait until the whale icon in the menu bar stops animating.

**Windows / Linux:** follow the official guide at [docs.docker.com/get-docker](https://docs.docker.com/get-docker/).

Verify:
```bash
docker version
docker run hello-world
```
Expected: `Hello from Docker!`

## 2. Install kubectl and kind

**Mac** (using [Homebrew](https://brew.sh/)):
```bash
brew install kubectl kind
```

**Windows:**
```bash
winget install -e --id Kubernetes.kubectl
winget install -e --id Kubernetes.kind
```

**Linux:** see [kubectl install](https://kubernetes.io/docs/tasks/tools/) and [kind install](https://kind.sigs.k8s.io/docs/user/quick-start/#installation).

Verify:
```bash
kubectl version --client
kind version
```

## 3. Install VS Code
Download from [code.visualstudio.com](https://code.visualstudio.com/), or on Mac:
```bash
brew install --cask visual-studio-code
```

Then install these free extensions (`Cmd+Shift+X` on Mac, `Ctrl+Shift+X` on Windows/Linux):
- **YAML** by Red Hat: catches indentation mistakes as you type
- **Kubernetes** by Microsoft: autocompletes Kubernetes fields

## 4. Create your cluster
```bash
kind create cluster --name learning
kubectl get nodes
```

Expected:
```
NAME                     STATUS   ROLES           AGE   VERSION
learning-control-plane   Ready    control-plane   1m    v1.3x.x
```

### What just happened?
1. kind downloaded a node image with all Kubernetes components inside
2. It started **one Docker container** that acts as a server (a **node**)
3. Inside it, the **control plane** started: API server, etcd, scheduler, controller manager
4. kind set up `kubectl` to talk to it (the connection is named `kind-learning`)
5. `--name learning` named the cluster, so you can run several side by side later

See it for yourself. Your whole cluster is one container:
```bash
docker ps
```

| Real cluster (in the cloud) | kind on your laptop |
|---|---|
| Node = a server | Node = a Docker container |
| Many nodes | One node by default |
| Costs money | Free |

The commands and YAML you learn here work the same way on a real cluster.

## Common errors

| Error | Cause | Fix |
|---|---|---|
| `Cannot connect to the Docker daemon` | Docker Desktop isn't running | Open Docker Desktop and wait for the whale icon |
| `node(s) already exist for a cluster with the name "learning"` | Cluster already exists | Just use it, or delete it first (below) |
| Node shows `NotReady` | Still starting up | Wait 30 seconds and run `kubectl get nodes` again |
| `kubectl` can't connect after a restart | Docker was closed | Open Docker Desktop. The cluster comes back automatically |

## Cleanup
Delete the cluster when you no longer need it:
```bash
kind delete cluster --name learning
```

## Next
👉 [01 - Pods](../01-pods/)
