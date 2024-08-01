# Pod 

- Pods are the smallest deployable units in Kubernetes.
- A Pod is a group of one or more containers with shared storage and network resources. It includes a specification for running the containers.
- Pod contents are co-located, co-scheduled, and run in a shared context.
- A Pod models an application-specific "logical host" with tightly coupled application containers. This is similar to applications on the same physical or virtual machine in non-cloud contexts.
- A Pod can also contain init containers that run during startup.
- Ephemeral containers can be injected for debugging a running Pod.

## Sample Pod Definition File

- The following is a sample pod definition file named pod.yaml.

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
  ```