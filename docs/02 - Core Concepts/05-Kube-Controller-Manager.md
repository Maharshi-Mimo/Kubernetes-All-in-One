# Kube-Controller Manager

- The Kubernetes controller manager is a daemon that embeds core control loops.
- In robotics and automation, a control loop is a non-terminating loop that regulates the system's state.
- In Kubernetes, a controller is a control loop that:
  - Watches the shared state of the cluster through the apiserver.
  - Makes changes to move the current state towards the desired state.
- Examples of controllers in Kubernetes include:
  - **Replication controller**: It is responsible for monitoring the status of replicasets and ensuring that the desired number of pods are available at all time within the set.
  - **Node Controller**: Responsible for monitoring the state of the Nodes and taking necessary actions to keep the application running.
  - **Endpoints controller**: 
  - **Namespace controller**: 
  - **Service account controller**: 

## Installing Kube-Controller Manager 

 - Download the kube-controller manager binary from the kubernetes release page: 

```bash
 $ wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-controller-manager
```
- By default all the controller but you can manage them individually from ```kube-controller-manager.service``` 

```bash 
$ cat /etc/systemd/system/kube-controller-manager.service
``` 

- See the running process and effective options by listing the process on master ndoe and searching for kube-controller-manager

```bash 
$ ps -aux | grep kube-controller-manager
```



