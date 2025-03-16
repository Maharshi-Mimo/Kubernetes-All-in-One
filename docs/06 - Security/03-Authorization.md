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

### Default ClusterRoles in Kubernetes

Kubernetes comes with several pre-defined ClusterRoles that serve specific purposes:

#### 1. Core ClusterRoles

| ClusterRole | Purpose | Typical Users |
|-------------|---------|---------------|
| `cluster-admin` | Superuser access, full control over every resource | Cluster administrators |
| `admin` | Full access within a namespace | Namespace administrators |
| `edit` | Read/write access to most resources | Developers |
| `view` | Read-only access to most resources | Helpdesk, readonly users |

#### 2. Technology-Specific ClusterRoles

```yaml
# Examples of specialized ClusterRoles
- system:node  # Used by kubelets
- system:kube-scheduler  # Used by scheduler
- system:kube-controller-manager  # Used by controller manager
- system:certificates.k8s.io:certificatesigningrequests:nodeclient  # For CSR approval
```

#### 3. Common Use Cases

1. **cluster-admin**
   ```yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: ClusterRoleBinding
   metadata:
     name: ops-cluster-admin
   subjects:
   - kind: Group
     name: ops-team
     apiGroup: rbac.authorization.k8s.io
   roleRef:
     kind: ClusterRole
     name: cluster-admin
     apiGroup: rbac.authorization.k8s.io
   ```

2. **view** (Read-only access)
   ```yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: RoleBinding
   metadata:
     name: monitoring-view
     namespace: production
   subjects:
   - kind: Group
     name: monitoring-team
     apiGroup: rbac.authorization.k8s.io
   roleRef:
     kind: ClusterRole
     name: view
     apiGroup: rbac.authorization.k8s.io
   ```

#### Best Practices for ClusterRole Usage

1. **Default Role Assignment**
   - Use `view` for read-only access
   - Use `edit` for developer access
   - Use `admin` for namespace owners
   - Restrict `cluster-admin` to minimal users

2. **Aggregation Rules**
   ```yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: ClusterRole
   metadata:
     name: monitoring
     labels:
       rbac.authorization.k8s.io/aggregate-to-view: "true"
   rules:
   - apiGroups: ["monitoring.coreos.com"]
     resources: ["servicemonitors"]
     verbs: ["get", "list", "watch"]
   ```

3. **Security Guidelines**
   - Avoid direct cluster-admin assignments
   - Use namespace-scoped roles when possible
   - Regularly audit ClusterRole bindings
   - Document custom ClusterRole creation

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
