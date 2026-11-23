## Instruction how to reach ArgoCD UI

To reach ArgoCD interface and get admin password use official ArgoCD documentation:  or follow next steps:

1. Install ArgoCLI
```
brew install argocd
```
2. Get password:
```
argocd admin initial-password -n argocd 
```

3. Turn on port forwarding to reach UI in the browser:
```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
