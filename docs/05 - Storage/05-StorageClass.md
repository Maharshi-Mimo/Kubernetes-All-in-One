# Kubernetes StorageClass Guide

## What is StorageClass?
StorageClass is a Kubernetes resource that enables dynamic provisioning of persistent volumes. It serves as a blueprint that defines how storage will be provisioned for applications requesting persistent storage.

## How StorageClass Works
1. When a PVC (PersistentVolumeClaim) is created requesting a specific StorageClass
2. The storage provisioner associated with that StorageClass creates a PV (PersistentVolume)
3. The PV is then bound to the PVC
4. The pod can now use this storage through the PVC

## Cloud Provider Integration

### AWS EBS StorageClass
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: aws-ebs-gp2
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  fsType: ext4
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

### Azure Disk StorageClass
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: azure-disk
provisioner: kubernetes.io/azure-disk
parameters:
  storageaccounttype: Standard_LRS
  kind: Managed
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

### GCP Persistent Disk StorageClass
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gcp-standard
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
  fstype: ext4
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

## Common Parameters Explained
- `provisioner`: Determines which volume plugin is used
- `reclaimPolicy`: What happens to PVs when PVC is deleted (Delete/Retain)
- `volumeBindingMode`: When volume binding and dynamic provisioning occurs
- `allowVolumeExpansion`: Whether volumes can be expanded after creation
- `parameters`: Provider-specific configuration options

## Sample PVC Using StorageClass

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-app-storage
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: aws-ebs-gp2  # Reference to StorageClass
  resources:
    requests:
      storage: 10Gi
```

## Best Practices

1. **Default StorageClass**: Mark one StorageClass as default using annotation:
   ```yaml
   annotations:
     storageclass.kubernetes.io/is-default-class: "true"
   ```

2. **Volume Binding Mode**: Use `WaitForFirstConsumer` for better pod scheduling

3. **Reclaim Policy**: Use `Retain` for production data to prevent accidental deletion

4. **Volume Expansion**: Enable `allowVolumeExpansion` for flexibility

## Troubleshooting

Common issues and solutions:

1. **PVC Stuck in Pending**
   - Check StorageClass exists
   - Verify cloud provider credentials
   - Check cloud quotas and limits

2. **Volume Not Mounting**
   - Verify PVC is bound
   - Check pod events
   - Validate node has access to storage

3. **Storage Class Not Working**
   - Verify provisioner is running
   - Check cloud provider configuration
   - Validate API versions

## Reference Commands

```bash
# List StorageClasses
kubectl get storageclass

# Describe StorageClass
kubectl describe storageclass <name>

# Check PVC Status
kubectl get pvc

# Check PV Status
kubectl get pv
```
</markdown>
