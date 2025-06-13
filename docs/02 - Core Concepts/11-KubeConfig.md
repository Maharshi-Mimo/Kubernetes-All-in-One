# Common KubeConfig Operations

1. View current context:
```bash
kubectl config current-context
```

2. Switch context:
```bash
kubectl config use-context prod-context
```

3. Set namespace for a context:
```bash
kubectl config set-context --current --namespace=new-namespace
```

4. Add a new cluster:
```bash
kubectl config set-cluster new-cluster --server=https://new-cluster:6443
```

5. Add new credentials:
```bash
kubectl config set-credentials new-user --token=<bearer_token>
```

6. Create a new context:
```bash
kubectl config set-context new-context --cluster=new-cluster --user=new-user
```

7. View kubeconfig:
```bash
kubectl config view
```

8. Merge kubeconfig files:
```bash
KUBECONFIG=~/.kube/config:/path/to/new/config kubectl config view --flatten > ~/.kube/merged_config
```
