# Install Standalone Kubernetes on Ubuntu with Security Hardening

Learn how to install a production-ready standalone Kubernetes cluster on Ubuntu with security hardening including UFW firewall configuration, system optimization, and best practices.

## 🎯 What You'll Learn

-   Install and configure a standalone Kubernetes cluster on Ubuntu
-   Harden Ubuntu system for production Kubernetes workloads
-   Configure UFW firewall for Kubernetes components
-   Set up containerd as the container runtime
-   Deploy and verify a working Kubernetes cluster
-   Apply security best practices

## ⏱️ Time Required

**30-45 minutes**

## 📋 Prerequisites

-   Ubuntu 22.04 LTS or 24.04 LTS server (fresh installation recommended)
-   Root or sudo access
-   At least 2 CPU cores and 4GB RAM
-   20GB+ available disk space
-   Static IP address configured
-   Internet connectivity

## 🏗️ Architecture Overview

This tutorial sets up a **single-node Kubernetes cluster** suitable for:

-   Development environments
-   Small production workloads
-   Learning and experimentation
-   Edge computing scenarios

## 🚀 Step-by-Step Installation

### Step 1: System Preparation and Hardening

First, update the system and install essential security tools:

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install security and utility packages
sudo apt install -y apt-transport-https ca-certificates conntrack curl \
    iptables \
    ebtables \
    ethtool \
    socat \
    iproute2 \
    gnupg lsb-release software-properties-common ufw fail2ban

```

Disable swap (required by Kubernetes)

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.\*\)$/#\1/g' /etc/fstab
```

or

```bash
# edit /etc/fstab
vi /etc/fstab

# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sda2 during curtin installation
/dev/disk/by-uuid/d2f40dad-bb1b-42c3-b588-7fbd87fd7747 / ext4 defaults 0 1
swap.img     none    swap    sw      0       0  # -> comment swap image
```

### Step 2: Configure UFW Firewall

Set up UFW with rules for Kubernetes components:

```bash
# Reset UFW to default settings
sudo ufw --force reset

# Set default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (IMPORTANT: Do this first!)
sudo ufw allow 22/tcp comment 'SSH'

# Kubernetes API Server
sudo ufw allow 6443/tcp comment 'Kubernetes API Server'

# etcd server client API
sudo ufw allow 2379:2380/tcp comment 'etcd server client API'

# Kubelet API
sudo ufw allow 10250/tcp comment 'Kubelet API'

# kube-scheduler
sudo ufw allow 10259/tcp comment 'kube-scheduler'

# kube-controller-manager
sudo ufw allow 10257/tcp comment 'kube-controller-manager'

# NodePort Services (if you plan to use NodePort)
sudo ufw allow 30000:32767/tcp comment 'NodePort Services'

# Cilium health checks
sudo ufw allow 4240/tcp comment 'Cilium Health Checks'

# Cilium VXLAN overlay
sudo ufw allow 8472/udp comment 'Cilium VXLAN'

# Cilium Hubble server (optional, for observability)
sudo ufw allow 4244/tcp comment 'Cilium Hubble Server'

# Cilium Hubble Relay (optional)
sudo ufw allow 4245/tcp comment 'Cilium Hubble Relay'

# Enable UFW
sudo ufw --force enable

# Verify firewall status
sudo ufw status numbered
```

### Step 3: System Kernel Parameters

Configure kernel parameters for Kubernetes networking:

```bash
# Load required kernel modules
cat <<EOF | sudo tee /etc/modules-load.d/kubernetes.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# Set sysctl parameters
cat <<EOF | sudo tee /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```

### Step 4: Install Containerd Runtime with Dependencies

Install and configure containerd, runc, and CNI plugins:

