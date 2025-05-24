# Kubernetes Volume Expansion Guide

## Overview
Volume expansion in Kubernetes allows you to increase the size of Persistent Volumes (PVs) and Persistent Volume Claims (PVCs) dynamically, without having to recreate the resources.

## Online vs Offline Expansion

### Online Expansion
- Volumes can be expanded while they are in use by a Pod
- Supported by many storage providers (e.g., AWS EBS, GCE PD, Azure Disk)
- No Pod restart required
- Real-time expansion without service interruption

### Offline Expansion
- Requires Pod to be deleted and recreated
- Used when storage provider doesn't support online expansion
- Involves service interruption
- Typically needed for older storage systems

## Storage Class Configuration

To enable volume expansion, you need to:

1. Set `allowVolumeExpansion: true` in StorageClass definition:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard-expandable
provisioner: kubernetes.io/aws-ebs
allowVolumeExpansion: true
parameters:
  type: gp2
```

## Expansion Workflows

### 1. Online Expansion Process
1. Edit PVC to increase storage request
2. Wait for expansion to complete
3. Verify new size is available to Pod

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
spec:
  resources:
    requests:
      storage: 10Gi  # Increase this value
```

### 2. Offline Expansion Process
1. Delete Pod using PVC
2. Edit PVC to increase storage
3. Wait for expansion
4. Recreate Pod

## Troubleshooting Guide

### Common Issues

1. Expansion Stuck
- Check PVC events: `kubectl describe pvc <pvc-name>`
- Verify StorageClass allows expansion
- Check CSI driver logs

2. Filesystem Expansion Failed
```bash
# Check filesystem size
kubectl exec <pod-name> -- df -h

# Check volume status
kubectl describe pvc <pvc-name>
```

3. CSI Driver Issues
- Check CSI controller logs
- Verify CSI driver supports expansion
- Check storage provider quotas

### Best Practices

1. Always monitor expansion progress
2. Have backup before expansion
3. Test expansion in non-production first
4. Document minimum/maximum size limits
5. Consider setting up alerts for expansion events

## Limitations

- Volume size can only be increased, not decreased
- Some storage providers have minimum expansion steps
- Not all StorageClasses support expansion
- May require specific Kubernetes version

## Monitoring

Monitor volume expansion using:
- Kubernetes events
- Prometheus metrics
- CSI driver logs
- Storage provider metrics

## Example Commands

```bash
# Edit PVC size
kubectl patch pvc data-pvc -p '{"spec":{"resources":{"requests":{"storage":"20Gi"}}}}'

# Monitor expansion status
kubectl get pvc data-pvc -w

# Check filesystem size in Pod
kubectl exec -it app-pod -- df -h /data
```

## References

- [Kubernetes Official Documentation](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#expanding-persistent-volumes-claims)
- [CSI Volume Expansion](https://kubernetes-csi.github.io/docs/volume-expansion.html)
- [Storage Provider-specific Documentation]
