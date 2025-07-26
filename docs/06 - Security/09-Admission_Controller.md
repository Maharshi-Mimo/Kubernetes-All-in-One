# Kubernetes Admission Controllers

## What is an Admission Controller?

Admission controllers are plugins that intercept requests to the Kubernetes API server before the persistence of an object but after the request is authenticated and authorized. These controllers can be thought of as gatekeepers that help enforce security policies, resource management, and various other operational requirements.

## Types of Admission Controllers

There are two types of admission controllers:
1. **Validating Admission Controllers**: These can reject requests
2. **Mutating Admission Controllers**: These can modify requests

## Common Admission Controllers

Some important admission controllers include:

- **NamespaceLifecycle**: Prevents deletion of system namespaces
- **LimitRanger**: Enforces resource limits per namespace
- **ServiceAccount**: Implements automation around service accounts
- **ResourceQuota**: Enforces resource quotas per namespace
- **PodSecurityPolicy**: Controls security-sensitive aspects of pod specification

## How to Enable/Disable Admission Controllers

To enable admission controllers, use the `--enable-admission-plugins` flag:

```yaml
--enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount
```

To disable specific controllers:

```yaml
--disable-admission-plugins=PodSecurityPolicy
```

## Custom Admission Controllers

You can create custom admission controllers using:
1. **Webhook Admission Controllers**: External HTTP callbacks
2. **Dynamic Admission Control**: Runtime-configurable controllers

Example of a Webhook Configuration:

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: pod-policy.example.com
webhooks:
  - name: pod-policy.example.com
    rules:
      - apiGroups: [""]
        apiVersions: ["v1"]
        operations: ["CREATE"]
        resources: ["pods"]
```

## Best Practices

1. Always enable essential security admission controllers
2. Use PodSecurityPolicy or alternatives for pod security
3. Implement custom controllers for organization-specific policies
4. Monitor admission controller performance
5. Maintain documentation of enabled controllers

## Common Use Cases

- Enforcing security policies
- Resource management
- Configuration validation
- Automated resource modification
- Compliance requirements

## Troubleshooting

Common issues and solutions:
1. API server logs for admission errors
2. Webhook configuration issues
3. Performance impact monitoring
4. Certificate management for webhooks

## References

- [Kubernetes Documentation](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)
- [Using Admission Controllers](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#what-does-each-admission-controller-do)