Installing CNI plugins, Download the `cni-plugins-<OS>-<ARCH>-<VERSION>.tgz` archive from [github releases](https://github.com/containernetworking/plugins/releases) , verify its `sha256sum`, and extract it under `/opt/cni/bin`:


```bash
# Install containerd and runc
sudo apt install -y containerd runc

mkdir -p /opt/cni/bin && \
tar Cxzvf /opt/cni/bin cni-plugin-version.tgz

# Create default containerd configuration
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml

# Configure systemd cgroup driver (required by Kubernetes)
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

# Restart containerd
sudo systemctl restart containerd
sudo systemctl enable containerd

# Verify containerd is running
sudo systemctl status containerd

# Verify runc is available
runc --version
```

!!! info "Why These Components?" 
- **containerd**: Container runtime that manages the complete container lifecycle 
- **runc**: Low-level container runtime that actually runs containers (OCI runtime)
- **CNI plugins**: Standard plugins for container networking (bridge, host-local, loopback, etc.)

Even though Cilium will handle pod networking, the CNI plugins provide essential network utilities that containerd and kubelet need for basic container operations.

add --config=/etc/containerd/config.toml to exec systemd library

```bash
[Service]
#uncomment to enable the experimental sbservice (sandboxed) version of containerd/cri integration
#Environment="ENABLE_CRI_SANDBOXES=sandboxed"
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/bin/containerd --config=/etc/containerd/config.toml
```

### Step 5: Install Kubernetes Components

Install kubeadm, kubelet, and kubectl:

```bash
# Add Kubernetes GPG key
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | \
    sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Add Kubernetes repository
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | \
    sudo tee /etc/apt/sources.list.d/kubernetes.list

# Update apt and install Kubernetes components
sudo apt update
sudo apt install -y kubelet kubeadm kubectl

# Hold packages at current version
sudo apt-mark hold kubelet kubeadm kubectl

# Verify installation
kubeadm version
kubectl version --client
```

### Step 6: Initialize Kubernetes Cluster

Initialize the cluster with kubeadm:

```bash
# Get your server's IP address
SERVER_IP=$(hostname -I | awk '{print $1}')

# Initialize the cluster (skip-phases=addon/kube-proxy for Cilium in kube-proxy replacement mode)
sudo kubeadm init \
    --pod-network-cidr=10.244.0.0/16 \
    --apiserver-advertise-address=$SERVER_IP \
    --node-name=$(hostname) \
    --skip-phases=addon/kube-proxy

# Set up kubeconfig for the current user
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

!!! success "Save the Join Command"
The `kubeadm init` output includes a join command. Save this if you plan to add worker nodes later.

### Step 7: Remove Taint (For Single-Node Cluster)

Allow pods to run on the control plane node:

```bash
# Remove the control-plane taint
kubectl taint nodes --all node-role.kubernetes.io/control-plane-

# Verify node is ready
kubectl get nodes
```

### Step 8: Install Cilium CNI

Install Cilium as the Container Network Interface with Hubble for observability:

```bash
# Install Cilium CLI
CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/main/stable.txt)
CLI_ARCH=amd64
if [ "$(uname -m)" = "aarch64" ]; then CLI_ARCH=arm64; fi
curl -L --fail --remote-name-all https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
sha256sum --check cilium-linux-${CLI_ARCH}.tar.gz.sha256sum
sudo tar xzvfC cilium-linux-${CLI_ARCH}.tar.gz /usr/local/bin
rm cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}

# Install Cilium with Hubble enabled (kube-proxy replacement mode)
cilium install \
    --version 1.16.5 \
    --set kubeProxyReplacement=true \
    --set k8sServiceHost=$SERVER_IP \
    --set k8sServicePort=6443

# Enable Hubble for network observability
cilium hubble enable --ui

# Wait for Cilium to be ready
cilium status --wait

# Verify Cilium installation
kubectl get pods -n kube-system -l k8s-app=cilium
```

!!! info "Cilium Features"
Cilium provides:

    - **eBPF-based networking** for high performance
    - **kube-proxy replacement** for better scalability
    - **Hubble** for network observability and monitoring
    - **Network policies** with L3-L7 security
    - **Service mesh capabilities** without sidecars### Step 9: Configure Fail2Ban for Additional Security

Protect SSH and other services:

```bash
# Create Fail2Ban SSH jail
cat <<EOF | sudo tee /etc/fail2ban/jail.local
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 5

[sshd]
enabled = true
port = 22
logpath = /var/log/auth.log
EOF

# Restart Fail2Ban
sudo systemctl restart fail2ban
sudo systemctl enable fail2ban

# Check Fail2Ban status
sudo fail2ban-client status sshd
```

### Step 10: Install Metrics Server (Optional but Recommended)

Enable resource metrics for kubectl top:

```bash
# Install Metrics Server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Patch Metrics Server for self-signed certificates (single-node setup)
kubectl patch deployment metrics-server -n kube-system --type='json' \
    -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--kubelet-insecure-tls"}]'

