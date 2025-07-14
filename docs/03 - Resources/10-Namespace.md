# Kubernetes Namespaces

## Purpose
Namespaces are a fundamental concept in Kubernetes that provides a mechanism for isolating groups of resources within a single cluster. They help in organizing resources, managing access control, and preventing naming conflicts between different teams or projects sharing the same cluster.

## Use Cases
- Multi-tenant environments where multiple teams share a cluster
- Environment separation (dev, staging, prod) within the same cluster
- Resource quotas management at a group level
- Access control for different teams or projects
- Organizing resources for better clarity and management

## Namespace vs Virtual Clusters
Unlike Virtual Clusters which provide complete isolation with dedicated control planes, Namespaces:
- Share the same control plane
- Provide logical separation only
- Cannot isolate cluster-level resources
- Have less overhead and are easier to manage
- Are built into Kubernetes core

## Features
1. **Resource Isolation**
   - Scope for names of resources
   - Basis for authorization and access control
   - Target for resource quotas

2. **Resource Quotas**
   - CPU and memory limits
   - Storage quotas
   - Object count limits

3. **Network Policies**
   - Network isolation between namespaces
   - Traffic control within namespace

## Example

```yaml
# Create a namespace
apiVersion: v1
kind: Namespace
metadata:
  name: development

---
# Deploy resources in the namespace
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  namespace: development
spec:
  containers:
  - name: myapp
    image: nginx:latest
```

## Best Practices

1. **Naming Conventions**
   - Use meaningful, clear names
   - Follow consistent naming patterns
   - Avoid special characters

2. **Resource Organization**
   - Group related resources together
   - Separate different environments
   - Use labels for additional organization

3. **Access Control**
   - Implement RBAC policies
   - Limit administrative access
   - Use service accounts appropriately

4. **Resource Quotas**
   - Set appropriate limits
   - Monitor resource usage
   - Plan for scaling

5. **Default Namespace**
   - Avoid using the default namespace
   - Create explicit namespaces for workloads
   - Document namespace purpose

6. **Network Policies**
   - Define clear ingress/egress rules
   - Implement zero-trust network policies
   - Review and update regularly

## Common Commands

```bash
# Create namespace
kubectl create namespace myspace

# List namespaces
kubectl get namespaces

# Set default namespace for context
kubectl config set-context --current --namespace=myspace

# Delete namespace (and all resources in it)
kubectl delete namespace myspace
```

## Limitations
- Cluster-wide resources cannot be namespaced
- Some resources must be in default namespace
- Cannot nest namespaces
- DNS names are affected by namespace names
