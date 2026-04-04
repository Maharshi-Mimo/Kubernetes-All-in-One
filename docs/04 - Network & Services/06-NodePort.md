# NodePort Service in Kubernetes

## Features
- Exposes the service on each Node's IP at a static port
- Port range: 30000-32767 (default)
- Automatically creates a ClusterIP service
- Accessible from outside the cluster
- Each port can only be used by one service at a time

## Working Architecture
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
    - port: 80          # Service port
      targetPort: 8080  # Container port
      nodePort: 30007   # External port (optional)
```

### Flow of Traffic
1. Traffic enters through NodePort (e.g., 30007)
2. Routes to ClusterIP service
3. Forwards to pod(s) matching selector
4. Load balanced across pods

## Integration with Cloud Providers

### AWS
- Works with Security Groups
- Requires manual configuration of VPC settings
- Can be used with AWS Load Balancer

### GCP
- Functions with Firewall Rules
- Automatically opens required ports
- Compatible with GCP Load Balancer

### Azure
- Works with Network Security Groups
- Requires NSG rule configuration
- Can be used with Azure Load Balancer

## NodePort vs External Load Balancer

### NodePort
- Manual port management
- Direct node access required
- Lower cost
- Better for development/testing
- Works in any environment

### External Load Balancer
- Automatic port management
- Cloud provider integration
- Higher availability
- Better for production
- Cloud provider dependent

## Best Practices

1. Security Considerations
   - Limit exposed ports
   - Use firewall rules
   - Consider IP whitelisting

2. Port Management
   - Document used ports
   - Avoid port conflicts
   - Use consistent port numbering

3. High Availability
   - Deploy across multiple nodes
   - Consider using readiness probes
   - Implement proper health checks

4. Production Usage
   - Prefer LoadBalancer for production
   - Use NodePort for development/testing
   - Implement proper monitoring

5. Performance
   - Monitor node capacity
   - Balance traffic across nodes
   - Consider node affinity rules

## Common Issues and Solutions

1. Port Conflicts
   ```bash
   # Check used NodePorts
   kubectl get svc --all-namespaces | grep NodePort
   ```

2. Access Issues
   - Verify firewall rules
   - Check node network connectivity
   - Confirm service selectors

3. Load Balancing
   - Use session affinity if needed
   - Monitor pod distribution
   - Check service endpoints