# Wait for Metrics Server to be ready
kubectl wait --for=condition=available deployment/metrics-server -n kube-system --timeout=300s
```

## ✅ Verification

### Check Cluster Status

```bash
# Check all nodes
kubectl get nodes -o wide

# Check system pods
kubectl get pods -A

# Check component health
kubectl get cs

# Test metrics server
kubectl top nodes
kubectl top pods -A
```

Expected output:

```
NAME          STATUS   ROLES           AGE   VERSION
ubuntu-k8s    Ready    control-plane   5m    v1.31.x

NAMESPACE        NAME                                   READY   STATUS    RESTARTS   AGE
kube-system      cilium-xxxxx                           1/1     Running   0          3m
kube-system      cilium-operator-xxxxxxxxxx-xxxxx       1/1     Running   0          3m
kube-system      coredns-xxxxxxxxxx-xxxxx               1/1     Running   0          5m
kube-system      coredns-xxxxxxxxxx-xxxxx               1/1     Running   0          5m
kube-system      etcd-ubuntu-k8s                        1/1     Running   0          5m
kube-system      kube-apiserver-ubuntu-k8s              1/1     Running   0          5m
kube-system      kube-controller-manager-ubuntu-k8s     1/1     Running   0          5m
kube-system      kube-scheduler-ubuntu-k8s              1/1     Running   0          5m
kube-system      hubble-relay-xxxxxxxxxx-xxxxx          1/1     Running   0          3m
kube-system      hubble-ui-xxxxxxxxxx-xxxxx             2/2     Running   0          3m
kube-system      metrics-server-xxxxxxxxxx-xxxxx        1/1     Running   0          2m
```

### Deploy Test Application

```bash
# Create a test deployment
kubectl create deployment nginx --image=nginx:latest

# Expose the deployment
kubectl expose deployment nginx --port=80 --type=NodePort

# Check the deployment
kubectl get deployment nginx
kubectl get pods -l app=nginx
kubectl get svc nginx

# Test access (get NodePort)
NODE_PORT=$(kubectl get svc nginx -o jsonpath='{.spec.ports[0].nodePort}')
curl http://localhost:$NODE_PORT
```

### Security Verification

```bash
# Check firewall status
sudo ufw status verbose

# Check Fail2Ban status
sudo fail2ban-client status

# Verify kernel modules
lsmod | grep -E 'overlay|br_netfilter'

# Check sysctl settings
sudo sysctl net.bridge.bridge-nf-call-iptables
sudo sysctl net.ipv4.ip_forward
```

### Cilium Network Verification

```bash
# Check Cilium status
cilium status

# Run Cilium connectivity test (comprehensive network verification)
cilium connectivity test

# Check Cilium health
kubectl exec -n kube-system ds/cilium -- cilium-health status

# View Cilium endpoints
kubectl get ciliumnodes -A
kubectl get ciliumendpoints -A
```

### Access Hubble UI (Optional)

```bash
# Port forward Hubble UI to access from your browser
kubectl port-forward -n kube-system svc/hubble-ui 12000:80

# Access Hubble UI at: http://localhost:12000
# Or expose via NodePort for remote access
kubectl patch svc hubble-ui -n kube-system -p '{"spec": {"type": "NodePort"}}'

# Get the NodePort
kubectl get svc hubble-ui -n kube-system
```

!!! tip "Hubble CLI"
Install Hubble CLI for command-line network observability:

```bash
HUBBLE_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/hubble/master/stable.txt)
    HUBBLE_ARCH=amd64
    if [ "$(uname -m)" = "aarch64" ]; then HUBBLE_ARCH=arm64; fi
curl -L --fail --remote-name-all https://github.com/cilium/hubble/releases/download/$HUBBLE_VERSION/hubble-linux-${HUBBLE_ARCH}.tar.gz{,.sha256sum}
sha256sum --check hubble-linux-${HUBBLE_ARCH}.tar.gz.sha256sum
    sudo tar xzvfC hubble-linux-${HUBBLE_ARCH}.tar.gz /usr/local/bin
rm hubble-linux-${HUBBLE_ARCH}.tar.gz{,.sha256sum}

    # Port forward Hubble Relay
    kubectl port-forward -n kube-system svc/hubble-relay 4245:80 &

    # Use Hubble CLI
    hubble status
    hubble observe
