# Namespaces

In this section, we will take a look at **`Namespaces`**

We have created **`Objects`** such as **`PODs`**, **`Deployments`** and **`Services`** in our cluster. Whatever we have been doing we have been doing in a **`NAMESPACE`**.

- There is a **`default`** namespace in kubernetes where all resouces are deployed unless specified otherwise. It is automatically created when kubernetes is setup initially. It also have **`kube-system`** and **`kube-node-relase`**

- We can create our own namespace using the **`kubectl create namespace`** command.

- Namespace cannot be nested and each resource can only be in one namespace. 


- To list the pods in default namespace
  ```
  $ kubectl get pods
  ```

- To list the pods in another namespace. Use **`kubectl get pods`** command along with the **`--namespace`** or **`-n`** flag argument.

  ```
  $ kubectl get pods --namespace=kube-system
  ```
- Here we have a pod definition file, when we create a pod with pod-definition file, the pod is created in the default namespace.

```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: front-end
spec:
  containers:
  - name: nginx-container
    image: nginx
```

## Kubernetes Resouces scope

- In a namespace
` kubectl api-resources --namespaced=true`

- Not in a namespace
` kubectl api-resources --namespaced=false`