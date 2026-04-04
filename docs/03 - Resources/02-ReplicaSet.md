# ReplicaSet

## Difference between ReplicaSet and Replication Controller
- **`Replication Controller`** is the older technology that is being replaced by a **`ReplicaSet`**.
- **`ReplicaSet`** is the new way to setup replication, and it requires a `selector` definition when compare to Replication Controller

### Replication Controller Definition File
  
```
    apiVersion: v1
    kind: ReplicationController
    metadata:
      name: myapp-rc
      labels:
        app: myapp
        type: front-end
    spec:
     template:
        metadata:
          name: myapp-pod
          labels:
            app: myapp
            type: front-end
        spec:
         containers:
         - name: nginx-container
           image: nginx
     replicas: 3
```

### ReplicaSet Definition File

```
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: myapp-replicaset
      labels:
        app: myapp
        type: front-end
    spec:
     template:
        metadata:
          name: myapp-pod
          labels:
            app: myapp
            type: front-end
        spec:
         containers:
         - name: nginx-container
           image: nginx
     replicas: 3
     selector:
       matchLabels:
        type: front-end
 ```

  - To Create the replicaset
    ```
    $ kubectl create -f replicaset-definition.yaml
    ```
  - To list all the replicaset
    ```
    $ kubectl get replicaset
    ```
  - To list pods that are launch by the replicaset
    ```
    $ kubectl get pods
    ```

### Scaling of the Replicaset

There are two ways to achieve this: 

- scaling command

```
kubectl scale --replicas=6 replicaset <replicaset-resource-name>
```

- editing the `spec.replicas` field in the definition file or the replicaset resource. 

## Selectors and Labels 

- Labels are `key/value` pairs that are attached to objects such as Pods. Labels are intended to be used to specify identifying attributes of objects that are meaningful and relevant to users, but do not directly imply semantics to the core system. Labels can be used to organize and to select subsets of objects. 

- Via a label selector, the client/user can identify a set of objects. The label selector is the core grouping primitive in Kubernetes. 

- The API currently supports two types of selectors: equality-based and set-based. 
  - Equality-based requirement : Equality- or inequality-based requirements allow filtering by label keys and values. For example
    ``` 
    environment = production
    tier != frontend
    ```
  - Set-based selectors: Set-based label requirements allow filtering keys according to a set of values. Three kinds of operators are     supported: in,notin and exists (only the key identifier). For example:
    ``` 
    environment in (production, qa)
    tier notin (frontend, backend)
    partition
    !partition
    ``` 