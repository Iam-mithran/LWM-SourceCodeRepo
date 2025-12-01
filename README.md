# Alpha – CI/CD Pipeline for Microservices with Jenkins, Docker & Argo CD on EKS  

A complete end-to-end CI/CD pipeline for deploying microservices to Kubernetes (EKS) using Jenkins (build), Docker (containerization), and Argo CD (GitOps), with a two-repo model (source repo + deployment repo).  

## 🚀 What is this  

This repository demonstrates how to:  
- Build and containerize microservices (for example, a Spring Boot backend + frontend) using Jenkins and Docker.  
- Store and manage deployment manifests (YAML/Helm) in a separate “deployment” repo (GitHub).  
- Use Argo CD to automatically deploy to an EKS cluster — enabling GitOps-based continuous delivery and environment promotion.  
- Provide a clear, minimal codebase + pipeline configuration — ideal for learning and teaching CI/CD fundamentals.  

## ✅ What You’ll Learn  

- How to setup a Jenkins pipeline that builds Docker images on code commit.  
- How to push Docker images to a container registry (e.g. AWS ECR / Docker Hub).  
- How to structure microservices and manifests for a GitOps workflow.  
- How to configure Argo CD to sync manifests from Git to EKS.  

---


# 🌐 How the Pipeline Works (Step-by-Step)

1. **Developer pushes code → Source Repo**
2. Jenkins CI job starts
3. SonarQube scans code quality
4. Docker image is built
5. Image pushed to AWS ECR
6. Jenkins updates `deployment.yaml` with new image tag
7. Jenkins commits the updated manifest to **Deployment Repo**
8. Argo CD detects the commit
9. Argo CD syncs & deploys to Kubernetes **EKS**
10. App is live 🎉

---

# 📂 Repository Structure

You will maintain **two separate repos**:

### **1️⃣ Source Code Repository**

Contains:

* `main.py` (FastAPI app) 
* `form.html` (frontend UI) 
* `output.html` (output UI) 
* `Dockerfile`
* `requirements.txt` 
* `sonar-project.properties` (static code analysis) 

### **2️⃣ Deployment Repository**

Contains:

* `deployment.yaml`
* `service.yaml`

Only these two files are needed for GitOps (Argo CD).

---

# 🟦 `main.py`

FastAPI Application


```python
#uvicorn main:app --reload
from fastapi import FastAPI, Request, Form
from fastapi.templating import Jinja2Templates

app = FastAPI()
templates = Jinja2Templates(directory="/code")

@app.get("/")
def form_post(request: Request):
    return templates.TemplateResponse('form.html', context={'request': request})
```

---



# 🟦 `form.html`

Frontend Home Page


```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Welcome Page</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin-top: 50px;
        }
        .button {
            background-color: blue;
            color: white;
            padding: 10px 20px;
            font-size: 16px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
        .button:hover {
            background-color: darkblue;
        }
    </style>
</head>
<body>

    <h1>Welcome to Learn With Mithran</h1>
    <h1>powered by Greens Technologies</h1>
    <button class="button" onclick="alert('Hello!')">Hello</button>

</body>
</html>
```

---


# 🟦 `requirements.txt`



```txt
fastapi==0.68.1
uvicorn==0.13.4
Jinja2==3.0.3
python-multipart==0.0.5
```

---

# 🟦 `sonar-project.properties`



```txt
sonar.projectKey=lwm-fastapi-project
sonar.projectName=lwm FastAPI Project
sonar.projectVersion=1.0
sonar.sources=.
sonar.language=python
sonar.sourceEncoding=UTF-8
```

---

# 🟦 `Dockerfile`


```dockerfile
FROM python:3.9  
WORKDIR /code
COPY ./requirements.txt /code/requirements.txt
RUN pip install --no-cache-dir --upgrade -r /code/requirements.txt
COPY ./main.py /code/main.py
COPY ./form.html /code/form.html
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "80"]
```

