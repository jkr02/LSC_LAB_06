# LSC_LAB_06
Lab 6 of Large Scale Computing

# AWS setup
Configuration was done by aws EKS
- Created cluster
- Created nodes

![alt text](images/image-1.png)

# Kubernetes setup
- aws configure set default.region us-east-1
- aws configure set default.output table
- aws eks describe-cluster --region us-east-1 --name kroczek-cluster --query cluster.status
- aws eks --region us-east-1 update-kubeconfig --name kroczek-cluster

# Helm setup
- helm repo add stable https://charts.helm.sh/stable
- helm repo update
- helm install nfs-server stable/nfs-server-provisioner --set storageClass.name=nfs-client-provisioner

# Creating Persistent Volume Claim (PVC)
- kubectl apply -f pvc.yaml

# Deployment with HTTP server
- kubectl apply -f deployment.yaml

# Creating service with pods
- kubectl apply -f service.yaml

# Adding job
- kubectl apply -f job.yaml

# Testing HTTP server
- kubectl get svc web-server-service

![alt text](images/image.png)

# Architecture
```mermaid
graph TB
  subgraph AWS Cloud
    direction TB
    EKS[EKS Cluster]
    ELB[AWS ELB]
  end

  subgraph Kubernetes Cluster within EKS
    direction LR
    HelmNFS[NFS Provisioner]
    PVC[nfs-pvc PVC]
    Job[copy-content-job]
    Deployment[web-server Deployment Nginx]
    Service[web-server-service LB]
  end

  EKS -->|hosts| HelmNFS
  HelmNFS -->|provisions| PVC
  PVC --> Job
  Job -->|writes HTML| PVC
  PVC --> Deployment
  Deployment -->|serves content| Service
  Service -->|provisions| ELB
  ELB -->|external user traffic| User[Internet User]

```

- The NFS Server provides shared storage across the cluster.
- The NFS Provisioner dynamically creates PVs bound to PVCs using a StorageClass.
- A PVC connects both the Nginx Deployment and the Job and they can share files.
- The Job writes web content to the shared volume.
- The Nginx HTTP Server serves content.
- A Service exposes the Nginx pods and users can access the website.

