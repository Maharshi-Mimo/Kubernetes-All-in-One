
# Kube Scheduler

In this section, we will take a look at kube-scheduler.

#### kube-scheduler is responsible for scheduling pods on nodes.  
- The kube-scheduler is only responsible for deciding which pod goes on which node. It doesn't actually place the pod on the nodes, that's the job of the **`kubelet`**.

kube-scheduler selects a node for the pod in a 2-step operation:

1. Filtering
2. Scoring

  ![kube-scheduler1](/images/scheduling-framework-extensions.png)


## Install kube-scheduler - Manual
- Download the kubescheduler binary from the kubernetes release pages 

[kube-scheduler](https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-scheduler). 
For example: To download kube-scheduler v1.13.0, Run the below command.

```bash 
  $ wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-scheduler
```
- Extract it
- Run it as a service

## View kube-scheduler options - kubeadm
- If you set it up with kubeadm tool, kubeadm tool will deploy the kube-scheduler as pod in kube-system namespace on master node.

```bash 
  $ kubectl get pods -n kube-system
```
- You can see the options for kube-scheduler in pod definition file that is located at **`/etc/kubernetes/manifests/kube-scheduler.yaml`**

```bash 
  $ cat /etc/kubernetes/manifests/kube-scheduler.yaml
```


- You can also see the running process and effective options by listing the process on master node and searching for kube-apiserver.
```bash 
  $ ps -aux | grep kube-scheduler
```


