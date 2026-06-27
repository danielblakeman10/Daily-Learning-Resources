# 💻 Pillar 04 — Compute & Application Hosting

> **EC2, auto-scaling, and load balancing — the workhorse services that power the cloud.**

---

## 🎯 Learning Objectives

By the end of this pillar, you will be able to:

- [ ] Launch and manage EC2 instances with IAM roles and user data scripts
- [ ] Design Auto Scaling groups with target tracking and scheduled scaling
- [ ] Configure Application and Network Load Balancers for traffic distribution
- [ ] Implement EC2 Image Builder for automated AMI/pipeline builds
- [ ] Understand Spot Instances for cost optimization
- [ ] Deploy containerized apps with Fargate and App Runner
- [ ] Use Outposts for hybrid on-premises compute
- [ ] Implement health checks and auto-recovery for EC2

---

## 🔑 Key Services

| Service | Purpose | DevOps Relevance |
|---------|---------|------------------|
| **EC2** | Virtual servers in the cloud | Foundation of AWS compute |
| **Auto Scaling Groups** | Dynamic capacity management | Scales with demand, handles failures |
| **Elastic Load Balancing** | Traffic distribution across targets | Health checks, SSL termination |
| **EC2 Image Builder** | Automated AMI/pipeline creation | Immutable infrastructure, security scanning |
| **Spot Instances** | Discounted compute (up to 90%) | Cost optimization for fault-tolerant workloads |
| **Fargate** | Serverless containers | No EC2 management for containers |
| **App Runner** | Serverless web apps | Deploy from container/Dockerfile instantly |
| **AWS Outposts** | AWS infrastructure on-premises | Hybrid compute, data residency |
| **AMI (Amazon Machine Image)** | Custom server templates | Immutable infrastructure, versioning |

---

## 🔄 Real DevOps Workflow Mapping

### Workflow: Auto Scaling + Load Balancer

```
┌─────────────────────────────────────────────────────────────┐
│  Architecture Flow                                          │
│                                                             │
│  Internet → ALB → ASG (2-8 instances) → EC2 (app servers)  │
│                    │                                          │
│                    ├── Health Check: /health → 200 OK       │
│                    ├── Scale Up: CPU > 70% for 5 min         │
│                    └── Scale Down: CPU < 30% for 10 min      │
├─────────────────────────────────────────────────────────────┤
│  Terraform: ASG Configuration                               │
│                                                             │
│  resource "aws_autoscaling_group" "web" {                   │
│    desired_capacity     = 2                                 │
│    max_size             = 8                                 │
│    min_size             = 2                                 │
│    target_group_arns    = [aws_lb_target_group.web.arn]     │
│    health_check_type    = "ELB"                             │
│                                                             │
│    launch_template {                                       │
│      id      = aws_launch_template.web.id                   │
│      version = "$Latest"                                    │
│    }                                                        │
│                                                             │
│    tag {                                                   │
│      key                 = "Name"                           │
│      value               = "web-tier"                       │
│      propagate_at_launch = true                             │
│    }                                                        │
│  }                                                          │
│                                                             │
│  resource "aws_autoscaling_policy" "scale_up" {              │
│    name                   = "scale-up-policy"               │
│    scaling_adjustment     = 1                               │
│    cooldown               = 300                             │
│    autoscaling_group_name = aws_autoscaling_group.web.id    │
│  }                                                            │
│                                                             │
│  resource "aws_cloudwatch_metric_alarm" "high_cpu" {        │
│    alarm_name          = "high-cpu"                         │
│    comparison_operator = "GreaterThanThreshold"             │
│    evaluation_periods  = "2"                                │
│    metric_name         = "CPUUtilization"                   │
│    namespace           = "AWS/EC2"                          │
│    period              = "120"                              │
│    statistic           = "Average"                          │
│    threshold           = "70"                               │
│    alarm_actions       = [aws_autoscaling_policy.scale_up.arn]│
│  }                                                            │
└─────────────────────────────────────────────────────────────┘
```

### Workflow: EC2 Auto Scaling with User Data

