# 🚀 Simple Local CI/CD Pipeline (HTML + Kubernetes)

**Sabse Simple Version** — Local Kubernetes pe fully automated CI/CD pipeline

### ✅ Features
- GitHub push → Jenkins automatic trigger
- Docker build + push
- Deploy to local Kubernetes (Kind / Minikube)
- Rolling updates
- Prometheus + Grafana monitoring (optional but included)
- **No AWS cost** — Purely local

---

## 🛠️ Tech Stack (Simplified)

| Tool          | Purpose                        | Local Tool          |
|---------------|--------------------------------|---------------------|
| GitHub        | Code + Webhook trigger         | Your repo           |
| Jenkins       | CI/CD automation               | Docker container    |
| Docker        | Containerization               | Docker Desktop      |
| Kubernetes    | Orchestration                  | **Kind** (recommended) |
| Nginx         | Web server (HTML)              | Alpine Nginx        |
| Prometheus    | Metrics                        | Helm chart          |
| Grafana       | Dashboards                     | Helm chart          |

---

## 📁 Project Structure

```
simple-local-cicd-k8s/
├── app/
│   ├── index.html          # Beautiful static HTML website
│   └── Dockerfile          # Nginx image
├── jenkins/
│   └── Jenkinsfile         # CI/CD Pipeline
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
└── README.md
```

---

## 🚀 Step-by-Step Setup (Local)

### Step 1: Install Prerequisites

```bash
# 1. Docker Desktop (already installed hoga mostly)
# 2. Install Kind (Kubernetes in Docker)
# macOS / Linux:
brew install kind

# Windows: https://kind.sigs.k8s.io/docs/user/quick-start/#installation

# 3. kubectl
# Already Docker Desktop ke saath aata hai ya alag se install karo
```

### Step 2: Create Local Kubernetes Cluster (Minikube)

```bash
minikube start --profile minikube

# Verify
kubectl get nodes
```

### Step 3: Start Jenkins (Docker mein)

```bash
docker run -d \
  --name jenkins \
  --restart unless-stopped \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins/jenkins:lts-jdk17
```

Jenkins URL: **http://localhost:8080**

Pehli baar password ke liye:
```bash
docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

### Step 4: Jenkins Setup (Important)

1. Jenkins mein login karo
2. **Plugins install karo**:
   - GitHub Integration
   - Pipeline
   - Docker Pipeline (optional)
3. **GitHub Credentials** banao (GitHub Container Registry ke liye):
   - Manage Jenkins → Credentials → Global → Add Credentials
   - Kind: **Username with password**
   - ID: `github-creds`
   - Username: Apna GitHub username
   - Password: GitHub **Personal Access Token (PAT)** with `write:packages` scope
   - Note: Classic PAT banao (Fine-grained bhi chal sakta hai)
4. Jenkins container mein `kubectl` access dena zaroori hai (next step)

### Step 5: Jenkins ko Kubernetes access do (Minikube)

Minikube usually apne aap `kubectl` configure kar deta hai. Phir bhi Jenkins container mein copy karna padta hai:

```bash
# Minikube ka config copy karo
minikube kubectl -- config view --flatten > /tmp/minikube-config.yaml

# Jenkins container mein copy karo
docker cp /tmp/minikube-config.yaml jenkins:/var/jenkins_home/.kube/config

# Test karo
docker exec -it jenkins kubectl get nodes
```

Agar upar wala kaam na kare toh yeh bhi try kar sakte ho:
```bash
minikube docker-env
```

### Step 6: Apna Project GitHub pe daalo

1. Is folder ko apne GitHub pe push kar do
2. `jenkins/Jenkinsfile` mein ye line change karo:
   ```groovy
   DOCKERHUB_REPO = 'your-dockerhub-username/devops-html-demo'
   ```

### Step 7: GitHub Webhook Setup (Automatic Trigger ke liye)

**Option A (Recommended for local):**
- [ngrok](https://ngrok.com/) install karo
- `ngrok http 8080` run karo
- Ngrok se jo public URL mile (jaise `https://xxxx.ngrok.io`), usko GitHub webhook mein daalo:
  - GitHub Repo → Settings → Webhooks → Add webhook
  - Payload URL: `https://xxxx.ngrok.io/github-webhook/`
  - Content type: `application/json`

**Option B (Simple):**
Jenkins job mein **Poll SCM** enable kar do (har 1 minute check karega).

### Step 8: Jenkins Job banao

1. Jenkins mein **New Item** → **Pipeline**
2. Pipeline from SCM → Git
3. Apna GitHub repo URL daalo
4. Branch: `main`
5. Script Path: `jenkins/Jenkinsfile`
6. **Build Now** dabaao

---

## 🌐 Apni Website Kaise Dekhe?

Pipeline success hone ke baad:

```bash
kubectl port-forward svc/devops-html-demo 8080:80 -n devops-demo
```

Browser mein jaao: **http://localhost:8080**

Beautiful HTML site dikhegi!

---

## 📊 Monitoring (Prometheus + Grafana) — Optional

Agar monitoring bhi dekhna hai to ye commands run karo:

```bash
# Add Helm repo
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install monitoring stack
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --set grafana.adminPassword='admin123'

# Port forward Grafana
kubectl port-forward svc/monitoring-grafana 3000:80 -n monitoring
```

Grafana: http://localhost:3000  
Username: `admin`  
Password: `admin123`

---

## 🧹 Cleanup (Sab delete karna ho to)

```bash
# Kubernetes cluster delete
kind delete cluster --name devops-local

# Jenkins container delete
docker rm -f jenkins

# Volume bhi delete karna ho to
docker volume rm jenkins_home
```

---

## 💡 Tips

- Pehli baar setup thoda time leta hai (especially Jenkins + kubectl config)
- Agar `kubectl` Jenkins mein nahi chal raha to Step 5 carefully follow karo
- Image tag `BUILD_NUMBER` se hota hai, isliye har push pe naya version deploy hota hai
- Rolling update ki wajah se website down nahi hoti

---

**Yeh version bahut simple hai** — sirf HTML + Nginx + Local Kubernetes.

Agar baad mein FastAPI ya Python app add karna ho, ya ArgoCD use karna ho, to bata dena. Main upgrade kar dunga.

Happy Learning! 🚀
```

---

**Made simple for local development** — No cloud charges, no complexity.