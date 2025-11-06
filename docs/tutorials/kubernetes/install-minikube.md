# Install Kubernetes with Minikube

⏱️ **Duration:** 8 minutes | 📊 **Level:** Beginner

Set up a local Kubernetes cluster using Minikube for development and learning.

## 🎯 What You'll Learn

-   Install Minikube on your local machine
-   Start your first Kubernetes cluster
-   Verify the installation with basic commands
-   Access the Kubernetes dashboard

## 📋 Prerequisites

-   **Docker** installed and running
-   **4GB+ RAM** available
-   **2+ CPU cores**
-   **20GB free disk space**
-   **Internet connection** for downloading images

Check your Docker installation:

```bash
docker --version
docker run hello-world
```

## 🚀 Step-by-Step Installation

### Step 1: Install kubectl

**On macOS:**

```bash
# Using Homebrew
brew install kubectl

# Or download directly
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

**On Ubuntu/Linux:**

```bash
# Download kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Make executable and move to PATH
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

**On Windows:**

```powershell
# Using Chocolatey
choco install kubernetes-cli

# Or download from GitHub releases
# https://github.com/kubernetes/kubernetes/releases
```

### Step 2: Install Minikube

**On macOS:**

```bash
# Using Homebrew
brew install minikube

# Or download directly
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
sudo install minikube-darwin-amd64 /usr/local/bin/minikube
```

**On Ubuntu/Linux:**

```bash
# Download and install
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

**On Windows:**

```powershell
# Using Chocolatey
choco install minikube

# Or download installer from:
# https://github.com/kubernetes/minikube/releases
```

### Step 3: Start Minikube

```bash
# Start Minikube with Docker driver
minikube start --driver=docker

# Specify resources (optional)
minikube start --driver=docker --memory=4096 --cpus=2
```

Expected output:

```
😄  minikube v1.32.0 on Darwin 13.0
✨  Using the docker driver based on user configuration
📌  Using Docker Desktop driver with root privileges
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=4096MB) ...
🐳  Preparing Kubernetes v1.28.3 on Docker 24.0.6 ...
🔎  Verifying Kubernetes components...
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

### Step 4: Verify Installation

```bash
# Check cluster status
minikube status

# Check nodes
kubectl get nodes

# Check cluster info
kubectl cluster-info
```

Expected output:

```bash
$ kubectl get nodes
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   2m    v1.28.3

$ kubectl cluster-info
Kubernetes control plane is running at https://192.168.49.2:8443
CoreDNS is running at https://192.168.49.2:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

## ✅ Verification Steps

### 1. Deploy a Test Application

```bash
# Create a simple nginx deployment
kubectl create deployment hello-minikube --image=nginx

# Expose it as a service
kubectl expose deployment hello-minikube --type=NodePort --port=80

# Check the deployment
kubectl get deployments
kubectl get services
```

### 2. Access the Application

```bash
# Get the service URL
minikube service hello-minikube --url

# Or open in browser directly
minikube service hello-minikube
```

### 3. Access Kubernetes Dashboard (Optional)

```bash
# Enable dashboard addon
minikube addons enable dashboard

# Start dashboard
minikube dashboard
```

## 🛠️ Useful Minikube Commands

```bash
# Check status
minikube status

# Stop cluster
minikube stop

# Delete cluster
minikube delete

# SSH into minikube VM
minikube ssh

# Check addons
minikube addons list

# Enable addon
minikube addons enable <addon-name>

# Get cluster IP
minikube ip
```

## 🧹 Cleanup

When you're done experimenting:

```bash
# Delete test deployment
kubectl delete deployment hello-minikube
kubectl delete service hello-minikube

# Stop minikube
minikube stop

# Delete cluster (if you want to start fresh)
minikube delete
```

## 🔧 Troubleshooting

### Issue: Docker not found

```bash
# Make sure Docker is running
docker ps

# If Docker Desktop, ensure it's started
# Check Docker settings for resource allocation
```

### Issue: Not enough resources

```bash
# Start with more resources
minikube delete
minikube start --driver=docker --memory=6144 --cpus=3
```

### Issue: Network connectivity

```bash
# Check if you can pull images
docker pull hello-world

# Check firewall settings
# Ensure corporate proxy isn't blocking
```

## 🔗 Next Steps

Now that you have Kubernetes running locally:

<!-- 1. **Learn the basics** - Try our [Kubernetes Fundamentals Course](../../courses/kubernetes-fundamentals/index.md) -->
<!-- 2. **Deploy real apps** - Follow [Deploy Your First App](deploy-first-app.md) tutorial -->
<!-- 3. **Explore addons** - Try [Setting up Ingress Controller](setup-ingress.md) -->
<!-- 4. **Production ready** - Learn about [managed Kubernetes services](../../courses/kubernetes-fundamentals/05-production/cluster-management.md) -->

---

!!! success "Congratulations! 🎉"
You now have a local Kubernetes cluster running. This is perfect for learning, development, and testing Kubernetes applications.

!!! tip "Development Workflow"
Keep Minikube running during development and use `kubectl` commands to interact with your cluster. Stop it with `minikube stop` when not in use to free up resources.

!!! info "Resource Usage"
Minikube uses Docker containers, so it shares resources with your Docker Desktop. Monitor your system resources if running many containers.
