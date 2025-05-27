# Kubectl Aliases & Shortcuts

Speed up your Nautilus workflow with these helpful kubectl aliases.

## Linux/macOS Setup

**For Bash** (add to `~/.bashrc`):
```bash
# K8s Aliases & Bash Completion
alias k='kubectl'
alias kgp='kubectl get pods'
alias kpvc='kubectl get pvc'
alias ka='kubectl apply -f'
alias kl='kubectl logs'
alias kf='kubectl logs -f'
alias kd='kubectl describe'
alias ke='kubectl exec -it'

# Enable tab completion
if command -v kubectl &> /dev/null; then
  source <(kubectl completion bash)
fi
```

**For Zsh** (add to `~/.zshrc`):
```bash
# K8s Aliases & Zsh Completion  
alias k='kubectl'
alias kgp='kubectl get pods'
alias kpvc='kubectl get pvc'
alias ka='kubectl apply -f'
alias kl='kubectl logs'
alias kf='kubectl logs -f'
alias kd='kubectl describe'
alias ke='kubectl exec -it'

# Enable tab completion
if command -v kubectl &> /dev/null; then
  source <(kubectl completion zsh)
fi
```

After editing, run: `source ~/.bashrc` or `source ~/.zshrc`

## Windows Setup

**PowerShell** (add to your PowerShell profile):

Find your profile: `echo $PROFILE`

Add to that file:
```powershell
# K8s Aliases & PowerShell Completion
Set-Alias -Name k -Value kubectl
function kgp { kubectl get pods $args }
function kpvc { kubectl get pvc $args }
function ka { kubectl apply -f $args }
function kl { kubectl logs $args }
function kf { kubectl logs -f $args }
function kd { kubectl describe $args }
function ke { kubectl exec -it $args }

# Enable tab completion
if (Get-Command kubectl -ErrorAction SilentlyContinue) {
    kubectl completion powershell | Out-String | Invoke-Expression
}
```

After editing, restart PowerShell or run: `. $PROFILE`

## Usage Examples

With aliases, your commands become much shorter:

```bash
# Instead of: kubectl get pods
kgp

# Instead of: kubectl apply -f data_pod.yml  
ka data_pod.yml

# Instead of: kubectl exec -it cyclegan-train-pod -- bash
ke cyclegan-train-pod -- bash

# Instead of: kubectl logs cyclegan-train-pod -f
kf cyclegan-train-pod

# Instead of: kubectl describe pod cyclegan-data-pod
kd pod cyclegan-data-pod
```

## Nautilus Workflow with Aliases

```bash
# Create storage and launch data pod
ka pvc.yml
ka data_pod.yml

# Check status
kgp

# Connect to data pod
ke cyclegan-data-pod -- bash

# After data prep, clean up and launch training
k delete pod cyclegan-data-pod
ka train_pod.yml

# Connect to training pod
ke cyclegan-train-pod -- bash

# Monitor logs
kf cyclegan-train-pod
```

## Pro Tips

- **Tab completion works with aliases!** Type `k get p<TAB>` to autocomplete
- **Check everything quickly:** `kgp && kpvc`
- **Monitor pod startup:** `ka train_pod.yml && kgp && kf <pod-name>`