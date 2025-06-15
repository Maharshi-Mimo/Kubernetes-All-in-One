# PersistentVolumeClaim

A PersistentVolumeClaim (PVC) is a request for storage by a user in Kubernetes. It is similar to a Pod in that Pods consume node resources and PVCs consume PersistentVolumes (PVs). PVCs are used to request specific storage resources and can be dynamically provisioned or bound to existing PVs.

## Features

- **Storage Requests**: PVCs specify the amount of storage required and the access mode (ReadWriteOnce, ReadOnlyMany, or ReadWriteMany).
- **Dynamic Provisioning**: PVCs can trigger the creation of a new PV if none exist that match the requested storage.
- **Binding**: PVCs are bound to PVs that satisfy the requested storage and access modes.
- **Storage Classes**: PVCs can specify a storage class to use for dynamic provisioning.
- **Resource Management**: PVCs allow for resource quotas and limits to be applied to storage.

## PVC Lifecycle

![Persistent Volume](/images/architecture-volumemount.webp)

The lifecycle of a PersistentVolumeClaim goes through several stages:

1. **Provisioning**
   - **Static**: Administrator creates PVs that are available for use
   - **Dynamic**: PVs are automatically created when PVC is created based on StorageClass

2. **Binding**
   - PVC is matched and bound to a specific PV
   - The binding is exclusive and one-to-one
   - PVC remains unbound until a suitable PV is available

3. **Using**
   - Pods use the PVC as a volume
   - As long as a Pod is using the PVC, the PV cannot be deleted
   - This protection is called "Storage Object in Use Protection"

4. **Releasing**
   - When a PVC is deleted, the PV enters the "Released" phase
   - The volume is no longer bound to the PVC but is not yet available for another PVC

5. **Reclaiming**
   - The PV can be reclaimed based on its reclaim policy:
     - Retain: Manual reclamation required
     - Delete: PV and underlying storage are automatically deleted
     - Recycle: Basic scrub (rm -rf) is performed (deprecated)

Here is an example of a PVC manifest:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: my-pvc
spec:
    accessModes:
        - ReadWriteOnce
    resources:
        requests:
            storage: 1Gi
    storageClassName: standard
```

In this example:
- `accessModes` specifies that the volume can be mounted as read-write by a single node.
- `resources.requests.storage` specifies the amount of storage requested.
- `storageClassName` specifies the storage class to use for dynamic provisioning.

## Warnings

- **Storage Limits**: Ensure that the requested storage does not exceed the available storage in the cluster.
- **Access Modes**: Be aware of the access modes supported by your storage provider.
- **Binding**: PVCs may remain unbound if no suitable PVs are available or if dynamic provisioning fails.
- **Storage Class**: If no storage class is specified, the default storage class will be used, which may not meet the requirements.

By understanding and properly configuring PVCs, you can effectively manage storage resources in your Kubernetes cluster.
