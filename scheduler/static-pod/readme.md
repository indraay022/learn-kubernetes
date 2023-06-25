First idenity the kubelet config file:
```
root@controlplane:~# ps -aux | grep /usr/bin/kubelet
root      3668  0.0  1.5 1933476 63076 ?       Ssl  Mar13  16:18 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.2
root      4879  0.0  0.0  11468  1040 pts/0    S+   00:06   0:00 grep --color=auto /usr/bin/kubelet
root@controlplane:~# 
```
From the output we can see that the kubelet config file used is `/var/lib/kubelet/config.yaml`


Next, lookup the value assigned for `staticPodPath:`
```
root@controlplane:~# grep -i staticpod /var/lib/kubelet/config.yaml
staticPodPath: /etc/kubernetes/manifests
root@controlplane:~# 
```
As you can see, the path configured is the `/etc/kubernetes/manifests` directory.



---
## Deleting Static Pod
First, let's identify the node in which the pod called static-greenbox is created. To do this, run:
```
root@controlplane:~# kubectl get pods --all-namespaces -o wide  | grep static-greenbox
default       static-greenbox-node01                 1/1     Running   0          19s     10.244.1.2   node01       <none>           <none>
root@controlplane:~#
```
From the result of this command, we can see that the pod is running on node01.


Next, SSH to node01 and identify the path configured for static pods in this node.

Important: The path need not be /etc/kubernetes/manifests. Make sure to check the path configured in the kubelet configuration file.
```root@controlplane:~# ssh node01 
root@node01:~# ps -ef |  grep /usr/bin/kubelet 
root        4147       1  0 14:05 ?        00:00:00 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock --pod-infra-container-image=registry.k8s.io/pause:3.9
root        4773    4733  0 14:05 pts/0    00:00:00 grep /usr/bin/kubelet

root@node01:~# grep -i staticpod /var/lib/kubelet/config.yaml
staticPodPath: /etc/just-to-mess-with-you

root@node01:~# 
```
Here the staticPodPath is /etc/just-to-mess-with-you


Navigate to this directory and delete the YAML file:
```
root@node01:/etc/just-to-mess-with-you# ls
greenbox.yaml
root@node01:/etc/just-to-mess-with-you# rm -rf greenbox.yaml 
root@node01:/etc/just-to-mess-with-you#
```
Exit out of node01 using CTRL + D or type exit. You should return to the controlplane node. Check if the static-greenbox pod has been deleted:
```
root@controlplane:~# kubectl get pods --all-namespaces -o wide  | grep static-greenbox
root@controlplane:~# 
``` 