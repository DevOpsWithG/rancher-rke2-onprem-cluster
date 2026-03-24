# 🚀 RKE2 Kubernetes Cluster Setup, Upgrade & Recovery Guide

## 📌 Overview

This document describes the end-to-end setup of a Kubernetes cluster using RKE2, including:

* Cluster installation
* Node joining
* Backup & restore (etcd snapshot)
* Safe upgrade strategy
* Troubleshooting real-world failure

---

## 🏗️ Cluster Architecture

* 1 Control Plane Node (Master)
* 1 Worker Node
* Private networking (AWS EC2)

---

## ⚙️ Installation Steps

### 1. Install RKE2 Server (Master)

```bash
curl -sfL https://get.rke2.io | sh -
sudo systemctl enable rke2-server
sudo systemctl start rke2-server
```

### 2. Configure kubectl

```bash
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
```

---

### 3. Get Node Token

```bash
sudo cat /var/lib/rancher/rke2/server/node-token
```

---

### 4. Install RKE2 Agent (Worker)

```bash
curl -sfL https://get.rke2.io | INSTALL_RKE2_TYPE="agent" sh -
```

Edit config:

```yaml
server: https://<MASTER_IP>:9345
token: <NODE_TOKEN>
```

Start agent:

```bash
sudo systemctl enable rke2-agent
sudo systemctl start rke2-agent
```

---

## ✅ Verify Cluster

```bash
kubectl get nodes
```

---

## 💾 Backup (etcd Snapshot)

```bash
sudo rke2 etcd-snapshot save
```

Snapshots stored at:

```
/var/lib/rancher/rke2/server/db/snapshots/
```

---

## ♻️ Restore Cluster

```bash
sudo systemctl stop rke2-server

sudo rke2 server \
  --cluster-reset \
  --cluster-reset-restore-path=<snapshot-path>

sudo systemctl start rke2-server
```

---

## 🔄 Upgrade Strategy (Safe Approach)

### Step 1: Drain Worker

```bash
kubectl drain <worker-node> --ignore-daemonsets --delete-emptydir-data
```

---

### Step 2: Upgrade Worker

```bash
curl -sfL https://get.rke2.io | INSTALL_RKE2_TYPE=agent sh -
sudo systemctl restart rke2-agent
```

---

### Step 3: Uncordon Worker

```bash
kubectl uncordon <worker-node>
```

---

### Step 4: Cordon Master

```bash
kubectl cordon <master-node>
```

---

### Step 5: Upgrade Master

```bash
curl -sfL https://get.rke2.io | sh -
sudo systemctl restart rke2-server
```

---

## ⚠️ Failure Scenario (Real Case)

### Issue:

* Upgrade to RC version broke control plane
* etcd not reachable (port 2379)

### Error:

```
connection refused 127.0.0.1:2379
```

---

## 🛠️ Recovery Steps

1. Stop RKE2
2. Restore snapshot
3. Restart service
4. Rejoin worker node

---

## 🧠 Key Learnings

* Always take backup before upgrade
* Avoid RC versions in production
* Kubernetes does not
