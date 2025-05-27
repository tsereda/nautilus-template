# Nautilus Usage Guide & Template

A template repository for running machine learning projects on the Nautilus cluster using Kubernetes. This example uses [pytorch-CycleGAN-and-pix2pix](https://github.com/junyanz/pytorch-CycleGAN-and-pix2pix) but can be adapted for any ML project.

## About Nautilus & Kubernetes

**Nautilus** is part of the National Research Platform (NRP), a national resource sharing program that provides researchers access to distributed computing infrastructure across multiple universities. It uses **Kubernetes**, an orchestration system that manages containerized applications across clusters of machines, allowing you to run your ML jobs on shared GPU resources.



## Important: Read First

**Nautilus is a shared environment with strict rules and bannable offenses.**

- Only request the resources you need
- Free up resources you don't use  
- Jobs must always self-terminate
- **DELETE UNUSED PODS** (wasting resources can get the namespace banned!)

Read the [Cluster Policies](https://docs.nationalresearchplatform.org/userdocs/running/policies/) before use.

## Template Files

- `pvc.yml` - Persistent storage for your data
- `data_pod.yml` - Download & preprocess data
- `train_pod.yml` - Training with 'sleep infinity' at end for debugging, max 2 non-A100 GPUs and 6hr runtime
- `train_job.yml` - Self-terminating training
- `aliases.md` - Helpful kubectl shortcuts

## Getting Started

### 1. Access Nautilus

1. **Login** to https://portal.nrp-nautilus.io/
   - Use your `@coyotes.usd.edu` email
   - Select "CILogon" → "University of South Dakota"

2. **Have your PI create an account** (same process)

3. **Request namespace** at https://element.nrp-nautilus.io/#/room/#general:matrix.nrp-nautilus.io
   - PI should message: *"Hello, I am a professor at the University of South Dakota, may I please be made a namespace admin?"*

4. **PI creates namespace** at https://portal.nrp-nautilus.io/ → 'Namespace Manager'
   - Name must start with "gp-engine" for grant tracking
   - PI adds you as a user

5. **Verify access**: Portal → "Namespace Manager" → check namespace field

### 2. Set Up Kubernetes

1. **Install kubectl**: https://kubernetes.io/releases/download/

2. **Download config**: https://portal.nrp-nautilus.io/ → 'Get Config'

3. **Place config** in `~/.kube/config` 
   - Create the directory if it doesn't exist
   - `~` means your home directory

4. **Verify access**: 
   ```bash
   kubectl config get-contexts
   # You should see your namespace
   ```

5. **Set up aliases** (optional but recommended): See [aliases.md](aliases.md)

## Usage Workflow

### Step 1: Create Storage
```bash
kubectl apply -f pvc.yml
kubectl get pvc #Verify the status of your pvc is 'Bound'

#With Aliases
ka pvc.yml
kgp
```

### Step 2: Download Data
```bash
kubectl apply -f data_pod.yml
# Data pod automatically downloads horse2zebra dataset
watch kubectl get pods # Wait for pod status 'Running' (Ctrl+C to cancel)
kubectl logs -f cyclegan-data-pod  # Monitor download progress

# For manual downloads or other data:
kubectl exec -it cyclegan-data-pod -- bash
# Inside the pod, use gdown, wget, curl, or other download utility
# Clean up when done
kubectl delete pod cyclegan-data-pod

#With Aliases
ka data_pod.yml
wkgp
kl cyclegan-data-pod 
ke cyclegan-data-pod -- bash
k delete pod cyclegan-data-pod
```

### Step 3a: Interactive Training
```bash
kubectl apply -f train_pod.yml
kubectl exec -it cyclegan-train-pod -- bash

# Inside the training pod - GPU available!
cd /app/pytorch-CycleGAN-and-pix2pix
python train.py --dataroot /data/datasets/horse2zebra --name test_run --model cycle_gan --n_epochs 5

# Clean up when done
kubectl delete pod cyclegan-train-pod
```

### Step 3b: Automated Training
```bash
kubectl apply -f train_job.yml
kubectl logs -f job/cyclegan-train-job  # Monitor progress
# Job exits automatically when complete
```

**data_pod.yml** - Data preparation (CPU only)
- Downloads datasets efficiently without wasting GPU resources
- Handles preprocessing and data validation
- Multiple people can reuse the same prepared data

**train_pod.yml** - Interactive development (GPU access)
- Experiment with hyperparameters
- Debug training code
- Quick test runs and validation

**train_job.yml** - Production training (GPU access)
- Fully automated training runs
- No manual interaction required
- Saves results and exits cleanly (Nautilus compliant)

## Customizing This Template

1. **Fork this repository**

2. **Edit `pvc.yml`:**
   - Change storage name and size

3. **Edit `data_pod.yml`:**
   - Update git repo URL for your project
   - Modify data download commands
   - Add custom preprocessing steps

4. **Edit `train_pod.yml` and `train_job.yml`:**
   - Change pod/job names
   - Update git repo URL  
   - Adjust resource requests (remember: limits = 1.2x requests)
   - Modify training commands for your model

## Resource Guidelines

**Small jobs:**
```yaml
requests: {memory: 8Gi, cpu: "4", nvidia.com/gpu: "1"}
limits: {memory: 10Gi, cpu: "5", nvidia.com/gpu: "1"}  # 1.2x requests
```

**Medium jobs:**
```yaml
requests: {memory: 20Gi, cpu: "10", nvidia.com/gpu: "1"}
limits: {memory: 24Gi, cpu: "12", nvidia.com/gpu: "1"}
```

**GPU Types:**
- **GTX, T4, V100**: Available in regular pods and jobs (6 hour limit for pods)
- **A100**: Requires special job queue and access approval

**Recommended workflow:** Test your code with train_pod.yml first, then use train_job.yml for longer runs, and only request A100 access after confirming your job works properly.

## Common Commands

```bash
# Monitor
kubectl get pods
kubectl logs <pod-name> -f
kubectl describe pod <pod-name>

# Interact
kubectl exec -it <pod-name> -- bash

# File transfer
kubectl cp <pod-name>:/path/to/file ./local-file
kubectl cp ./local-file <pod-name>:/path/to/file

# Clean up
kubectl delete pod <pod-name>
```

## Troubleshooting

**Pod won't start:**
```bash
kubectl describe pod <pod-name>
kubectl get events
```

**Authentication issues:**
- Redownload config from https://portal.nrp-nautilus.io/
- Replace `~/.kube/config`

## Additional Resources

- [Kubectl Aliases](aliases.md) - Speed up your workflow
- [Cluster Policies](https://docs.nationalresearchplatform.org/userdocs/running/policies/)
- [Nautilus Documentation](https://docs.nationalresearchplatform.org/)
- [Matrix Chat Support](https://element.nrp-nautilus.io/#/room/#general:matrix.nrp-nautilus.io)