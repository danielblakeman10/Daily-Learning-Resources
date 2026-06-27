# 🐳 Pillar 09 — Containers & Kubernetes

> **EKS, ECS, and Fargate — orchestration at any scale, from simple microservices to complex Kubernetes clusters.**

---

## 🎯 Learning Objectives

By the end of this pillar, you will be able to:

- [ ] Deploy and manage ECS clusters with Fargate and EC2 launch types
- [ ] Design EKS clusters with managed node groups and self-managed nodes
- [ ] Implement IRSA (IAM Roles for Service Accounts) for Kubernetes authentication
- [ ] Deploy Helm charts to EKS with proper value overrides
- [ ] Set up container image scanning with ECR and CodeGuru
- [ ] Configure ALB/NLB controller for Kubernetes ingress
- [ ] Implement container security best practices (non-root, read-only root FS)
- [ ] Design multi-cluster and multi-region container architectures
- [ ] Use App Mesh for service mesh observability and traffic management

---

## 🔑 Key Services

| Service | Purpose | DevOps Relevance |
|---------|---------|------------------|
| **ECS** | Container orchestration | Simplified container management |
| **EKS** | Managed Kubernetes | Industry-standard Kubernetes at scale |
| **ECR** | Container registry | Secure, private Docker image storage |
| **Fargate** | Serverless compute for containers | No EC2 management for containers |
| **ELB (ALB/NLB)** | Load balancing | Traffic routing to container targets |
| **App Mesh** | Service mesh | Traffic management, observability |
| **EKS Add-ons** | Kubernetes plugins | Metrics Server, CoreDNS, VPC CNI |
| **Helm** | Kubernetes package manager | Templated Kubernetes deployments |

---

## 🔄 Real DevOps Workflow Mapping

### Workflow: ECS + Fargate Deployment

```
┌─────────────────────────────────────────────────────────────┐
│  ECS Fargate Architecture                                   │
│                                                             │
│  ECS Cluster (Fargate)                                     │
│  ├── Task Definition (Docker image, CPU, memory)          │
│  ├── Service (desired count, networking)                   │
│  ├── ALB (Application Load Balancer)                      │
│  │   └── Target Group → Fargate Tasks                    │
│  └── CloudWatch Logs (streaming log collection)            │
│                                                             │
│  ECR Repository → Docker build → push → ECS pull → deploy  │
└─────────────────────────────────────────────────────────────┘
```

### Workflow: EKS with IRSA

```yaml
# Kubernetes Deployment with IRSA

apiVersion: apps/v1
kind: Deployment
metadata:
  name: orders-service
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: orders-service
  template:
    metadata:
      labels:
        app: orders-service
      annotations:
        # IRSA: Attach IAM role to service account
        eks.amazonaws.com/role-arn: arn:aws:iam::ACCOUNT:role/orders-s3-access
    spec:
      serviceAccountName: orders-sa
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
      containers:
        - name: orders
          image: ACCOUNT.dkr.ecr.us-east-1.amazonaws.com/orders:latest
          ports:
            - containerPort: 8080
          securityContext:
            readOnlyRootFilesystem: true
            allowPrivilegeEscalation: false
          resources:
            requests:
              memory: "128Mi"
              cpu: "250m"
            limits:
              memory: "256Mi"
              cpu: "500m"
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 30
          readinessProbe:
            httpGet:
              path: /ready
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10
```

```hcl
# Terraform: EKS with IRSA

# Kubernetes Service Account
resource "kubernetes_service_account" "orders" {
  metadata {
    name      = "orders-sa"
    namespace = "production"
    annotations = {
      "eks.amazonaws.com/role-arn" = aws_iam_role.orders.arn
    }
  }
}

# IAM Role for Service Account
resource "aws_iam_role" "orders" {
  name = "orders-s3-access"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Federated = aws_iam_openid_connect_provider.eks.arn
      }
      Action = "sts:AssumeRoleWithWebIdentity"
      Condition = {
        StringEquals = {
          "${replace(aws_iam_openid_connect_provider.eks.url, "https://", "")}:sub" = "system:serviceaccount:production:orders-sa"
        }
      }
    }]
  })
}

# EKS OIDC Provider
resource "aws_iam_openid_connect_provider" "eks" {
  client_id_list  = ["sts.amazonaws.com"]
  thumbprint_list = [data.tls_certificate.eks.certificates[0].cert_pem_sha1_fingerprint]
  url             = aws_eks_cluster.main.identity[0].oidc[0].issuer
}
```

