# kubernets kubeadm cluster upgrade 

This is the document to upgrade the k8s version 

NOTE: The upgrade activity must be one by one version not to be like direct jump. 

Let we assume your current kubernets version is 1.18.0 and the current stable version is 1.20.0 so here we cant update the clsuter directly to the 1.20.0 the flow of upgrade is like

  1.18.0 --> 1.19.0 --> 1.20.0

This is the exactly the way of cluster updaiting.

Before going to do this activity you have root privilization.

## Upgrade the cluster
Note: alias k='kubectl' which means I am 'using' k instead of kubectl

The current verion of my cluster is 1.19.0
```
root@controlplane:~# k get nodes
NAME           STATUS   ROLES    AGE    VERSION
controlplane   Ready    master   110m   v1.19.0
node01         Ready    worker   109m   v1.19.0
```
Now we need to check the kubeadm laterst version

```
root@controlplane:~# kubeadm upgrade plan

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT       AVAILABLE
kubelet     2 x v1.19.0   v1.20.0

Upgrade to the latest version in the v1.19 series:

COMPONENT                 CURRENT   AVAILABLE
kube-apiserver            v1.19.0   v1.20.0
kube-controller-manager   v1.19.0   v1.20.0
kube-scheduler            v1.19.0   v1.20.0
kube-proxy                v1.19.0   v1.20.0
CoreDNS                   1.7.0     1.7.0
etcd                      3.4.9-1   3.4.9-1
```

The updated version of kubeadm is 1.20.0 

so we wil start the upgrade activity

### Master Upgrade

First we need to drain the master 

```
root@controlplane:~# k drain controlplane --ignore-daemonsets
node/controlplane cordoned
node/controlplane evicted
```

Now update the packages then install kubeadm 1.20.0 and finnaly upgrade the node

```
root@controlplane:~# apt update
root@controlplane:~# apt install kubeadm=1.20.0-00
```
upgrade kubernetes controlplane

```
root@controlplane:~# kubeadm upgrade apply v1.20.0
```
even though we have upgrade our kubeadm but the version still appear on old one for that we need to upgrade the kubelet also

```
root@node01:~# apt install kubelet=1.20.0-00
root@node01:~# systemctl restart kubelet
```
Now list the nodes

```
root@controlplane:~# k get nodes
NAME           STATUS                        ROLES                  AGE   VERSION
controlplane   NotReady,SchedulingDisabled   control-plane,master   42m   v1.20.0
node01         Ready                         <none>                 41m   v1.19.0
```
The master node is in SchedulingDisabled state uncoron that node to make it as active
```
root@controlplane:~# kubectl uncordon controlplane
node/controlplane uncordoned

root@controlplane:~# k get nodes
NAME           STATUS   ROLES                  AGE    VERSION
controlplane   Ready    control-plane,master   149m   v1.20.0
node01         Ready    worker                 148m   v1.19.0
```


### Worker Upgrade

First drain the worker node 

```
root@controlplane:~# k drain node01 --ignore-daemonsets
node/node01 cordoned
node/node01 evicted
```

Now ssh into the worker node and update the packages then install kubeadm 1.20.0 and finnaly upgrade the node 
```
ssh node01
apt update
apt install kubeadm=1.20.0-00
kubeadm upgrade node
```

even though we have upgrade our kubeadm but the version still appear on old one for that we need to upgrade the kubelet also

```
root@node01:~# apt install kubelet=1.20.0-00
root@node01:~# systemctl restart kubelet
```
Now exist from worker node again log in to the master then list the nodes

```
root@controlplane:~# k get nodes
NAME           STATUS                     ROLES                  AGE    VERSION
controlplane   Ready                      control-plane,master   147m   v1.20.0
node01         Ready,SchedulingDisabled   <none>                 146m   v1.20.0
```
The worker node is in SchedulingDisabled state uncoron that node to make it as active 
```
root@controlplane:~# kubectl uncordon node01
node/node01 uncordoned

root@controlplane:~# k get nodes
NAME           STATUS   ROLES                  AGE    VERSION
controlplane   Ready    control-plane,master   149m   v1.20.0
node01         Ready    worker                 148m   v1.20.0
```

Now we have successfully upgrade the kubernets clsuter from 1.19.0 --> 1.20.0