---

# 🟦 `deployment.yaml`

(From Deployment Repo)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lwm-alpha
  labels:
    app: lwm-alpha
spec:
  replicas: 3
  selector:
    matchLabels:
      app: lwm-alpha
  template:
    metadata:
      labels:
        app: lwm-alpha
    spec:
      containers:
        - name: cont1
          image: 992382429239.dkr.ecr.ap-south-1.amazonaws.com/lwm-alpha:latest
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "256Mi"
          env:
            - name: ENV
              value: "dev"
```

---

# 🟦 `service.yaml`

(From Deployment Repo)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: lwm-alpha-svc
  labels:
    app: lwm-alpha
spec:
  type: LoadBalancer
  selector:
    app: lwm-alpha
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 80
```

---

# 🚀  ArgoCD + Jenkins CI/CD Setup

This document lists all the prerequisites required to set up the complete CI/CD environment using **Jenkins**, **Docker**, **SonarQube**, **Sonar Scanner**, and **Trivy**. These packages and configurations must be installed on the Jenkins server before configuring the pipeline.

## 🔧 1. Jenkins Installation

Install Java and Jenkins along with the official Jenkins repository and start the service.

### **Commands:**

```bash
sudo apt update
sudo apt install -y fontconfig openjdk-21-jre
java -version

sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install -y jenkins

sudo systemctl enable jenkins
sudo systemctl start jenkins
```

---

## 🐳 2. Docker Installation

Jenkins needs Docker access to build Docker images during the CI pipeline.

### **Commands:**

```bash
sudo apt-get update
sudo apt install -y docker.io
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

---

## 📊 3. SonarQube Installation

Run SonarQube using Docker (recommended for lab/demo environments).

### **Commands:**

```bash
docker run -d --name sonarqube -p 9000:9000 sonarqube:latest
```

➡️ **Note:** You must manually generate a SonarQube token after logging into the dashboard.

Example token used:

```
sqa_4162defc09f0948e06e4ee6937396ec939579f99
```

---

## 🔍 4. Sonar Scanner Installation

Sonar Scanner is required for Jenkins to run code quality analysis.

### **Commands:**

```bash
wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-5.0.1.3006-linux.zip
sudo apt install -y unzip
unzip sonar-scanner-cli-5.0.1.3006-linux.zip
sudo mv sonar-scanner-5.0.1.3006-linux /opt/sonar-scanner

export PATH=$PATH:/opt/sonar-scanner/bin'
```

---

## 🛡️ 5. Trivy Installation

Trivy is used to scan Docker images for security vulnerabilities.

### **Commands:**

```bash
sudo apt-get install -y wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -

echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list

sudo apt-get update
sudo apt-get install -y trivy

sudo mkdir -p /var/lib/jenkins/.cache/trivy/db
sudo chown -R jenkins:jenkins /var/lib/jenkins/.cache/trivy
```

---

## 🧩 6. Install and Configure kubectl
```bash
curl -o kubectl https://s3.us-west-2.amazonaws.com/amazon-eks/1.29.0/2024-01-04/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --client
```

---

## ☸️ 7. Connect to Your EKS Cluster

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws configure
aws eks update-kubeconfig --name <cluster-name> --region <region>
kubectl get nodes
```

---

## ⚙️ 8. Install Helm 3

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

Ensure nodes are listed and ready.

---

## 🚀 9. Install Argo CD in EKS

Create a namespace:

```bash
kubectl create namespace argocd
```

Use this values.yaml for exposing argocd in NodePort
```yaml
# Basic Argo CD values for NodePort exposure
server:
  service:
    type: NodePort
    nodePortHttp: 30080        # optional — choose a fixed NodePort (HTTP)
    nodePortHttps: 30443       # optional — choose a fixed NodePort (HTTPS)
    ports:
      http: 80
      https: 443
  ingress:
    enabled: false
  insecure: true                # serve UI without enforcing HTTPS (for lab use)
  extraArgs:
    - --insecure                # disable TLS redirection in ArgoCD server

configs:
  cm:
    # Disable TLS redirect so UI works over plain HTTP
    server.insecure: "true"

redis:
  enabled: true

controller:
  replicas: 1
repoServer:
  replicas: 1
applicationSet:
  replicas: 1
```

