# Nautilus Usage Guide & Template

A template repository for running machine learning projects on the Nautilus cluster using Kubernetes. This example uses [pytorch-CycleGAN-and-pix2pix](https://github.com/junyanz/pytorch-CycleGAN-and-pix2pix) but can be adapted for any ML project.

## About Nautilus & Kubernetes

**Nautilus** is part of the National Research Platform (NRP), a national resource sharing program that provides researchers access to distributed computing infrastructure across multiple universities. It uses **Kubernetes**, an orchestration system that manages containerized applications across clusters of machines, allowing you to run ML jobs on shared GPU resources.

## Alternative: JupyterHub Platform

**For users who want a simpler entry point:**

- Web-based Jupyter notebooks - no Kubernetes knowledge needed
- Access at [jupyterhub-west.nrp-nautilus.io](https://jupyterhub-west.nrp-nautilus.io)
- Default: 5GB memory, upgradeable (ask in Matrix chat)
- Good for experimentation and learning
- No namespace required - just institutional login

## Prerequisites and Important Notes

**Nautilus is a shared environment with strict resource policies.**

- Request only the resources required for your workload
- Delete unused pvcs,pods,jobs
- Ensure all jobs terminate automatically
- **Delete unused pods immediately**

Review the [Cluster Policies](https://docs.nationalresearchplatform.org/userdocs/running/policies/) before beginning.

## Template Components

- `pvc.yml` - Persistent storage configuration for datasets and results
- `data_pod.yml` - Data acquisition and preprocessing pipeline
- `train_pod.yml` - Interactive development environment with GPU access (max 2 non-A100 GPUs, 6-hour runtime)
- `train_job.yml` - Automated training job with self-termination

## Setup Instructions

### 1. Nautilus Account Access

1. **User Registration**: Navigate to https://portal.nrp-nautilus.io/
   - Use your `@coyotes.usd.edu` email address
   - Select "CILogon" → "University of South Dakota"

2. **PI Account Creation**: Principal Investigator follows the same registration process

3. **Namespace Request**: PI contacts support at https://element.nrp-nautilus.io/#/room/#general:matrix.nrp-nautilus.io
   - Message: *"Hello, I am a professor at the University of South Dakota requesting namespace administrator privileges."*

4. **Namespace Configuration**: PI creates namespace via Portal → 'Namespace Manager'
   - Namespace should be prefixed with "gp-engine" for grant tracking compliance
   - PI adds users to the namespace

5. **Access Verification**: Confirm access via Portal → "Namespace Manager"

### 2. Kubernetes Configuration

1. **Install kubectl**: Follow instructions at https://kubernetes.io/releases/download/
2. **Download Configuration**: [Portal](https://portal.nrp.ai/authConfig) → 'Get Config'
3. **Install Configuration**: Place downloaded file at `~/.kube/config`
4. **Verify Installation**: 
   ```bash
   kubectl config get-contexts
   ```

## Workflow

### Storage Provisioning
```bash
kubectl apply -f pvc.yml
kubectl get pvc  # Verify cyclegan-data-pvc status shows 'Bound'
```

### Data Preparation
```bash
kubectl apply -f data_pod.yml
watch kubectl get pods  # Monitor until status shows 'Running'
kubectl logs -f cyclegan-data-pod  # Track download progress

# To shell into pod:
kubectl exec -it cyclegan-data-pod -- bash

# Cleanup:
kubectl delete pod cyclegan-data-pod

# Wait until the pod is fully terminated, do not use -force or it may corrupt the pvc
```

### Training Execution

**Interactive Development (Pod):**
```bash
kubectl apply -f train_pod.yml
watch kubectl get pods
kubectl logs -f cyclegan-train-pod
# The first epoch will take several minutes to complete
kubectl exec -it cyclegan-train-pod -- nvidia-smi  # Verify GPU allocation

# Access pod directly:
kubectl exec -it cyclegan-train-pod -- bash

# Cleanup:
kubectl delete pod cyclegan-train-pod
```

**Automated Training (Job):**
```bash
# train_job.yml is very simmilar to train_pod.yml, but with some syntactic differences, higher resource requests, and self-termination
kubectl apply -f train_job.yml
kubectl logs -f jobs/cyclegan-train-job
kubectl exec -it jobs/cyclegan-train-job -- nvidia-smi
# Job terminates automatically upon completion
kubectl delete job cyclegan-train-job
```

## Pods

**Data Pod (`data_pod.yml`)** - CPU-only data preparation
- Handles data download and preprocessing

**Training Pod (`train_pod.yml`)** - Interactive GPU-enabled development
- Persistent session for debugging

**Training Job (`train_job.yml`)** - Automated GPU-enabled production
- Fully autonomous training execution with automatic termination

## Customization

1. **Repository Adaptation**: Fork this repository for your project or copy .ymls to your repository
2. **Storage Configuration**: Modify `pvc.yml` for appropriate storage allocation
3. **Data Pipeline**: Update `data_pod.yml` with project-specific data sources and preprocessing
4. **Training Configuration**: Adapt `train_pod.yml` and `train_job.yml` with model-specific parameters and resource requirements

**GPU Resource Allocation:**
- **Standard GPUs** (GTX, T4, V100): Available in standard queues
- **A100 GPUs**: Require special queue access and approval

**Recommended Workflow:** Validate functionality using `train_pod.yml`, deploy production runs with `train_job.yml`, and request A100 access only after the job runs correctly.

**Request Namespace A100 GPU Access:** https://nrp.ai/documentation/userdocs/running/gpu-pods/#requesting-special-gpus

## Troubleshooting

**Pod Scheduling Issues:**
```bash
kubectl describe pod <pod-name>
```

**Authentication Problems:**
- Re-download configuration from https://portal.nrp-nautilus.io/
- Replace existing `~/.kube/config` file

## Additional Resources

- [Nautilus Cluster Policies](https://docs.nationalresearchplatform.org/userdocs/running/policies/)
- [Complete Nautilus Documentation](https://docs.nationalresearchplatform.org/)
- [Matrix Chat Support](https://element.nrp-nautilus.io/#/room/#general:matrix.nrp-nautilus.io)

**How to Cite:** This work was supported in part by National Science Foundation (NSF) awards CNS-1730158, ACI-1540112, ACI-1541349, OAC-1826967, OAC-2112167, CNS-2100237, CNS-2120019.