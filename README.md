# Argo CD Learning 🚀

A hands-on learning repository for understanding **Kubernetes, GitOps, and Argo CD** through real-world projects.

This repository starts with a simple **Nginx GitOps deployment** and will gradually progress toward complete CI/CD and GitOps workflows using Kubernetes, Docker, GitHub Actions, container registries, and Argo CD.

---

## 📚 Learning Roadmap

```text
Project 1 → Nginx + Kubernetes + Argo CD
Project 2 → Node.js + Express + Docker + Kubernetes + Argo CD
Project 3 → React + Express + Docker + Kubernetes + Argo CD
Project 4 → FarmConnect Dev Environment
Project 5 → Complete CI/CD + GitOps
```

---

# 🏗️ Project 1 — Nginx GitOps

The first project demonstrates how Argo CD manages a Kubernetes application using a Git repository as the **source of truth**.

### Architecture

```text
                Developer
                    │
                    ▼
                  Git
                    │
                    ▼
                GitHub
              Desired State
                    │
                    ▼
                Argo CD
             GitOps Controller
                    │
                    ▼
              Kubernetes
              Live State
                    │
                    ▼
              Nginx Pods
             ┌────┬────┬────┐
             │Pod1│Pod2│Pod3│
             └────┴────┴────┘
                    │
                    ▼
              ClusterIP Service
                    │
                    ▼
             localhost:8081
```

---

# 🛠️ Technologies Used

| Technology | Purpose                      |
| ---------- | ---------------------------- |
| Git        | Version control              |
| GitHub     | Source repository            |
| Docker     | Container runtime            |
| Kubernetes | Container orchestration      |
| Minikube   | Local Kubernetes cluster     |
| kubectl    | Kubernetes CLI               |
| Argo CD    | GitOps / Continuous Delivery |
| Nginx      | Example application          |

---

# 💻 Environment

The project was developed and tested locally using:

```text
Docker       29.6.2
kubectl      1.36.2
Minikube     1.38.1
Git          2.55.0
Kubernetes   v1.35.1
```

No cloud provider is required for this learning project.

---

# 📁 Project Structure

```text
argocd-learning/
│
├── k8s/
│   ├── deployment.yml
│   └── service.yml
│
└── README.md
```

---

# ☸️ Kubernetes Configuration

## Deployment

The Nginx Deployment runs **3 replicas**.

```yaml
replicas: 3
```

Container image:

```text
nginx:latest
```

The Deployment ensures that the desired number of Pods are running.

---

## Service

The Nginx application is exposed using a Kubernetes `ClusterIP` Service.

```text
Service Type: ClusterIP
Port: 80
Target Port: 80
```

---

# 🚀 Running the Project Locally

## 1. Start Minikube

```bash
minikube start --driver=docker
```

Check the cluster:

```bash
kubectl get nodes
```

Expected result:

```text
NAME       STATUS   ROLES           ...
minikube   Ready    control-plane   ...
```

---

# 2. Deploy Nginx Manually

From the project directory:

```bash
kubectl apply -f k8s/
```

Check the Pods:

```bash
kubectl get pods
```

Expected:

```text
nginx-xxxxx   1/1   Running
nginx-xxxxx   1/1   Running
nginx-xxxxx   1/1   Running
```

Check the Deployment:

```bash
kubectl get deployment nginx
```

Expected:

```text
NAME    READY   UP-TO-DATE   AVAILABLE
nginx   3/3     3            3
```

---

# 🌐 3. Access Nginx

Forward the Kubernetes Service to localhost:

```bash
kubectl port-forward service/nginx 8081:80
```

Open:

```text
http://localhost:8081
```

You should see:

```text
Welcome to nginx!
```

---

# 🔗 4. GitHub Repository

The Kubernetes manifests are stored in GitHub.

Repository:

```text
argocd-learning
```

Git is used to maintain the desired Kubernetes state.

Basic Git workflow:

```bash
git add .
git commit -m "Update Kubernetes configuration"
git push
```

---

# 🔄 5. Argo CD Setup

Argo CD was installed inside the Minikube Kubernetes cluster.

Create the namespace:

```bash
kubectl create namespace argocd
```

Install Argo CD:

```bash
kubectl apply -n argocd --server-side --force-conflicts -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Check Argo CD Pods:

```bash
kubectl get pods -n argocd
```

All major Argo CD components should eventually become:

```text
Running
```

---

# 🖥️ 6. Access Argo CD UI

Port-forward the Argo CD server:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Open:

```text
https://localhost:8080
```

Because this is a local learning environment, the browser may show a certificate warning.

---

# 🔐 7. Get Initial Argo CD Password

PowerShell:

```powershell
$pwd = kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}"
[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($pwd))
```

Login using:

```text
Username: admin
Password: <generated password>
```

---

# 📦 8. Argo CD Application

An Argo CD Application named:

```text
nginx-gitops
```

was created with:

```text
Repository: argocd-learning
Branch: main
Path: k8s
Destination: in-cluster
Namespace: default
```

Argo CD reads the Kubernetes manifests from GitHub and compares them with the live Kubernetes state.

---

# 🔁 GitOps Workflow

The main concept learned in this project is:

```text
GitHub
   │
   │ Desired State
   ▼
