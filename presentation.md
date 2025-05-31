---
marp: true
theme: default
paginate: true
backgroundColor: #fff
backgroundImage: url('https://marp.app/assets/hero-background.svg')
---

# Nautilus ML Template
## Running Machine Learning Projects on Kubernetes

---

## What is Nautilus?

- **National Research Platform (NRP)** distributed computing infrastructure
- **Kubernetes-based** orchestration across multiple universities
- **Shared GPU resources** for machine learning research
- **Containerized applications** for reproducable environments

### Key Benefits
- Access to distributed GPU clusters across universities
- Scalable computing resources
- Collaborative research environment
- **Free for academic research**

---

## Alternative: JupyterHub Platform

**Perfect for getting started quickly:**

- Web-based Jupyter notebooks - no Kubernetes knowledge needed
- Access at [jupyterhub-west.nrp-nautilus.io](https://jupyterhub-west.nrp-nautilus.io)
- Default: 5GB memory, upgradeable (ask in Matrix chat)
- Great for experimentation and learning
- No namespace required - just institutional login

**Use Kubernetes when you need:**
- Custom environments and dependencies
- More than 5GB memory
- Automated training pipelines

---

## Nautilus is a Collaborative Environment

### 🚀 **Shared Resources, Shared Success**

**Community Guidelines:**
- Be resource-conscious
- Monitor your jobs: GPU utilization should be >40% (theres a rule)

**Best Practices:**
- Delete unused resources promptly
- Start small, scale up
- Ask questions in matrix

---

## Resource Limits & Policies

### Pod Limits
- **Training Pods**: Max 2 non-A100 GPUs, 6-hour runtime
- Interactive development and debugging

### Job Limits  
- **Training Jobs**: Up to 8 GPUs, runs to completion
- Automated pipelines with self-termination

### Special Access
- **A100 GPUs**: Approval required

- Review [Request A100 Access](https://docs.nationalresearchplatform.org/userdocs/running/policies/)


---

## Setup Process: Account & Access

### 1. User Registration
- Log into https://portal.nrp-nautilus.io using a `@coyotes.usd.edu` email
- Select "CILogon" → "University of South Dakota"

### 2. PI Namespace Request and Configuration
PI contacts support: [Matrix Chat](https://element.nrp-nautilus.io/#/room/#general:matrix.nrp-nautilus.io)

Message: *"Hello, I am a professor at the University of South Dakota. May I please be made a namespace admin?"*

- PI creates namespace via Portal → 'Namespace Manager'
- Prefix with "gp-engine" for grant tracking
- PI adds users to namespace
---

## Setup Process: Kubernetes Config

### 1. Install kubectl
Follow instructions at: https://kubernetes.io/releases/download/

### 2. Download & Install Config
- Download from: https://portal.nrp.ai/authConfig
- Place at: `~/.kube/config`

### 3. Verify Installation
```bash
kubectl config get-contexts
```

---

## Workflow Overview

1. **Storage Provisioning** → Create persistent volume claim (PVC) for data
2. **Data Preparation** → Download and preprocess datasets in pod 
3. **Training** → Interactive development pod OR automated self-terminating job
4. **Cleanup** → Delete resources when finished

---


## Template

https://github.com/tsereda/nautilus-template

### Core Components

| Component | Purpose | Use Case |
|-----------|---------|----------|
| `pvc.yml` | Persistent storage for datasets/results | Data persistence |
| `data_pod.yml` | Data acquisition & preprocessing | CPU-only data work |
| `train_pod.yml` | Interactive development environment | Debugging & testing |
| `train_job.yml` | Automated training pipeline | Research runs |
---
## Step 1: Storage Provisioning



### Create Persistent Volume Claim
```bash
kubectl apply -f pvc.yml
kubectl get pvc  # Verify 'Bound' status
```

---

## Step 2: Data Preparation



### Launch Data Pod

```bash
kubectl apply -f data_pod.yml
watch kubectl get pods  # Monitor until 'Running'
kubectl logs -f cyclegan-data-pod  # Track progress
```
- Downloads datasets (horse2zebra example)
- **Multithreaded compression** for efficiency
- Timestamped storage to shared volume

### Debugging
```bash
kubectl describe cyclegan-data-pod # If pod not running
kubectl exec -it cyclegan-data-pod -- bash # Access pod
```

---

## Step 3: Interactive Development

### Training Pod for Development
```bash
kubectl apply -f train_pod.yml
watch kubectl get pods
kubectl logs -f cyclegan-train-pod
```

### Interactive Features
- GPU access for development
- Persistent session for debugging
- 6-hour runtime limit

```bash
kubectl top cyclegan-train-pod # Check CPU, Memory Usage
kubectl exec -it cyclegan-train-pod -- nvidia-smi # Check GPU Usage
```

---

## Step 4: Automated Research Runs

### Training Job for Research
```bash
kubectl apply -f train_job.yml
kubectl logs -f jobs/cyclegan-train-job
# Job terminates automatically when complete
```

### Job vs Pod Key Differences
- Higher memory/CPU allocation limits
- Automated execution without manual intervention
- **Require self-termination** upon completion

---

## GPU Access Levels

### Standard GPUs (Readily Available)
- **GTX Series, T4, V100** GPUs
- Available in standard queues
- **Multi-GPU training** supported (up to 8 GPUs)
- Can be challenging to find multi-GPU nodes

### A100 GPUs (Special Access Required)
- **High-performance** workloads (80GB memory)
- **Special queue** access required
- **Approval process** - demonstrate workflow first
- Request: [Request A100 Form](https://nrp.ai/documentation/userdocs/running/gpu-pods/#requesting-special-gpus)

---

## Customization Guide

### 1. Adapt for Your Project
- Copy YAML files to your repository

### 2. Update Data Pipeline
- Replace `data_pod.yml` with your data sources
- Adjust preprocessing scripts

### 3. Configure Training
- Update resource requirements
- Modify model parameters and training scripts
- Adjust for single vs multi-GPU needs

---

## Recommended Workflow

1. **Test** with `train_pod.yml` first
2. **Verify** everything works correctly  
3. **Scale** to `train_job.yml` for research runs
4. **Request A100** access after proving workflow

### Resource Considerations
- Start with 1-2 GPU(s), minimal CPUs and memory
- Monitor utilization (should be >40% GPU usage)
- Increase resources when needed

---

## Troubleshooting

### Pod Issues
```bash
kubectl describe pod <pod-name>  # Check events and status
kubectl logs <pod-name>          # View container logs
```

### Common Problems
- **Authentication**: Re-download config from portal
- **Resource constraints**: Check [documentation](https://docs.nationalresearchplatform.org/) for available resources
- **Node availability**: Some regions have limited multi-GPU nodes
- **Storage binding**: Verify storage class in your region

---

## Essential Commands

### Monitoring
```bash
kubectl get pods -o wide        # Pod status and locations
kubectl logs -f <pod-name>      # Follow logs in real-time
kubectl top pods                # Resource usage
```

### GPU Access
```bash
kubectl exec -it <pod-name> -- nvidia-smi    # Check GPU status
kubectl exec -it <pod-name> -- bash          # Shell access
```

### Cleanup
```bash
kubectl delete pod <pod-name>
kubectl delete job <job-name>
kubectl delete pvc <pvc-name>
```

---


## Getting Help & Community

### Support Channels
- **[Matrix Chat](https://element.nrp-nautilus.io/#/room/#general:matrix.nrp-nautilus.io)** - Primary support (very responsive!)
- **[Documentation](https://docs.nationalresearchplatform.org/)** - Comprehensive guides
- **Community** - Other researchers happy to help

### Pro Tips
- **Monitor resources** - respect shared environment
- **Start conversations** - "Has anyone tried X on Nautilus?"
- **Share solutions** - help others learn from your experience

---

## Additional Resources

### Documentation
- [Nautilus User Guide](https://docs.nationalresearchplatform.org/) - Complete documentation
- [Kubernetes Basics](https://kubernetes.io/docs/) - Learn more about K8s
- [Cluster Policies](https://docs.nationalresearchplatform.org/userdocs/running/policies/) - Important guidelines

### Community
- [Matrix Support Chat](https://element.nrp-nautilus.io/#/room/#general:matrix.nrp-nautilus.io)
- [Portal Access](https://portal.nrp-nautilus.io/) - Account management

---

## Getting Started & Citation

### To use Nautilus:
1. **Register** at [portal.nrp-nautilus.io](https://portal.nrp-nautilus.io)
2. **Get namespace access** (contact your PI)
3. **Install kubectl** and download config
4. **Try the template** with your own data

### How to Cite Nautilus
*This work was supported in part by National Science Foundation (NSF) awards CNS-1730158, ACI-1540112, ACI-1541349, OAC-1826967, OAC-2112167, CNS-2100237, CNS-2120019.*

**Questions?** Ask in [Matrix chat](https://element.nrp-nautilus.io/#/room/#general:matrix.nrp-nautilus.io) - community and admins are very helpful