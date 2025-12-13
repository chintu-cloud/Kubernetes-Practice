

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