Argo CD
   │
   │ Synchronization
   ▼
Kubernetes
   │
   │ Live State
   ▼
Nginx Application
```

### Desired State

The configuration stored in Git.

Example:

```yaml
replicas: 3
```

### Live State

The configuration currently running inside Kubernetes.

Argo CD continuously compares the desired state with the live state.

---

# 📈 GitOps Scaling Test

We changed the Deployment configuration from:

```yaml
replicas: 2
```

to:

```yaml
replicas: 3
```

Then committed and pushed the change:

```bash
git add k8s/deployment.yml
git commit -m "Scale nginx to 3 replicas"
git push
```

Argo CD detected the Git change.

After synchronization:

```text
Git              Kubernetes
3 replicas   →   3 replicas
```

Application status:

```text
SYNC STATUS: Synced
APP HEALTH:  Healthy
```

---

# ❤️ Kubernetes Self-Healing Test

One of the running Nginx Pods was manually deleted:

```bash
kubectl delete pod <pod-name>
```

Kubernetes automatically created a replacement Pod.

```text
Pod deleted
     │
     ▼
ReplicaSet detects missing Pod
     │
     ▼
New Pod created
     │
     ▼
3 Pods Running
```

This demonstrates Kubernetes **self-healing**.

---

# ⚠️ Argo CD Drift Detection Test

We manually changed the live Kubernetes state:

```bash
kubectl scale deployment nginx --replicas=2
```

But Git still contained:

```text
replicas: 3
```

Therefore:

```text
Git Desired State = 3
Kubernetes Live State = 2
```

Argo CD detected the difference:

```text
SYNC STATUS: OutOfSync
APP HEALTH: Healthy
```

After clicking:

```text
SYNC → SYNCHRONIZE
```

Argo CD restored Kubernetes to the desired Git state:

```text
Git = 3 replicas
Kubernetes = 3 replicas
```

---

# 🧠 What I Learned

Through Project 1, the following concepts were implemented and tested:

* ✅ Kubernetes cluster setup
* ✅ Minikube
* ✅ kubectl
* ✅ Kubernetes Deployment
* ✅ Kubernetes Service
* ✅ ReplicaSets
* ✅ Pod management
* ✅ Kubernetes self-healing
* ✅ Git
* ✅ GitHub
* ✅ GitOps fundamentals
* ✅ Argo CD installation
* ✅ Argo CD UI
* ✅ Argo CD Applications
* ✅ Git → Argo CD → Kubernetes workflow
* ✅ Manual synchronization
* ✅ GitOps scaling
* ✅ Drift detection
* ✅ OutOfSync state
* ✅ Argo CD reconciliation
* ✅ Desired State vs Live State

---

# ⭐ Important GitOps Concept

The most important concept from this project:

```text
Git = Desired State

Kubernetes = Live State

Argo CD = Reconciler
```

If the states are different:

```text
Desired State ≠ Live State
          │
          ▼
       OutOfSync
          │
          ▼
      Argo CD Sync
          │
          ▼
Desired State = Live State
```

---

# 🔮 Next Project

## Project 2 — Node.js + Express + Docker + Kubernetes + Argo CD

The next project will replace the simple Nginx application with a real Node.js/Express application.

Planned architecture:

```text
Developer
    │
    ▼
GitHub
    │
    ▼
Node.js + Express
    │
    ▼
Docker Image
    │
    ▼
Kubernetes
    │
    ▼
Argo CD
    │
    ▼
Running Application
```

Later projects will introduce:

```text
GitHub Actions
      ↓
Automated Tests
      ↓
Docker Build
      ↓
Container Registry
      ↓
GitOps Repository
      ↓
Argo CD
      ↓
Kubernetes
```

Eventually, this learning path will be applied to the **FarmConnect** production architecture.

---

# 🎯 Goal

The goal of this repository is to learn and implement a complete production-oriented DevOps and GitOps workflow step by step.

```text
Kubernetes
     +
Docker
     +
GitHub
     +
GitHub Actions
     +
Argo CD
     +
GitOps
     ↓
Complete CI/CD Pipeline
```

---

## Status

### Project 1 — Nginx GitOps

```text
Status: ✅ Completed
```

### Project 2 — Node.js + Express

```text
Status: 🔜 Next
```

### Project 3 — React + Express

```text
Status: 🔜 Planned
```

### Project 4 — FarmConnect

```text
Status: 🔜 Planned
```

### Project 5 — Complete CI/CD + GitOps

```text
Status: 🔜 Planned
```

---

## 👨‍💻 Learning Approach

This repository follows a practical, hands-on approach:

```text
Learn
  ↓
Build
  ↓
Deploy
  ↓
Break
  ↓
Troubleshoot
  ↓
Fix
  ↓
Automate
```

Each project builds on the previous one and gradually moves toward a production-ready DevOps workflow.