---

## 📚 Recommended Resources

### 📖 AWS Documentation

- [Amazon ECS User Guide](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/WhatIsECS.html)
- [Amazon EKS User Guide](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html)
- [Amazon ECR User Guide](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html)
- [IRSA Documentation](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html)
- [AWS Fargate Documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html)

### 🏗️ Well-Architected Framework

- [Container Best Practices](https://docs.aws.amazon.com/wellarchitected/latest/container-workload-pillar/welcome.html)
- [Security Pillar — Container Security](https://docs.aws.amazon.com/wellarchitected/latest/container-workload-pillar/container-security.html)

### 🛠️ Hands-On Tutorials

- [ECS Getting Started](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html)
- [EKS Workshop](https://eksworkshop.com/)
- [EKS Security Best Practices](https://docs.aws.amazon.com/eks/latest/userguide/security-considerations.html)
- [Helm Quick Start](https://helm.sh/docs/intro/quickstart/)

### 📄 Whitepapers

- [Running Microservices on AWS](https://d1.awsstatic.com/whitepapers/microservices-on-aws.pdf)
- [Securing Your Container Deployments on AWS](https://d1.awsstatic.com/whitepapers/security/containers-security-whitepaper.pdf)

---

## 🏋️ Practical Exercises

### Exercise 1: ECS + Fargate
1. Create an ECS cluster with Fargate launch type
2. Define a task with a public Docker image (nginx)
3. Create a service with ALB integration
4. Scale to 3 tasks and verify load distribution
5. Set up CloudWatch monitoring

### Exercise 2: EKS Cluster Deployment
1. Deploy an EKS cluster with managed node groups (2x m5.large)
2. Install kubectl and configure kubeconfig
3. Deploy a multi-container application (app + sidecar)
4. Set up ALB Ingress Controller for external access
5. Configure HPA (Horizontal Pod Autoscaler)

### Exercise 3: IRSA for EKS
1. Create an EKS cluster
2. Define a Kubernetes ServiceAccount
3. Create an IAM role with S3 read permissions
4. Annotate the ServiceAccount with the IAM role ARN
5. Deploy a pod that writes to S3 — verify it works without access keys

### Exercise 4: Helm Chart Deployment
1. Package a Kubernetes deployment as a Helm chart
2. Create values.yaml for dev, staging, and prod environments
3. Deploy to 3 EKS clusters with `helm install`
4. Use `helm upgrade --set` for configuration changes
5. Implement Helm rollback on failure

---

## 💡 Pro Tips

- **Prefer Fargate for most workloads** — No server management, pay per resource used
- **Use IRSA, not IAM roles for service accounts** — IAM roles for SA is being deprecated
- **Never embed credentials in containers** — Use IRSA, SSM, or Secrets Manager
- **Set resource requests and limits** — Prevent noisy neighbor, enable proper scheduling
- **Security groups for pods > network policies** — AWS Network Policies are still evolving
- **ECR lifecycle policies** — Automatically clean up old images to save storage costs

---

## ✅ Progress Checklist

- [ ] Completed learning objectives
- [ ] Deployed ECS cluster with Fargate
- [ ] Created and deployed ECS task + service
- [ ] Deployed EKS cluster with managed nodes
- [ ] Implemented IRSA for EKS
- [ ] Deployed Helm charts to EKS
- [ ] Configured ALB Ingress Controller
- [ ] Set up container security best practices
- [ ] Completed all practical exercises
- [ ] Documented learnings in personal notes