Install Argo CD using the official Helm:
```bash
# Add and update the Argo Helm repo
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update

# Install using the NodePort values file
helm upgrade --install argocd argo/argo-cd \
  -n argocd \
  -f values.yaml
```

Verify installation:

```bash
kubectl get pods -n argocd
```

---

## 🌐 10. Access the Argo CD UI

Access via browser:
Allow ==30080== port in Worker Node Security Group 
```
https://<ec2-publicip>:30080
```

Get the initial admin password:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o yaml
echo <base64secret> | base64 --decode
```

Login using:

```
Username: admin
Password: <decoded-password>
```

---

# 🧪 Testing the Application

Once deployed, access:

```
http://<LoadBalancer-IP>/
```

You will see the welcome page defined in `form.html`:

```html
<h1>Welcome to Learn With Mithran</h1>
```

---

# **📌 New Feature Release — Added `/output` Page (Post Initial Release)**

After the first full CI/CD + GitOps pipeline release and testing phase, a **new feature** was introduced to demonstrate how real-world teams handle updates through automated pipelines.

This feature includes:

### **1️⃣ New Route Added in `main.py`**

A new endpoint (`/output`) was added so the application can serve an additional page.

```python
@app.get("/output")
def form_post(request: Request):
    return templates.TemplateResponse('output.html', context={'request': request})