```bash
#!/bin/bash
# user-data.sh — Bootstrap EC2 instance

# Install application
curl -fsSL https://releases.hashicorp.com/terraform/1.6.0/terraform_1.6.0_linux_amd64.zip \
  -o /tmp/terraform.zip
unzip /tmp/terraform.zip -d /usr/local/bin/

# Install monitoring agent
curl -fsSL https://amazoncloudwatch-agent.amazonaws.com/linux_amd64/latest/autoconfigure \
  -o /tmp/cwa.sh
bash /tmp/cwa.sh --region us-east-1

# Apply IAM instance profile via EC2 role (already configured)
# Access secrets via SSM Parameter Store (no credentials in user data)
aws ssm get-parameter --name "/app/database-url" --with-decryption --query Parameter.Value --output text

# Start application
nohup /opt/app/server > /var/log/app.log 2>&1 &
echo "Application started at $(date)"
```

---

## 📚 Recommended Resources

### 📖 AWS Documentation

- [EC2 User Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html)
- [Auto Scaling User Guide](https://docs.aws.amazon.com/autoscaling/ec2/userguide/auto-scaling-groups.html)
- [ELB User Guide](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html)
- [EC2 Image Builder User Guide](https://docs.aws.amazon.com/image-builder/latest/userguide/what-is-image-builder.html)

### 🏗️ Well-Architected Framework

- [Compute Pillar](https://docs.aws.amazon.com/wellarchitected/latest/compute-pillar/welcome.html)
- [Auto Scaling Best Practices](https://docs.aws.amazon.com/wellarchitected/latest/compute-pillar/auto-scaling-best-practices.html)

### 🛠️ Hands-On Tutorials

- [Auto Scaling Workshop](https://catalog.workshops.aws/autoscaling)
- [ELB Configuration Guide](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-tasks.html)
- [EC2 Image Builder Walkthrough](https://docs.aws.amazon.com/image-builder/latest/userguide/image-builder-getting-started.html)

### 📄 Whitepapers

- [AWS Compute Optimizer](https://d1.awsstatic.com/whitepapers/aws-compute-optimization.pdf)

---

## 🏋️ Practical Exercises

### Exercise 1: Web Tier Auto Scaling
1. Create an ASG with ALB in front (2-8 instances across 2 AZs)
2. Set up target tracking scaling policy (CPU target 60%)
3. Deploy a web app using user data scripts
4. Generate load (ab or wrk) and watch it scale up
5. Verify instances are removed when load decreases

### Exercise 2: EC2 Image Builder Pipeline
1. Define an EC2 Image Builder pipeline
2. Add components: OS patches, application install, security scan
3. Build a custom AMI with your app pre-installed
4. Use the AMI in an ASG — verify instances launch with the image
5. Set up automatic weekly AMI updates

### Exercise 3: Spot Instances for Batch Processing
1. Create an ASG with spot capacity (60% of desired capacity)
2. Set up on-demand as the fallback
3. Deploy a batch processing workload (data transformation)
4. Monitor spot interruption events
5. Compare cost on-demand vs spot

### Exercise 4: Fargate + ALB Deployment
1. Create an ECS cluster with Fargate
2. Define a task definition with a Docker container
3. Create a service with ALB integration
4. Deploy a sample application (e.g., sample Node.js app)
5. Test blue/green deployment via ECS deployment controller

---

## 💡 Pro Tips

- **Use AMIs, not user data, for production** — AMIs are immutable and auditable; user data is ephemeral
- **Spot Instances + ASG = cheapest compute** — Combine for 60-90% savings on fault-tolerant workloads
- **Health checks > auto-recovery** — Let ALB detect unhealthy instances before EC2 does
- **Launch templates, not launch configs** — Launch templates support versioning and newer features
- **Use SSM Session Manager instead of SSH** — No key management, full audit trail, no bastion required

---

## ✅ Progress Checklist

- [ ] Completed learning objectives
- [ ] Deployed EC2 with user data bootstrapping
- [ ] Created Auto Scaling group with scaling policies
- [ ] Configured ALB/NLB with health checks
- [ ] Built EC2 Image Builder pipeline
- [ ] Deployed spot instances with ASG
- [ ] Deployed ECS/Fargate with ALB
- [ ] Completed all practical exercises
- [ ] Documented learnings in personal notes
