# Kubernetes Secret Management

## Overview
Kubernetes Secrets are objects that store sensitive information such as passwords, tokens, or keys. They allow you to separate sensitive data from your application code and configuration.

## Types of Secrets
1. **Generic**: Created from files, directories, or literal values
2. **docker-registry**: Used for pulling images from private Docker registries
3. **tls**: Used for storing TLS certificates
4. **service-account-token**: Used for service account authentication

## Creating Secrets

### Using Command Line
```bash
# Create a generic secret
kubectl create secret generic db-credentials \
  --from-literal=username=admin \
  --from-literal=password=s3cr3t

# Create secret from files
kubectl create secret generic ssl-certificate \
  --from-file=ssh-privatekey=~/.ssh/id_rsa
```

### Using YAML
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=     # base64 encoded 'admin'
  password: czNjcjN0    # base64 encoded 's3cr3t'
```

## Using Secrets

### As Environment Variables
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: mycontainer
    image: nginx
    env:
    - name: DATABASE_USER
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: username
```

### As Volume Mounts
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-vol-pod
spec:
  containers:
  - name: mycontainer
    image: nginx
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
  volumes:
  - name: secret-volume
    secret:
      secretName: mysecret
```

## Best Practices

1. **Encryption**: Enable encryption at rest for Secrets
2. **RBAC**: Implement strict RBAC policies for Secret access
3. **Rotation**: Regularly rotate Secret values
4. **Minimal Access**: Only mount Secrets to Pods that need them
5. **External Solutions**: Consider using external secret management solutions like:
   - HashiCorp Vault
   - AWS Secrets Manager
   - Azure Key Vault

## Security Considerations

1. Secrets are stored in base64 encoding (not encryption)
2. By default, Secrets are stored unencrypted in etcd
3. Enable encryption at rest using EncryptionConfiguration
4. Control Secret access using RBAC
5. Monitor Secret usage and access patterns

## Common Use Cases

1. Database credentials
2. API tokens
3. TLS certificates
4. SSH keys
5. Docker registry credentials

## Limitations

1. Maximum size of 1MB per Secret
2. Stored in etcd with potential replication
3. Available to all containers in a namespace by default
4. Base64 encoding is not encryption

## Integration with External Tools

1. **Sealed Secrets**: Encrypt Secrets for git storage
2. **External Secrets Operator**: Sync from external vaults
3. **SOPS**: Mozilla's Secret OPerationS for encrypted files
4. **Vault Operator**: Integration with HashiCorp Vault

Remember to always follow the principle of least privilege and regularly audit your Secret management practices.
