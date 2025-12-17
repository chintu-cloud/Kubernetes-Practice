

# Amazon EKS Cluster Setup Guide

### 🧱 Desired Diagram Structure (Textual Layout)

```
+-----------------------------+       +-----------------------------+
| IAM Role: EKS-role          |       | IAM Role: Node-role         |
| Policy: EKSClusterPolicy    |       | Policy: AdministratorAccess |
+-----------------------------+       +-----------------------------+
            |                                  |
            v                                  v
  +-----------------------------------------------------------+
  |                  Amazon EKS Control Plane                 |
  |  - API Server                                             |
  |  - etcd                                                   |
  |  - Scheduler                                              |
  |  - Controller Manager                                     |
  +-----------------------------------------------------------+
            |                                  |
            v                                  v
+-----------------------------+       +-----------------------------+
| Node Group: K8S             |       | Node Group: K8S             |
| EC2 Worker Node 1           |       | EC2 Worker Node 2           |
| - Kubelet                   |       | - Kubelet                   |
| - Pods                      |       | - Pods                      |
+-----------------------------+       +-----------------------------+
            |                                  |
            v                                  v
  +-----------------------------------------------------------+
  | Networking (VPC, Subnets, SG, Load Balancer)              |
  +-----------------------------------------------------------+
```

---

## 1. Create IAM Role for Cluster

**Step 1**  
- Select trusted entity: **AWS service**  
- Use case: **EKS cluster**

**Step 2**  
- Add permissions  
- Permission policies: **AmazonEKSClusterPolicy**

**Step 3**  
- Role name: **EKS-role**

➡️ Then create role

---

## 2. Create EKS Cluster (takes ~10 min)

**Step 1: Configure cluster**  
- Custom configuration  
- EKS Auto Mode: **OFF**  
- Cluster configuration: name = `cloud` (any name)  
- Cluster IAM role: **EKS-role** (select)  
- Kubernetes version: **1.34 (latest)**  
- Upgrade policy: **standard support**

**Step 2: Specify networking**  
- VPC: **default**  
- Subnet: **default**  
- SG: **default**  
- Cluster endpoint access: **public**

**Step 3: Configure observability**  
- Default (no changes)

**Step 4: Select add-ons**  
- Default (no changes)

**Step 5: Configure selected add-ons settings**  
- Default (no changes)

**Step 6: Review and create**

---

## 3. Create IAM Role for Node

**Step 1**  
- Select trusted entity: **AWS service**  
- Use case: **EC2**

**Step 2**  
- Add permissions  
- Permission policies: **AdministratorAccess**

**Step 3**  
- Role name: **Node-role**

➡️ Then create role

---

## 4. Add Node Group (takes ~30+ min)

Go to **Cluster → EKS → Compute → Node groups → Add**

**Step 1: Configure node group**  
- Name: **cloud-node**  
- Node IAM role: **Node-role** (select)  
- Instance type: **c7i-flex.large** (for low latency)

**Step 2: Set compute and scaling configuration**  
- Default (no changes)

**Step 3: Specify networking**  
- Default (no changes)

**Step 4: Review and create**

➡️ Then automatically creates **2 node servers**

---

## 5. Launch EC2 for Client

- Name: **client**  
- Instance type: **t3.micro**  
- VPC: **default**  
- SG: **default**

---
```
External Clients (Browser, API, CI/CD)
        |
     Ingress / LoadBalancer / NodePort / ClusterIP
        |
   Deployments & ReplicaSets
        |
        Pods
        |
     Containers
        |
   Worker Nodes (kubelet, kube-proxy)
        |
   Control Plane (etcd, kube-apiserver, scheduler, controller-manager)
```



---

# 📘 Kubernetes on AWS EKS – Setup Guide

This README documents the steps to install `kubectl`, configure AWS CLI, connect to an EKS cluster, and verify nodes.

---

## 🔧 Step 1: Install `kubectl`

Download the latest stable release:
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

Provide execution permission:
```bash
chmod +x kubectl
```

Move binary to system path:
```bash
mv kubectl /usr/local/bin/kubectl
```

Verify installation:
```bash
kubectl version --client
```

Output:
```
Client Version: v1.34.3
Kustomize Version: v5.7.1
```

---

## 🔧 Step 2: Configure AWS CLI

Run:
```bash
aws configure
```

Provide credentials:
```
AWS Access Key ID [None]: <YOUR_AWS_ACCESS_KEY>
AWS Secret Access Key [None]: <YOUR_AWS_SECRET_KEY>
Default region name [None]: us-east-1
Default output format [None]: json
```

---

## 🔧 Step 3: Update kubeconfig for EKS

Connect to your cluster:
```bash
aws eks update-kubeconfig --region us-east-1 --name cloud
```

Output:
```
Added new context arn:aws:eks:us-east-1:<ACCOUNT_ID>:cluster/cloud to /root/.kube/config
```

---

## 🔧 Step 4: Verify Cluster Nodes

Check nodes:
```bash
kubectl get nodes
```

Output:
```
NAME                           STATUS   ROLES    AGE     VERSION
ip-172-31-74-99.ec2.internal   Ready    <none>   6m40s   v1.34.2-eks-ecaa3a6
ip-172-31-8-15.ec2.internal    Ready    <none>   6m37s   v1.34.2-eks-ecaa3a6
```

---

## ✅ Conclusion

- Installed `kubectl` successfully.  
- Configured AWS CLI with credentials.  
- Connected to EKS cluster `cloud`.  
- Verified worker nodes are **Ready**.  

---