```

## 🔒 Security Best Practices

### 1. Regular Updates

```bash
# Create update script
cat <<'EOF' | sudo tee /usr/local/bin/k8s-update.sh
#!/bin/bash
apt update
apt list --upgradable
# Review updates before applying
# apt upgrade -y
EOF

sudo chmod +x /usr/local/bin/k8s-update.sh
```

### 2. Enable Audit Logging

```bash
# Create audit policy
sudo mkdir -p /etc/kubernetes/audit

cat <<EOF | sudo tee /etc/kubernetes/audit/policy.yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
  - level: Metadata
    resources:
    - group: ""
      resources: ["secrets", "configmaps"]
  - level: RequestResponse
    resources:
    - group: ""
      resources: ["pods"]
EOF

# Note: Add audit configuration to kube-apiserver manifest
# /etc/kubernetes/manifests/kube-apiserver.yaml
```

### 3. Configure RBAC

```bash
# Create a limited user role (example)
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: developer
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer-role
  namespace: default
rules:
- apiGroups: ["", "apps"]
  resources: ["pods", "deployments", "services"]
  verbs: ["get", "list", "watch", "create", "update", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-binding
  namespace: default
subjects:
- kind: ServiceAccount
  name: developer
  namespace: default
roleRef:
  kind: Role
  name: developer-role
  apiGroup: rbac.authorization.k8s.io
EOF
```

### 4. Network Policies

```bash
# Example: Deny all traffic by default
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
EOF
```

## 🔧 Troubleshooting

### Issue: Node Not Ready

```bash
# Check kubelet status
sudo systemctl status kubelet

# Check kubelet logs
sudo journalctl -u kubelet -f

# Verify containerd
sudo systemctl status containerd
```

### Issue: Pods Not Starting

```bash
# Check pod details
kubectl describe pod <pod-name> -n <namespace>

# Check Cilium status
cilium status

# Check Cilium logs
kubectl logs -n kube-system -l k8s-app=cilium

# Run Cilium connectivity test
cilium connectivity test

# Verify network configuration
ip route show
```

### Issue: Cannot Access Services

```bash
# Verify UFW rules
sudo ufw status numbered

# Check iptables rules
sudo iptables -L -n -v

# Test cluster DNS
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup kubernetes.default
```

<!-- ## 🔗 Next Steps

After your cluster is running:

1. **[Deploy Applications](../kubernetes/deploy-first-app.md)** - Deploy your first application
2. **[Set Up Ingress](../kubernetes/setup-ingress.md)** - Configure ingress controller for external access
3. **[Persistent Storage](../kubernetes/persistent-storage.md)** - Configure persistent volumes
4. **[Monitoring Setup](../kubernetes/prometheus-setup.md)** - Install Prometheus and Grafana
5. **[Backup Strategy](../kubernetes/backup-restore.md)** - Set up etcd backups -->

## 📚 Additional Resources

-   [Kubernetes Official Documentation](https://kubernetes.io/docs/)
-   [Cilium Documentation](https://docs.cilium.io/)
-   [Cilium Getting Started Guide](https://docs.cilium.io/en/stable/gettingstarted/)
-   [Hubble Observability](https://docs.cilium.io/en/stable/observability/hubble/)
-   [Ubuntu Server Security](https://ubuntu.com/server/docs/security-introduction)
-   [UFW Documentation](https://help.ubuntu.com/community/UFW)
-   [containerd Documentation](https://containerd.io/docs/)
-   [Kubernetes Security Best Practices](https://kubernetes.io/docs/concepts/security/security-checklist/)

## 💡 Tips for Production

!!! warning "Production Considerations" 
- Use a multi-node cluster for high availability 
- Implement proper backup and disaster recovery 
- Set up monitoring and alerting 
- Use infrastructure as code (Terraform, Ansible) 
- Implement GitOps workflows 
- Regular security audits and updates

!!! tip "Resource Management" 
- Set resource requests and limits for all pods 
- Use namespace quotas to prevent resource exhaustion 
- Monitor cluster capacity regularly 
- Plan for scaling before you need it

---

**Last Updated**: November 2025  
**Kubernetes Version**: 1.31.x  
**Ubuntu Version**: 22.04 LTS / 24.04 LTS
