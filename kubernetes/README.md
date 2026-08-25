# 🚀 Kubernetes & Amazon EKS Installation Guide

This guide explains how to install and configure the essential tools required to work with **Kubernetes and Amazon EKS** on an Ubuntu/Linux machine.

---

## 📋 Prerequisites

Make sure you have:

* Ubuntu/Linux machine
* Internet connection
* AWS account
* IAM user/role with appropriate permissions
* `sudo` access

---

# 🛠️ 1. Install Required Packages

Update the system and install the required packages:

```bash
sudo apt update -y

sudo apt install -y curl tar unzip git
```

---

# ⚙️ 2. Install eksctl

`eksctl` is a command-line tool used to create and manage Amazon EKS clusters.

### Download eksctl

```bash
curl --silent --location \
"https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" \
--output /tmp/eksctl.tar.gz
```

### Extract eksctl

```bash
tar -xzf /tmp/eksctl.tar.gz -C /tmp
```

### Move eksctl to `/usr/local/bin`

```bash
sudo mv /tmp/eksctl /usr/local/bin/eksctl
```

### Give execute permission

```bash
sudo chmod +x /usr/local/bin/eksctl
```

### Verify installation

```bash
eksctl version
```

---

# ☸️ 3. Install kubectl

`kubectl` is the Kubernetes command-line tool used to communicate with the Kubernetes cluster.

### Download kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

### Download checksum

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
```

### Verify kubectl

```bash
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
```

Expected output:

```text
kubectl: OK
```

### Install kubectl

```bash
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

### Verify installation

```bash
kubectl version --client
```

### Remove downloaded files

```bash
rm -f kubectl kubectl.sha256
```

---

# ☁️ 4. Install AWS CLI

AWS CLI allows you to interact with AWS services from the command line.

### Install required packages

```bash
sudo apt install -y unzip curl
```

### Download AWS CLI

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" \
-o "awscliv2.zip"
```

### Extract AWS CLI

```bash
unzip -q awscliv2.zip
```

### Install AWS CLI

```bash
sudo ./aws/install
```

### Verify installation

```bash
aws --version
```

### Remove installation files

```bash
rm -rf aws awscliv2.zip
```

---

# 🔐 5. Configure AWS CLI

Configure your AWS credentials:

```bash
aws configure
```

You will be asked for:

```text
AWS Access Key ID:
AWS Secret Access Key:
Default region name:
Default output format:
```

For example:

```text
Default region name: ap-south-1
Default output format: json
```

---

# ✅ 6. Verify AWS Configuration

Run:

```bash
aws sts get-caller-identity
```

If the configuration is correct, AWS will return information about your AWS account and identity.

---

# 🚀 7. Create an Amazon EKS Cluster

Create an EKS cluster using `eksctl`:

```bash
eksctl create cluster \
  --name my-eks-cluster \
  --region ap-south-1 \
  --version 1.35 \
  --nodegroup-name my-nodes \
  --node-type t3.medium \
  --nodes 2
```

### Parameters Explained

| Parameter          | Description            |
| ------------------ | ---------------------- |
| `--name`           | EKS cluster name       |
| `--region`         | AWS region             |
| `--version`        | Kubernetes version     |
| `--nodegroup-name` | Worker node group name |
| `--node-type`      | EC2 instance type      |
| `--nodes`          | Number of worker nodes |

> ⏳ EKS cluster creation can take several minutes because AWS creates the control plane, networking, IAM resources, and worker nodes.

---

# 🔗 8. Configure kubectl

After the EKS cluster is created, configure `kubectl` to communicate with it:

```bash
aws eks update-kubeconfig \
  --region ap-south-1 \
  --name my-eks-cluster
```

You should see a message similar to:

```text
Added new context arn:aws:eks:ap-south-1:ACCOUNT_ID:cluster/my-eks-cluster
```

---

# 🔍 9. Verify EKS Cluster

## Check Worker Nodes

```bash
kubectl get nodes
```

Example:

```text
NAME                                           STATUS   ROLES    AGE   VERSION
ip-192-168-1-10.ap-south-1.compute.internal   Ready    <none>   5m    v1.35.x
ip-192-168-1-11.ap-south-1.compute.internal   Ready    <none>   5m    v1.35.x
```

---

## Check All Pods

```bash
kubectl get pods -A
```

---

## Check Cluster Information

```bash
kubectl cluster-info
```

---

## Run All Verification Commands

You can also execute them separately:

```bash
kubectl get nodes
```

```bash
kubectl get pods -A
```

```bash
kubectl cluster-info
```

> ⚠️ Do not write multiple `kubectl` commands on the same line without separators. Run each command separately.

---

# 🗑️ 10. Delete the EKS Cluster

When you finish your lab, delete the cluster to avoid unnecessary AWS charges:

```bash
eksctl delete cluster \
  --name my-eks-cluster \
  --region ap-south-1
```

### Verify Cluster Deletion

You can check whether the cluster still exists:

```bash
eksctl get cluster --region ap-south-1
```

You can also check through AWS CLI:

```bash
aws eks list-clusters --region ap-south-1
```

If the cluster is deleted, `my-eks-cluster` should no longer appear.

---

# 📚 Kubernetes Learning Commands

After your EKS cluster is ready, some useful commands are:

```bash
kubectl get nodes
kubectl get pods
kubectl get pods -A
kubectl get deployments
kubectl get services
kubectl get namespaces
kubectl get all
```

### Create a Kubernetes Resource

```bash
kubectl apply -f filename.yaml
```

### Delete a Kubernetes Resource

```bash
kubectl delete -f filename.yaml
```

### Describe a Resource

```bash
kubectl describe pod <pod-name>
```

### View Pod Logs

```bash
kubectl logs <pod-name>
```

---

# 🎯 Learning Roadmap

This repository can be used to practice the following Kubernetes topics:

* ☸️ Kubernetes Pods
* 📦 Deployments
* 🔄 ReplicaSets
* ⚙️ DaemonSets
* 🌐 Kubernetes Services
* 🔐 ConfigMaps & Secrets
* 💾 PV & PVC
* 🗄️ EBS & EFS
* 🏷️ Namespaces
* 🧩 StatefulSets
* 🔗 Inter-Pod Communication
* 🔌 Intra-Pod Communication
* 🚪 Ingress
* ☁️ Amazon EKS
* 🐳 Docker
* 🔄 CI/CD

---

## 👨‍💻 Author

**Amit Kannojia**

⭐ If this repository helps you learn Kubernetes or AWS EKS, consider giving it a star!



