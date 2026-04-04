# ConfigMap

## Purpose
ConfigMap is a Kubernetes resource that allows you to decouple configuration artifacts from container image content to maintain portability. It stores non-confidential configuration data in key-value pairs that can be consumed by pods or other system components.

## Use Cases
- Application configuration files
- Command-line arguments
- Environment variables
- Port numbers
- Configuration for multiple environments (dev, staging, prod)
- Store proxy configurations
- Database connection strings (non-sensitive)

## ConfigMap vs Secret
| Feature | ConfigMap | Secret |
|---------|-----------|--------|
| Purpose | Store configuration data | Store sensitive data |
| Encoding | Plain text | Base64 encoded |
| Size Limit | 1MB | 1MB |
| Storage | Unencrypted | Encrypted (when encryption at rest is enabled) |
| Use Case | Non-sensitive configuration | Passwords, tokens, keys |
| Volume Type | configMap | secret |

## Features
1. **Multiple Creation Methods**
   - From literal values
   - From configuration files
   - From directories
   - From environment files

2. **Multiple Consumption Methods**
   - Environment variables
   - Command-line arguments
   - Configuration files in volumes

3. **Dynamic Updates**
   - ConfigMaps can be updated while running
   - Changes are reflected in mounted volumes
   - Note: Environment variables do not update automatically

## Example

### Creating a ConfigMap
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "mongodb://localhost:27017"
  app_mode: "development"
  config.yaml: |
    env: development
    log_level: debug
    api_port: 8080
```

### Using ConfigMap in Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-app
spec:
  containers:
  - name: web
    image: nginx
    env:
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database_url
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-config
```

## Best Practices

1. **Naming and Organization**
   - Use clear, descriptive names
   - Group related configurations in the same ConfigMap
   - Use namespace-specific ConfigMaps

2. **Version Control**
   - Store ConfigMaps in version control
   - Document all configuration options
   - Use labels for versioning

3. **Security**
   - Don't store sensitive data (use Secrets instead)
   - Implement RBAC for ConfigMap access
   - Regularly audit ConfigMap contents

4. **Management**
   - Keep ConfigMaps small and focused
   - Use separate ConfigMaps for different applications
   - Consider using Helm or Kustomize for managing ConfigMaps

5. **Updates and Rollouts**
   - Plan for configuration updates
   - Test configuration changes before deployment
   - Consider using rolling updates when changing configurations

6. **Monitoring and Validation**
   - Validate ConfigMap data before applying
   - Monitor ConfigMap usage and changes
   - Implement proper error handling in applications
