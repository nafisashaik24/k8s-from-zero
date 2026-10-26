# Prerequisites

You do **not** need any Kubernetes or cloud experience. This page covers everything you need before starting.

## What you should know
- How to open a terminal
- How to edit a text file
- That's it. Everything else is explained in the lessons.

## Your computer

| Requirement | Minimum | Recommended |
|---|---|---|
| Operating system | macOS, Windows 10/11, or Linux | — |
| Memory (RAM) | 8 GB | 16 GB |
| Free disk space | 10 GB | 20 GB |

Everything in this repo is **free**. No cloud account or credit card needed.

## Tools you'll install
Installation steps are in [00 - Setup](../00-setup/).

| Tool | Why you need it |
|---|---|
| **Docker Desktop** | Runs containers on your computer |
| **kubectl** | The command-line tool to talk to Kubernetes |
| **kind** | Creates a Kubernetes cluster inside Docker |
| **VS Code** | Editor for writing YAML files |
| **Git** | Saves your work and tracks changes |

## Terminal basics
These commands come up in every lesson:

| Command | What it does | Example |
|---|---|---|
| `pwd` | Show which folder you're in | `pwd` |
| `ls` | List files in the current folder | `ls` |
| `cd` | Move into a folder | `cd 01-pods` |
| `cd ..` | Go up one folder | `cd ..` |
| `mkdir` | Create a folder | `mkdir 03-services` |
| `Ctrl+C` | Stop a running command | stops `kubectl get pods -w` |
| `↑` (up arrow) | Repeat the previous command | — |

## YAML basics
Kubernetes is configured with **YAML** files. Four rules cover almost everything:

```yaml
# 1. Lines starting with # are comments
name: my-app            # 2. key: value pairs
metadata:               # 3. Indentation (2 spaces) shows that
  labels:               #    "labels" belongs to "metadata"
    app: web
containers:             # 4. A dash (-) starts a list item
  - name: nginx
  - name: helper
```

⚠️ **Always use spaces, never tabs.** Wrong indentation is the most common beginner error.

## GitHub account (optional)
Only needed if you want to save your own copy of your work. Create one free at [github.com](https://github.com/).

## Next
👉 [Concepts: the big picture](./concepts.md)
