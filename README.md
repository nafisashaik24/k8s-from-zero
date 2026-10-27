# k8s from zero 🚀

> Learning Kubernetes from zero, one hands-on lesson at a time, written so that a complete beginner can follow along.

Everything here runs **free on your own computer** using [kind](https://kind.sigs.k8s.io/) (Kubernetes in Docker). No cloud account, no credit card.

## Who this is for
- Students, career switchers, and engineers who have **never used Kubernetes**
- Anyone who wants to **understand why**, not just copy commands
- People who learn best by **doing**

## How to use this repo
1. Check the **[prerequisites](./docs/prerequisites.md)** (5 minutes)
2. Read **[concepts: the big picture](./docs/concepts.md)** (10 minutes)
3. Complete **[00 - Setup](./00-setup/)** to install the tools and create your cluster
4. Work through the lessons **in order**. Each one builds on the last

Every lesson follows the same structure:

**Why it exists → Analogy → YAML explained line by line → Hands-on with real output → Break it on purpose → Common errors → Cleanup → Key takeaways**

## Roadmap

| # | Lesson | What you'll learn | Status |
|---|---|---|---|
| 00 | [Setup](./00-setup/) | Install the tools and create a cluster | ✅ |
| 01 | [Pods](./01-pods/) | The smallest unit in Kubernetes | ✅ |
| 02 | [Deployments](./02-deployments/) | Self-healing, scaling, rolling updates, rollbacks | ✅ |
| 03 | [Services](./03-services/) | One stable address for changing Pods | ✅ |
| 04 | [Namespaces & Labels](./04-namespaces-labels/) | Organizing a cluster | ✅ |
| 05 | ConfigMaps & Secrets | Settings and passwords outside your code | 🚧 |
| 06 | Resources & Probes | CPU/memory limits and health checks | ⏳ |
| 07 | Storage | Keeping data when Pods restart | ⏳ |
| 08 | StatefulSets | Running databases | ⏳ |
| 09 | Ingress | Opening apps to the outside world | ⏳ |
| 10 | Jobs & CronJobs | One-time and scheduled tasks | ⏳ |
| 11 | RBAC | Who can do what in the cluster | ⏳ |
| 12 | Helm | Packaging apps | ⏳ |

✅ Done · 🚧 In progress · ⏳ Planned

## Projects
Real-world projects built after the core lessons: coming soon.

## Quick start
```bash
kind create cluster --name learning
kubectl get nodes
```

## How this repo is built
Each lesson is developed on its own branch and merged through a pull request, just like on a real engineering team. See **[git workflow](./docs/git-workflow.md)**.

## About
Built by **Nafisa Shaik** ([@nafisashaik24](https://github.com/nafisashaik24)) while learning Kubernetes from scratch. If this helped you, consider giving it a ⭐
