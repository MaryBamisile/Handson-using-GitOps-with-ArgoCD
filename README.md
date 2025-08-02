# Handson-using-GitOps-with-ArgoCD

✅ Step 1: Set Up Kubernetes Cluster on Your VM
Install kubectl and Kind (Kubernetes in Docker):

```
sudo apt-get update
sudo apt-get install -y curl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# Install Kind
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

Create the Kubernetes cluster: 
```
kind create cluster --name argocd-cluster
```
Verify the cluster is running:
```
kubectl get nodes
```


#### 🛠️ Step 2: Install ArgoCD
- Create the argocd namespace:
  ```
  kubectl create namespace argocd
  ```
  
- Install ArgoCD components:
  ```
  kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
  ```
  
- Wait for pods to be ready:
  ```
  kubectl get pods -n argocd
  ```
  
- Forward port to access the UI:
  ```kubectl port-forward svc/argocd-server -n argocd 8080:443```
  
- Get ArgoCD admin password (default is admin):
 ```
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d 
```

- Access the UI: https://localhost:8080
Use admin as the username and the password you just retrieved.

#### 📂 Step 3: Prepare Your Repository (already structured)

<img width="674" height="651" alt="image" src="https://github.com/user-attachments/assets/339954ec-2f87-4a21-abee-435ad173d073" />


#### 🚀 Step 4: GitHub Actions CI to Build and Push Docker Image
In .github/workflows/docker-build.yml:

```
name: Docker Build and Push

on:
  push:
    branches:
      - main

jobs:
  build_and_push:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Log in to DockerHub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Build Docker image
      run: |
        docker build -t yourdockerhubuser/my-app:${{ github.sha }} .

    - name: Push Docker image
      run: |
        docker push yourdockerhubuser/my-app:${{ github.sha }}
Commit and push to GitHub.
```


#### 📜 Step 5: Helm Chart Configuration
In helm/student-progress/values.yaml:

```
image:
  repository: yourdockerhubuser/my-app
  tag: latest
```

#### 🔄 Step 6: ArgoCD to Monitor Repo and Deploy App
Create an ArgoCD Application:

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: student-progress-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/MaryBamisile/CloudNative-DevOps.git'
    targetRevision: main
    path: helm/student-progress
  destination:
    server: https://kubernetes.default.svc
    namespace: student-progress
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
Apply the manifest:
```

```
kubectl apply -f application.yaml
```


#### ✅ Step 7: Sync and Monitor in ArgoCD UI
In the ArgoCD UI, click Sync to deploy. The app should be running in your cluster.
