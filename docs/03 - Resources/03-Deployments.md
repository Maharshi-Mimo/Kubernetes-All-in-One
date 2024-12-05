# Deployments

A Deployment provides declarative updates for Pods and ReplicaSets.

## How to create a deployment 

- Using Create Command: 

`kubectl create deployment NAME --image=image -- [COMMAND] [args] [options]` 

ex `kubectl create deployment mydep --image=busybox:latest --image=nginx --port=80 --replicas=2 -- date` 

- Declarative Format: 

``` 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
``` 
In this example 
1. `.metadata.name` (nginx-deployment) is taken up by the replicaSet and pod. 
2. `.spec.replicaSet` shows it will create 2 replica of the pod. 
3. The `.spec.selector` field defines how the created ReplicaSet finds which Pods to manage. In this case, you select a label that is defined in the Pod template (`app: nginx`).
4. The Pod template's specification, or `.template.spec` field, indicates that the Pods run one container, nginx. 

Now save this as a file and then deploy it using command `kubectl apply -f /path/to/deploy.yaml -n <namespace>` | (without `-n <namespace>` it will be deployed in `default` namespace)  

The you can observe the deployment rollout with the command `kubectl get dpeloyment|replicaset|pod -n <namespace>` 