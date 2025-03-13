# Kubernetes Authorization

## Overview
Authorization in Kubernetes determines whether a user has permission to perform specific actions on resources. Unlike authentication (which verifies who you are), authorization decides what you can do.

## Authorization Modes

### 1. Node Authorization
- Specifically handles API requests made by kubelets
- Built-in authorization mode that grants permissions to kubelets
- Cannot be disabled as it's critical for cluster operations

### 2. RBAC (Role-Based Access Control)
The most commonly used authorization mode.

```yaml
# Example Role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

### Understanding RoleBinding vs ClusterRoleBinding

RoleBinding and ClusterRoleBinding are two distinct ways to bind roles to users in Kubernetes RBAC.

#### Key Differences

| Feature | RoleBinding | ClusterRoleBinding |
|---------|-------------|-------------------|
| Scope | Namespace-specific | Cluster-wide |
| Reference | Can bind Roles or ClusterRoles | Can only bind ClusterRoles |
| Use Case | Namespace-level access control | Cluster-wide access control |
| Creation Location | Must be created in a namespace | Created at cluster level |

#### Examples:

1. **RoleBinding Example** (Namespace-scoped):
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: development
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

2. **ClusterRoleBinding Example** (Cluster-wide):
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-binding
subjects:
- kind: User
  name: admin-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

#### When to Use Which?

- Use **RoleBinding** when:
  - Permissions should be limited to specific namespaces
  - Implementing namespace-level isolation
  - Managing team-specific resources

- Use **ClusterRoleBinding** when:
  - Managing cluster-wide resources (nodes, PVs)
  - Setting up cluster administrators
  - Configuring cluster-level service accounts

```yaml
# Example RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### 3. ABAC (Attribute-Based Access Control)
```json
{
  "apiVersion": "abac.authorization.kubernetes.io/v1beta1",
  "kind": "Policy",
  "spec": {
    "user": "alice",
    "namespace": "*",
    "resource": "pods",
    "readonly": true
  }
}
```

### 4. Webhook Mode
External service integration for authorization decisions.

```yaml
# Example Webhook Configuration
apiVersion: v1
kind: Config
preferences: {}
clusters:
- name: webhook
  cluster:
    server: https://authz.example.com/authorize
users:
- name: webhook
current-context: webhook-context
contexts:
- context:
    cluster: webhook
    user: webhook
  name: webhook-context
```

## Best Practices

1. **Principle of Least Privilege**
   - Grant minimal permissions required for tasks
   - Use namespaced roles over cluster roles when possible

2. **Role Aggregation**
   - Combine multiple roles for complex access patterns
   - Use ClusterRoles for cluster-wide resources

3. **Regular Audit**
   ```bash
   kubectl auth can-i list pods --namespace dev --as jane
   ```

4. **Service Account Management**
   ```yaml
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: app-service-account
     namespace: default
   ```

## Security Recommendations

1. Avoid using cluster-admin privileges for regular operations
2. Implement proper RBAC policies before deploying applications
3. Regular review and cleanup of RBAC rules
4. Use dedicated service accounts for each application
5. Enable audit logging for authorization decisions

## Troubleshooting

Common commands for authorization troubleshooting:

```bash
# Check user permissions
kubectl auth can-i create pods --namespace production

# View role bindings in a namespace
kubectl get rolebindings,clusterrolebindings --all-namespaces

# Describe specific role
kubectl describe role pod-reader -n default
```

## Additional Resources

- [Kubernetes Official Documentation - Authorization](https://kubernetes.io/docs/reference/access-authn-authz/authorization/)
- [RBAC Good Practices](https://kubernetes.io/docs/concepts/security/rbac-good-practices/)
- [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