```



---

### **2️⃣ New HTML Page Added — `output.html`**

This page is used to visually display deployment logs and metadata in the UI.

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>{{ title | default("Deployment Output") }}</title>

  <!-- Simple, clean CSS (no external deps) -->
  <style>
    :root{
      --bg:#0f1724;    /* deep navy */
      --card:#0b1220;
      --muted:#9aa7bf;
      --accent:#6ee7b7; /* mint accent */
      --danger:#ff7b7b;
      --glass: rgba(255,255,255,0.04);
      --radius:12px;
      font-family: Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
    }
    *{box-sizing:border-box}
    body{
      margin:0;
      background: linear-gradient(180deg,#071024 0%, #071026 60%);
      color:#e6eef8;
      -webkit-font-smoothing:antialiased;
      -moz-osx-font-smoothing:grayscale;
      padding:28px;
    }

    .container{
      max-width:1100px;
      margin:0 auto;
      display:grid;
      grid-template-columns: 1fr 360px;
      gap:20px;
      align-items:start;
    }

    .card{
      background: linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
      border-radius:var(--radius);
      padding:18px;
      box-shadow: 0 6px 20px rgba(3,7,18,0.6);
      border: 1px solid rgba(255,255,255,0.03);
    }

    header.site-header{
      max-width:1100px;
      margin:0 auto 18px;
      display:flex;
      justify-content:space-between;
      align-items:center;
      gap:12px;
    }
    .title{
      display:flex;
      gap:12px;
      align-items:center;
    }
    .logo{
      width:44px; height:44px; border-radius:10px;
      background:linear-gradient(135deg,var(--accent), #4dd0e1);
      display:flex;align-items:center;justify-content:center;
      font-weight:700;color:#04202a;
      box-shadow: 0 6px 18px rgba(0,0,0,0.6);
    }
    h1{ font-size:20px; margin:0 0 2px }
    .subtitle{ color:var(--muted); font-size:13px }

    .meta { text-align:right; color:var(--muted); font-size:13px; }

    /* left column content */
    .summary { margin-bottom:12px; display:flex; gap:12px; align-items:center; flex-wrap:wrap; }
    .pill{
      background:var(--glass);
      color:var(--muted);
      padding:6px 10px;
      border-radius:999px;
      font-size:13px;
      border:1px solid rgba(255,255,255,0.02);
    }
    .status {
      padding:8px 12px;
      border-radius:999px;
      font-weight:700;
      font-size:13px;
      display:inline-block;
    }
    .status.success{ background: rgba(110,231,183,0.12); color:var(--accent); border:1px solid rgba(110,231,183,0.12) }
    .status.warn{ background: rgba(255,205,92,0.08); color:#ffd88a }
    .status.fail{ background: rgba(255,123,123,0.08); color:var(--danger) }

    /* logs */
    pre.log{
      background: #020617;
      padding:14px;
      border-radius:10px;
      max-height:480px;
      overflow:auto;
      color:#dbeafe;
      font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, "Roboto Mono", monospace;
      font-size:13px;
      line-height:1.45;
      border:1px solid rgba(255,255,255,0.02);
    }

    /* right column */
    .right h3{ margin:0 0 8px }
    .kv{ display:flex; justify-content:space-between; gap:8px; padding:8px 10px; border-radius:8px; background: linear-gradient(180deg, rgba(255,255,255,0.01), transparent); margin-bottom:8px; font-size:14px }
    .kv .k{ color:var(--muted); } .kv .v{ font-weight:700; text-align:right; }

    /* log line highlights */
    .ok{ color: #a7f3d0 }
    .warn{ color: #ffd88a }
    .err{ color: #ff9b9b }

    footer{ color:var(--muted); font-size:12px; margin-top:18px; text-align:center }
    @media (max-width:980px){
      .container{ grid-template-columns: 1fr; }
      .meta{ text-align:left; margin-top:6px }
    }
  </style>
</head>
<body>
  <header class="site-header">
    <div class="title">
      <div class="logo">AL</div>
      <div>
        <h1>{{ heading | default("Alpha — Deployment Output") }}</h1>
        <div class="subtitle">{{ subheading | default("CI → Build → Push → GitOps → Deploy") }}</div>
      </div>
    </div>
    <div class="meta">
      <div>Build: <strong>{{ build_number | default("—") }}</strong></div>
      <div>{{ timestamp | default("{{ now }}") }}</div>
    </div>
  </header>

  <main class="container" role="main">
    <!-- LEFT: summary + logs -->
    <section class="card">
      <div class="summary">
        <div class="pill">Repo: <strong>{{ repo | default("Iam-mithran/LWM-SourceCodeRepo") }}</strong></div>
        <div class="pill">Image: <strong>{{ image | default("992382.../lwm-alpha:latest") }}</strong></div>
        <div class="pill">Tag: <strong>{{ image_tag | default("latest") }}</strong></div>
        <div style="margin-left:auto">
          <span class="status {{ status_class | default('success') }}">{{ status_text | default("SUCCESS") }}</span>
        </div>
      </div>

      <h3 style="margin:8px 0 10px">Summary</h3>
      <p style="color:var(--muted); margin-top:0">
        {{ summary | default("The pipeline completed. See logs below for details. Click 'Copy' to copy output to clipboard.") }}
      </p>

      <h3 style="margin-top:14px">Build & Deploy Logs</h3>
      <pre class="log" id="logBlock" aria-live="polite">
{% for line in logs %}
<span class="{% if 'error' in line|lower or 'failed' in line|lower %}err{% elif 'warning' in line|lower or 'warn' in line|lower %}warn{% elif 'success' in line|lower or 'pushed' in line|lower or 'synced' in line|lower %}ok{% endif %}">{{ line }}</span>
{% endfor %}
      </pre>

      <div style="display:flex; gap:8px; margin-top:10px;">
        <button onclick="copyLogs()" style="background:var(--accent);border:none;padding:10px 12px;border-radius:10px;font-weight:700;cursor:pointer">Copy Logs</button>
        <a href="{{ external_link or '#' }}" target="_blank" style="text-decoration:none">
          <button style="background:transparent;border:1px solid rgba(255,255,255,0.06);padding:10px 12px;border-radius:10px;cursor:pointer">Open in Console</button>
        </a>
      </div>
    </section>

    <!-- RIGHT: metadata & actions -->
    <aside class="card right" aria-label="metadata">
      <h3>Deployment Info</h3>

      <div class="kv"><div class="k">Environment</div><div class="v">{{ env | default("dev") }}</div></div>
      <div class="kv"><div class="k">Replicas</div><div class="v">{{ replicas | default(3) }}</div></div>
      <div class="kv"><div class="k">Cluster</div><div class="v">{{ cluster | default("eks-prod") }}</div></div>
      <div class="kv"><div class="k">Namespace</div><div class="v">{{ namespace | default("default") }}</div></div>
      <div class="kv"><div class="k">Commit</div><div class="v">{{ commit | default("—") }}</div></div>
      <div style="height:12px"></div>

      <h3 style="margin-top:8px">Quick Actions</h3>
      <div style="display:flex;flex-direction:column;gap:8px">
        <form method="post" action="{{ action_retry or '#' }}">
          <button type="submit" style="width:100%;padding:10px;border-radius:8px;border:none;font-weight:700;cursor:pointer">Retry Deploy</button>
        </form>
        <a href="{{ argocd_url or '#' }}" target="_blank" style="text-decoration:none">
          <button style="width:100%;padding:10px;border-radius:8px;border:1px solid rgba(255,255,255,0.06);background:transparent;cursor:pointer">Open Argo CD</button>
        </a>
      </div>

      <footer>
        <div>Rendered by <strong>Alpha CI</strong></div>
        <div style="margin-top:6px">Status: <span style="font-weight:700">{{ status_text | default("SUCCESS") }}</span></div>
      </footer>
    </aside>
  </main>

  <script>
    function copyLogs(){
      const text = document.getElementById('logBlock').innerText;
      navigator.clipboard?.writeText(text).then(()=> {
        alert('Logs copied to clipboard');
      }, ()=> {
        alert('Failed to copy. Select and copy manually.');
      });
    }
    // auto-scroll to bottom
    const lb = document.getElementById('logBlock');
    if(lb) lb.scrollTop = lb.scrollHeight;
  </script>
</body>
</html>
```

