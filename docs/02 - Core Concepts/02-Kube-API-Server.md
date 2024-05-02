# kube API Server

The Kube-apiserver the primary component of the control plane that is responsible for providing the kubernetes control and communication between master, worker & Cloud API server.

**Kube-apiserver is responsible for:**

 - Athentication
 - Authorization
 - Admission Control
 - Endpoint Management
 - Validation
 - Updating the Kube-scheduler
 - Updating the Kube-controller-manager

All the activity of the Kube-apiserver is stored ETCD in the form of key-value pairs. Also, kube-apiserver is only component that communicate with ETCD server  

## Kube-apiserver Installation. 

Don't need to worry about this if you are using `kubeadm` or similar culster management tool. 

Building it the hardway! (Good luck)

You can download the kube-apiserver binary from the [release page](https://github.com/kubernetes/kubernetes/releases)

You need to choose the:
 - version 
 - OS
 - Architecture

Or just 

```bash 
$ wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-apiserver
```
## Kube-apiserver Configuration 

You can view the kube-apiserver configuration by running the following command: 

```bash 
$ kubectl get pods -n kube-system
```
or 

```bash 
$ cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

