# Service: ClusterIP

- In Kubernetes, Services are an abstract way to expose an application running on a set of Pods. Services can have a cluster-scoped virtual IP address (using a Service of type: ClusterIP). Clients can connect using that virtual IP address.

- ClusterIP is the default Service type. It is used to expose a service to the cluster only 

> [!NOTE]
> 1. **Internal-Only Access**: ClusterIP services are only accessible within the cluster. They cannot be accessed from outside the cluster.
> 2. **Automatic Load Balancing**: ClusterIP services automatically distribute traffic across the Pods that are part of the service
> 3. **DNS Resolution**: Kubernetes automatically assigns a DNS name to the ClusterIP service, which can be used by other services and Pods within the cluster to access it.
> 4. **Service Discovery** : ClusterIP services are discoverable within the cluster using the DNS name or the ClusterIP address.
> 5. **Immutable ClusterIP**: Once a ClusterIP is assigned to a service, it cannot be changed. If you need to change it, you must delete and recreate the service.
> 6. **No External Access**: ClusterIP services are not accessible from outside the cluster. If you
need to expose a service to the outside world, you need to use a different Service type, such
as `NodePort` or `LoadBalancer`.


```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    k8s-app: kube-dns
    kubernetes.io/cluster-service: "true"
    kubernetes.io/name: CoreDNS
  name: kube-dns
  namespace: kube-system
spec:
  clusterIP: 10.96.0.10
  ports:
  - name: dns
    port: 53
    protocol: UDP
    targetPort: 53
  - name: dns-tcp
    port: 53
    protocol: TCP
    targetPort: 53
  selector:
    k8s-app: kube-dns
  type: ClusterIP
  ```
  - The above Service definition is for the CoreDNS service, which is a cluster-scoped service that 
  exposes the DNS service running on a set of Pods. The Service has a cluster-scoped virtual 
  IP address of 10.96.0.10, and it listens on port 53