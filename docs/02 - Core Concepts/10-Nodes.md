# Node

**Kubernetes runs your workload by placing containers into Pods to run on Nodes. A node may be a virtual or physical machine, depending on the cluster. Each node is managed by the control plane and contains the services necessary to run Pods.**

## What is a Node?

A Node is a worker machine in Kubernetes. It can be a virtual machine (VM) or a physical machine, depending on the cluster setup. Each node contains the necessary services to run containers and is managed by the Kubernetes Control Plane.

### Key Characteristics of Nodes:

- **Compute Resource Provider**: Provides CPU, memory, and storage for running containers
- **Pod Host**: Runs one or more Pods and maintains running containers
- **Service Integration**: Includes a container runtime, kubelet, and kube-proxy
- **Status Management**: Reports its status back to the control plane
- **Self-Healing**: Automatically restarts failed containers (through kubelet)

## Node Components

Each node contains the following key components:

### 1. **Kubelet**
- An agent running on every node
- Ensures containers are running in a Pod
- Communicates with the API server
- Manages container lifecycle (create, start, stop, delete)
- Performs health checks and reports node status

### 2. **Container Runtime**
- Software responsible for running containers (e.g., containerd, Docker, CRI-O)
- Implements the Container Runtime Interface (CRI)
- Pulls images and manages container lifecycle

### 3. **kube-proxy**
- Network proxy running on each node
- Maintains network rules for Pods
- Enables communication within and outside the cluster
- Handles service discovery and load balancing

## Node Management

There are two main ways to add Nodes to the Kubernetes cluster:

### 1. **Self-Registration (Automatic)**
- The kubelet on a node automatically registers itself with the control plane
- Most common approach in cloud environments and automated deployments
- Kubelet reports node status and resource availability

### 2. **Manual Registration**
- An administrator manually creates a Node object in the Kubernetes API server
- Used when kubelet cannot self-register
- Useful in on-premises environments

### Example Node Manifest

```yaml
apiVersion: v1
kind: Node
metadata:
  name: node-1
  labels:
    kubernetes.io/hostname: node-1
    node-role.kubernetes.io/worker: ""
    environment: production
  annotations:
    node.alpha.kubernetes.io/description: "Production worker node"
spec:
  taints:
    - key: dedicated
      value: database
      effect: NoSchedule
status:
  capacity:
    cpu: "4"
    memory: "8Gi"
    pods: "110"
  allocatable:
    cpu: "3.5"
    memory: "7Gi"
    pods: "110"
```

## Node Status

### Node Conditions

Nodes report several important conditions:

| Condition | Description |
|-----------|-------------|
| **Ready** | True if the node is healthy and ready to accept pods; False if unhealthy |
| **DiskPressure** | True if there is insufficient disk space |
| **MemoryPressure** | True if there is insufficient memory |
| **PIDPressure** | True if there are too many processes |
| **NetworkUnavailable** | True if network is not properly configured |

### Node Phase (Deprecated)

Older Kubernetes versions used node phases, but conditions are now the preferred method:
- `Pending`: Node created but not initialized
- `Running`: Node is ready and operational
- `Terminated`: Node is no longer available

### Check Node Status

```bash
# View all nodes
kubectl get nodes

# View detailed node information
kubectl describe node <node-name>

# Check specific node conditions
kubectl get nodes -o wide
```

## Node Capacity and Allocation

### Capacity
The maximum resources available on a node:
- CPU cores
- Memory
- Number of Pods

### Allocatable
Resources reserved for system processes and not available for Pods:
- Kubelet reserves CPU and memory for node operation
- Container runtime reserves resources
- OS reserves resources

```bash
# View node capacity and allocatable resources
kubectl describe node <node-name> | grep -A 2 "Capacity\|Allocatable"
```

## Node Labels and Selectors

Labels are key-value pairs used to organize and manage nodes:

### Common Node Labels
```yaml
kubernetes.io/hostname: node-1
kubernetes.io/os: linux
kubernetes.io/arch: amd64
node-role.kubernetes.io/master: ""
node-role.kubernetes.io/worker: ""
environment: production
disk: ssd
```

### Label a Node
```bash
kubectl label nodes <node-name> <key>=<value>
```

### Use Node Selectors in Pods
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  nodeSelector:
    environment: production
  containers:
  - name: nginx
    image: nginx:latest
```

## Node Taints and Tolerations

### Taints
Taints repel Pods from scheduling on specific nodes. They prevent pods from being scheduled unless they tolerate the taint.

```bash
# Add a taint to a node
kubectl taint nodes <node-name> <key>=<value>:<effect>

# Examples of effects:
# - NoSchedule: Pod will not be scheduled
# - NoExecute: Pod will be evicted if already running
# - PreferNoSchedule: Scheduler will try to avoid, but not guaranteed
```

### Tolerations
Tolerations allow Pods to be scheduled on nodes with taints.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "database"
    effect: "NoSchedule"
  containers:
  - name: nginx
    image: nginx:latest
```

## Node Lifecycle

1. **Node Creation**: Node is registered with the API server
2. **Node Ready**: kubelet communicates with API server and passes health checks
3. **Pod Scheduling**: Scheduler assigns pods to the node based on resources and constraints
4. **Pod Execution**: Pods run on the node; kubelet monitors and maintains them
5. **Node Termination**: Node is cordoned (stops receiving new pods), existing pods are evicted

### Cordoning and Draining Nodes

```bash
# Prevent new pods from being scheduled (cordon)
kubectl cordon <node-name>

# Drain node gracefully (remove running pods)
kubectl drain <node-name> --ignore-daemonsets

# Allow pods to be scheduled again (uncordon)
kubectl uncordon <node-name>
```

## Node Resource Requests and Limits

Pods must specify resource requirements so the scheduler can place them correctly:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    resources:
      requests:
        cpu: "100m"        # minimum CPU required
        memory: "128Mi"    # minimum memory required
      limits:
        cpu: "500m"        # maximum CPU allowed
        memory: "512Mi"    # maximum memory allowed
```

## Best Practices for Nodes

1. **Proper Labeling**: Use meaningful labels for node organization and scheduling
2. **Resource Allocation**: Ensure nodes have sufficient resources for workloads
3. **Regular Maintenance**: Update kernel, runtime, and other software regularly
4. **Health Monitoring**: Monitor CPU, memory, disk, and network usage
5. **Pod Density**: Avoid overloading nodes with too many pods
6. **Graceful Shutdown**: Use `drain` before removing nodes
7. **DaemonSets**: Use for system services that need to run on all nodes
8. **Affinity Rules**: Use pod and node affinity for advanced scheduling
9. **Resource Quotas**: Set limits on resource consumption per namespace
10. **Backup Strategy**: Plan for node failures and have recovery procedures

## Important Notes

>[!Note]
> - Kubernetes keeps the object for an invalid Node and continues checking to see whether it becomes healthy
> - The Node object name (`metadata.name`) must be a valid DNS subdomain name
> - Node names are case-insensitive but are typically lowercase
> - kubelet runs with elevated privileges to manage container lifecycle
> - Nodes automatically report metrics like CPU, memory, and storage usage
> - Remove nodes gracefully using `kubectl drain` before shutting down to migrate workloads 