---

### **3️⃣ Dockerfile Updated**

The Dockerfile was modified to ensure the new template (`output.html`) is copied into the container image during the build stage.

```Dockerfile
FROM python:3.9  
WORKDIR /code
COPY ./requirements.txt /code/requirements.txt
RUN pip install --no-cache-dir --upgrade -r /code/requirements.txt
COPY ./main.py /code/main.py
COPY ./form.html /code/form.html
COPY ./output.html /code/output.html
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "80"]
```

---

## 🚀 How This Demonstrates Real CI/CD behavior

This update shows how:

* Developers add new features
* Code is committed to **Source Repo**
* Jenkins automatically:

  * Builds a new Docker image
  * Pushes to ECR
  * Updates `deployment.yaml` in the **Deployment Repo**
* Argo CD detects the change and deploys it instantly

---

## 📺 Watch the Complete CI/CD + GitOps Pipeline Tutorial | Jenkins, Argo CD, Docker, SonarQube, Trivy, Kubernetes

👉 [Complete CI/CD + GitOps Pipeline Tutorial | Jenkins, Argo CD, Docker, SonarQube, Trivy, Kubernetes](https://youtu.be/IV0ndga0Oq4)

### 📞 Contact Us
**Phone:** [+91 91500 87745](tel:+919150087745)

### 💬 Ask Your Doubts
Join our **Discord Community**  
👉 [Click here to connect](https://discord.gg/N7GBNHBdqw)

### 📺 Explore More Learning
Subscribe to my **YouTube Channel** – *Learn With Mithran*  
🎯 [Watch Now](https://www.youtube.com/@LearnWithMithran)

---