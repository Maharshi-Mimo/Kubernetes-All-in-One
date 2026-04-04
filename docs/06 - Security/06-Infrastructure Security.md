# Kubernetes Infrastructure Security

## Node Security Hardening

### Host Security
- Keep the host OS updated with latest security patches
- Implement secure boot mechanisms
- Enable SELinux/AppArmor
- Use Container-Optimized OS (COS) where possible
- Regularly audit system logs

### Container Runtime Security
- Use containerd or CRI-O with security configurations
- Enable seccomp profiles
- Configure appropriate Linux capabilities
- Implement pod security standards

## Control Plane Security Measures

### API Server Security
- Enable RBAC authentication
- Use TLS certificates for all components
- Configure admission controllers
- Implement audit logging
- Set appropriate authentication methods

### Controller Manager Security
- Configure service account credentials
- Set appropriate RBAC permissions
- Enable secure port configurations

### Scheduler Security
- Implement pod security policies
- Configure resource quotas
- Set node selectors and affinity rules

## Network Security Policies

### Pod-to-Pod Communication
- Implement default deny policies
- Define explicit ingress/egress rules
- Use network policy selectors
- Isolate namespaces

Example Network Policy:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

### External Access Control
- Configure load balancer security
- Implement ingress controllers with TLS
- Use external firewall rules
- Set up WAF for ingress traffic

## Securing etcd

### Encryption and Access
- Enable encryption at rest
- Use TLS certificates for etcd communication
- Implement strong authentication
- Regular backup and disaster recovery plans

### Best Practices
- Run etcd on dedicated nodes
- Configure proper file permissions
- Regular security audits
- Monitor etcd metrics

### Example etcd Configuration
```yaml
etcd:
  encryption:
    - resources:
        - secrets
      providers:
        - aescbc:
            keys:
              - name: key1
                secret: <base64-encoded-key>
```

## Additional Security Measures

### Regular Security Tasks
- Audit logging review
- Certificate rotation
- Security patch management
- Vulnerability scanning
- Compliance checks

### Monitoring and Alerting
- Set up security monitoring
- Configure alerts for suspicious activities
- Implement intrusion detection
- Regular security assessments
