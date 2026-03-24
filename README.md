
```markdown
# 🚀 RKE2 Kubernetes Cluster Setup on AWS EC2

This repository documents the step-by-step setup of a multi-node Kubernetes cluster using Rancher RKE2 on AWS EC2 instances.

---

## 📌 Overview

- Kubernetes Distribution: Rancher RKE2
- Infrastructure: AWS EC2
- Setup Type: Single Control Plane + Worker Node
- Networking: Default CNI (Calico)
- OS: Ubuntu

---

## 🧱 Architecture

```

EC2 Instance 1 (Master)

* RKE2 Server
* Control Plane + etcd

EC2 Instance 2 (Worker)

* RKE2 Agent
* Workload Node

````

---

## ⚙️ Prerequisites

- 2 EC2 instances (Ubuntu)
- Open ports:
  - 22 (SSH)
  - 6443 (Kubernetes API)
  - 9345 (RKE2 cluster communication)
- Root or sudo access

> RKE2 installation must be executed with root privileges or sudo. :contentReference[oaicite:1]{index=1}

---

## 🚀 Step 1: Install RKE2 Server (Master Node)

```bash
curl -sfL https://get.rke2.io | sudo sh -
sudo systemctl enable rke2-server
sudo systemctl start rke2-server
````

### Verify status:

```bash
sudo systemctl status rke2-server
```

---

## 🔑 Step 2: Get Node Token

```bash
sudo cat /var/lib/rancher/rke2/server/node-token
```

---

## ⚙️ Step 3: Configure kubectl

RKE2 installs kubectl internally (not in PATH).

```bash
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
export PATH=$PATH:/var/lib/rancher/rke2/bin
```

> kubeconfig is stored at `/etc/rancher/rke2/rke2.yaml` by default. ([docs.rke2.io][1])

---

## 🧩 Step 4: Install RKE2 Agent (Worker Node)

```bash
curl -sfL https://get.rke2.io | sudo INSTALL_RKE2_TYPE="agent" sh -
```

---

## ⚙️ Step 5: Configure Agent

```bash
sudo mkdir -p /etc/rancher/rke2
sudo vi /etc/rancher/rke2/config.yaml
```

### Add:

```yaml
server: https://<MASTER_PRIVATE_IP>:9345
token: <NODE_TOKEN>
```

> RKE2 server listens on port 9345 for node registration. ([docs.rke2.io][1])

---

## ▶️ Step 6: Start Agent

```bash
sudo systemctl enable rke2-agent
sudo systemctl start rke2-agent
```

---

## ✅ Step 7: Verify Cluster

Run on master node:

```bash
kubectl get nodes
```

### Expected Output:

```
NAME              STATUS   ROLES
master-node       Ready    control-plane,etcd
worker-node       Ready    <none>
```

---

## 🧪 Step 8: Deploy Test Application

```bash
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=NodePort
```

---

## 🔥 Troubleshooting

### ❌ kubectl localhost:8080 error

```bash
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
```

---

### ❌ Worker not joining

* Check:

  * Token mismatch
  * Port 9345 open
  * Using private IP

---

### ❌ kubectl not found

```bash
export PATH=$PATH:/var/lib/rancher/rke2/bin
```

---

## 🔐 Security Considerations

* Use private IP for node communication
* Restrict security group access
* Enable RBAC (default in Kubernetes)

---

## 📚 References

* Official RKE2 Docs: [https://docs.rke2.io/install/quickstart](https://docs.rke2.io/install/quickstart)

---

## 🎯 What We Achieved

* ✅ Multi-node Kubernetes cluster
* ✅ RKE2 control plane setup
* ✅ Worker node joined successfully
* ✅ kubectl configured
* ✅ Application deployed

---

## 🚀 Next Steps

* Install Ingress Controller
* Setup Load Balancer (MetalLB)
* Enable Monitoring (Prometheus + Grafana)
* Configure CI/CD pipeline

---

## 👨‍💻 Author
Ganesh - DevOps Engineer

[1]: https://docs.rke2.io/install/quickstart?utm_source=chatgpt.com "Quick Start | RKE2"
