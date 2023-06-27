# Scenario
We have a production cluster with applications running on it. Let us explore the setup first.

What is the current version of the cluster?  
How many nodes are part of this cluster?
```
controlplane ~ ➜  kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   76m   v1.26.0
node01         Ready    <none>          75m   v1.26.0

```
How many nodes can host workloads in this cluster?
```
kubectl get nodes
```

How many nodes can host workloads in this cluster?
```
controlplane ~ ➜  kubectl describe nodes  controlplane | grep -i taint
Taints:             <none>

controlplane ~ ➜  kubectl describe nodes  node01 | grep -i taint
Taints:             <none>
```

How many applications are hosted on the cluster?
```
controlplane ~ ➜  kubectl get deployments.apps
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
blue   5/5     5            5           19m
```
What nodes are the pods hosted on?

```
controlplane ~ ➜  kubectl get pods -o wide
NAME                   READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
blue-987f68cb5-5t6cw   1/1     Running   0          20m   10.244.0.5   controlplane   <none>           <none>
blue-987f68cb5-7tbn9   1/1     Running   0          20m   10.244.1.2   node01         <none>           <none>
blue-987f68cb5-krbsw   1/1     Running   0          20m   10.244.0.4   controlplane   <none>           <none>
blue-987f68cb5-v6ksb   1/1     Running   0          20m   10.244.1.4   node01         <none>           <none>
blue-987f68cb5-x6pb4   1/1     Running   0          20m   10.244.1.3   node01         <none>           <none>

```
In order to ensure minimum downtime, upgrade the cluster one node at a time, while moving the workloads to another node.

What is the latest stable version of Kubernetes as of today?
What is the latest version available for an upgrade with the current version of the kubeadm tool installed?
Look at the remote version
```
controlplane ~ ➜ kubeadm upgrade plan
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks.
[upgrade] Running cluster health checks
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: v1.26.0
[upgrade/versions] kubeadm version: v1.26.0
I0627 05:35:10.512126   19579 version.go:256] remote version is much newer: v1.27.3; falling back to: stable-1.26
[upgrade/versions] Target version: v1.26.6
[upgrade/versions] Latest version in the v1.26 series: v1.26.6

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT       TARGET
kubelet     2 x v1.26.0   v1.26.6

Upgrade to the latest version in the v1.26 series:

COMPONENT                 CURRENT   TARGET
kube-apiserver            v1.26.0   v1.26.6
kube-controller-manager   v1.26.0   v1.26.6
kube-scheduler            v1.26.0   v1.26.6
kube-proxy                v1.26.0   v1.26.6
CoreDNS                   v1.9.3    v1.9.3
etcd                      3.5.6-0   3.5.6-0

You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.26.6

Note: Before you can perform this upgrade, you have to update kubeadm to v1.26.6.

_____________________________________________________________________


The table below shows the current state of component configs as understood by this version of kubeadm.
Configs that have a "yes" mark in the "MANUAL UPGRADE REQUIRED" column require manual config upgrade or
resetting to kubeadm defaults before a successful upgrade can be performed. The version to manually
upgrade to is denoted in the "PREFERRED VERSION" column.

API GROUP                 CURRENT VERSION   PREFERRED VERSION   MANUAL UPGRADE REQUIRED
kubeproxy.config.k8s.io   v1alpha1          v1alpha1            no
kubelet.config.k8s.io     v1beta1           v1beta1             no
_____________________________________________________________________

```

We will be upgrading the controlplane node first. Drain the controlplane node of workloads and mark it UnSchedulable

```
controlplane ~ ➜  kubectl drain controlplane --ignore-daemonsets
node/controlplane cordoned
Warning: ignoring DaemonSet-managed Pods: kube-flannel/kube-flannel-ds-fdcgw, kube-system/kube-proxy-m25hs
evicting pod kube-system/coredns-787d4945fb-vcfw4
evicting pod default/blue-987f68cb5-xcw7s
evicting pod default/blue-987f68cb5-9jfpp
evicting pod kube-system/coredns-787d4945fb-fvhjb
pod/blue-987f68cb5-9jfpp evicted
pod/blue-987f68cb5-xcw7s evicted
pod/coredns-787d4945fb-fvhjb evicted
pod/coredns-787d4945fb-vcfw4 evicted
node/controlplane drained
```

Upgrade the controlplane components to exact version v1.27.0
```
controlplane ~ ➜ apt-get install kubeadm=1.27.0-00
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages will be upgraded:
  kubeadm
1 upgraded, 0 newly installed, 0 to remove and 83 not upgraded.
Need to get 9,931 kB of archives.
After this operation, 1,393 kB of additional disk space will be used.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubeadm amd64 1.27.0-00 [9,931 kB]
Fetched 9,931 kB in 0s (33.5 MB/s)
debconf: delaying package configuration, since apt-utils is not installed
(Reading database ... 20428 files and directories currently installed.)
Preparing to unpack .../kubeadm_1.27.0-00_amd64.deb ...
Unpacking kubeadm (1.27.0-00) over (1.26.0-00) ...
Setting up kubeadm (1.27.0-00) ...

controlplane ~ ➜  kubeadm upgrade apply v1.27.0
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks.
[upgrade] Running cluster health checks
[upgrade/version] You have chosen to change the cluster version to "v1.27.0"
[upgrade/versions] Cluster version: v1.26.0
[upgrade/versions] kubeadm version: v1.27.0
[upgrade] Are you sure you want to proceed? [y/N]: y
[upgrade/prepull] Pulling images required for setting up a Kubernetes cluster
[upgrade/prepull] This might take a minute or two, depending on the speed of your internet connection
[upgrade/prepull] You can also perform this action in beforehand using 'kubeadm config images pull'
W0627 06:30:04.118174   18898 images.go:80] could not find officially supported version of etcd for Kubernetes v1.27.0, falling back to the nearest etcd version (3.5.7-0)
W0627 06:30:18.362021   18898 checks.go:835] detected that the sandbox image "k8s.gcr.io/pause:3.6" of the container runtime is inconsistent with that used by kubeadm. It is recommended that using "registry.k8s.io/pause:3.9" as the CRI sandbox image.
[upgrade/apply] Upgrading your Static Pod-hosted control plane to version "v1.27.0" (timeout: 5m0s)...
[upgrade/etcd] Upgrading to TLS for etcd
W0627 06:30:30.475893   18898 staticpods.go:305] [upgrade/etcd] could not find officially supported version of etcd for Kubernetes v1.27.0, falling back to the nearest etcd version (3.5.7-0)
W0627 06:30:30.480056   18898 images.go:80] could not find officially supported version of etcd for Kubernetes v1.27.0, falling back to the nearest etcd version (3.5.7-0)
[upgrade/staticpods] Preparing for "etcd" upgrade
[upgrade/staticpods] Renewing etcd-server certificate
[upgrade/staticpods] Renewing etcd-peer certificate
[upgrade/staticpods] Renewing etcd-healthcheck-client certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/etcd.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2023-06-27-06-30-30/etcd.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
[apiclient] Found 1 Pods for label selector component=etcd
[upgrade/staticpods] Component "etcd" upgraded successfully!
[upgrade/etcd] Waiting for etcd to become available
[upgrade/staticpods] Writing new Static Pod manifests to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests182138621"
[upgrade/staticpods] Preparing for "kube-apiserver" upgrade
[upgrade/staticpods] Renewing apiserver certificate
[upgrade/staticpods] Renewing apiserver-kubelet-client certificate
[upgrade/staticpods] Renewing front-proxy-client certificate
[upgrade/staticpods] Renewing apiserver-etcd-client certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-apiserver.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2023-06-27-06-30-30/kube-apiserver.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
[apiclient] Found 1 Pods for label selector component=kube-apiserver
[upgrade/staticpods] Component "kube-apiserver" upgraded successfully!
[upgrade/staticpods] Preparing for "kube-controller-manager" upgrade
[upgrade/staticpods] Renewing controller-manager.conf certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-controller-manager.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2023-06-27-06-30-30/kube-controller-manager.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
[apiclient] Found 1 Pods for label selector component=kube-controller-manager
[upgrade/staticpods] Component "kube-controller-manager" upgraded successfully!
[upgrade/staticpods] Preparing for "kube-scheduler" upgrade
[upgrade/staticpods] Renewing scheduler.conf certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-scheduler.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2023-06-27-06-30-30/kube-scheduler.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
[apiclient] Found 1 Pods for label selector component=kube-scheduler
[upgrade/staticpods] Component "kube-scheduler" upgraded successfully!
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upgrade] Backing up kubelet config file to /etc/kubernetes/tmp/kubeadm-kubelet-config1515392865/config.yaml
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.27.0". Enjoy!

[upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets if you haven't already done so.

controlplane ~ ➜  apt-get install kubelet=1.27.0-00 
Reading package lists... 5%
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages will be upgraded:
  kubelet
1 upgraded, 0 newly installed, 0 to remove and 83 not upgraded.
Need to get 18.8 MB of archives.
After this operation, 15.1 MB disk space will be freed.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubelet amd64 1.27.0-00 [18.8 MB]
Fetched 18.8 MB in 1s (26.6 MB/s)  
debconf: delaying package configuration, since apt-utils is not installed
(Reading database ... 20428 files and directories currently installed.)
Preparing to unpack .../kubelet_1.27.0-00_amd64.deb ...
/usr/sbin/policy-rc.d returned 101, not running 'stop kubelet.service'
Unpacking kubelet (1.27.0-00) over (1.26.0-00) ...
Setting up kubelet (1.27.0-00) ...
/usr/sbin/policy-rc.d returned 101, not running 'start kubelet.service'

controlplane ~ ➜  systemctl daemon-reload

controlplane ~ ➜  systemctl restart kubelet
```

Mark the controlplane node as "Schedulable" again

```
controlplane ~ ➜  kubectl uncordon controlplane
node/controlplane uncordoned
```

Next is the worker node. Drain the worker node of the workloads and mark it UnSchedulable

```
controlplane ~ ➜  kubectl drain node01 --ignore-daemonsets
node/node01 cordoned
Warning: ignoring DaemonSet-managed Pods: kube-flannel/kube-flannel-ds-rzpkv, kube-system/kube-proxy-trgn9
evicting pod kube-system/coredns-5d78c9869d-w9skz
evicting pod default/blue-987f68cb5-nx856
evicting pod default/blue-987f68cb5-w9qmb
evicting pod default/blue-987f68cb5-gkfxq
evicting pod default/blue-987f68cb5-n5vbr
evicting pod kube-system/coredns-5d78c9869d-f9kt4
evicting pod default/blue-987f68cb5-pkqcn
pod/blue-987f68cb5-nx856 evicted
pod/blue-987f68cb5-n5vbr evicted
I0627 06:37:10.651827   24948 request.go:682] Waited for 1.005656551s due to client-side throttling, not priority and fairness, request: GET:https://controlplane:6443/api/v1/namespaces/kube-system/pods/coredns-5d78c9869d-f9kt4
pod/blue-987f68cb5-w9qmb evicted
pod/blue-987f68cb5-pkqcn evicted
pod/blue-987f68cb5-gkfxq evicted
pod/coredns-5d78c9869d-w9skz evicted
pod/coredns-5d78c9869d-f9kt4 evicted
node/node01 drained
```

Upgrade the worker node to the exact version v1.27.0
```
controlplane ~ ➜  ssh node01

root@node01 ~ ➜  apt-get update
Get:2 https://download.docker.com/linux/ubuntu focal InRelease [57.7 kB]                                     
Get:3 http://archive.ubuntu.com/ubuntu focal InRelease [265 kB]                                                                
Get:4 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages [35.7 kB]                         
Get:5 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]                                                             
Get:6 http://archive.ubuntu.com/ubuntu focal-updates InRelease [114 kB]        
Get:7 http://archive.ubuntu.com/ubuntu focal-backports InRelease [108 kB]                 
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial InRelease [8993 B]        
Get:8 http://archive.ubuntu.com/ubuntu focal/universe amd64 Packages [11.3 MB]                       
Get:9 http://archive.ubuntu.com/ubuntu focal/main amd64 Packages [1275 kB]
Get:10 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 Packages [67.3 kB]
Get:11 http://archive.ubuntu.com/ubuntu focal/restricted amd64 Packages [33.4 kB]          
Get:12 http://archive.ubuntu.com/ubuntu focal/multiverse amd64 Packages [177 kB]
Get:13 http://archive.ubuntu.com/ubuntu focal-updates/restricted amd64 Packages [2554 kB]
Get:14 http://archive.ubuntu.com/ubuntu focal-updates/universe amd64 Packages [1360 kB]
Get:15 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 Packages [3299 kB]
Get:16 http://archive.ubuntu.com/ubuntu focal-updates/multiverse amd64 Packages [31.2 kB]
Get:17 http://archive.ubuntu.com/ubuntu focal-backports/universe amd64 Packages [28.6 kB]
Get:18 http://archive.ubuntu.com/ubuntu focal-backports/main amd64 Packages [55.2 kB]
Get:19 http://security.ubuntu.com/ubuntu focal-security/universe amd64 Packages [1064 kB]
Get:20 http://security.ubuntu.com/ubuntu focal-security/main amd64 Packages [2820 kB]
Get:21 http://security.ubuntu.com/ubuntu focal-security/multiverse amd64 Packages [28.5 kB]
Get:22 http://security.ubuntu.com/ubuntu focal-security/restricted amd64 Packages [2416 kB]
Fetched 27.3 MB in 3s (7831 kB/s)                           
Reading package lists... Done

root@node01 ~ ➜  apt-get install kubeadm=1.27.0-00
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following held packages will be changed:
  kubeadm
The following packages will be upgraded:
  kubeadm
1 upgraded, 0 newly installed, 0 to remove and 91 not upgraded.
Need to get 9931 kB of archives.
After this operation, 1393 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubeadm amd64 1.27.0-00 [9931 kB]
Fetched 9931 kB in 0s (36.4 MB/s)
debconf: delaying package configuration, since apt-utils is not installed
(Reading database ... 14818 files and directories currently installed.)
Preparing to unpack .../kubeadm_1.27.0-00_amd64.deb ...
Unpacking kubeadm (1.27.0-00) over (1.26.0-00) ...
Setting up kubeadm (1.27.0-00) ...

root@node01 ~ ➜  kubeadm upgrade node
[upgrade] Reading configuration from the cluster...
[upgrade] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks
[preflight] Skipping prepull. Not a control plane node.
[upgrade] Skipping phase. Not a control plane node.
[upgrade] Backing up kubelet config file to /etc/kubernetes/tmp/kubeadm-kubelet-config3105983606/config.yaml
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[upgrade] The configuration for this node was successfully updated!
[upgrade] Now you should go ahead and upgrade the kubelet package using your package manager.

root@node01 ~ ➜  apt-get install kubelet=1.27.0-00 
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following held packages will be changed:
  kubelet
The following packages will be upgraded:
  kubelet
1 upgraded, 0 newly installed, 0 to remove and 91 not upgraded.
Need to get 18.8 MB of archives.
After this operation, 15.1 MB disk space will be freed.
Do you want to continue? [Y/n] y
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubelet amd64 1.27.0-00 [18.8 MB]
Fetched 18.8 MB in 0s (45.2 MB/s)
debconf: delaying package configuration, since apt-utils is not installed
(Reading database ... 14818 files and directories currently installed.)
Preparing to unpack .../kubelet_1.27.0-00_amd64.deb ...
/usr/sbin/policy-rc.d returned 101, not running 'stop kubelet.service'
Unpacking kubelet (1.27.0-00) over (1.26.0-00) ...
Setting up kubelet (1.27.0-00) ...
/usr/sbin/policy-rc.d returned 101, not running 'start kubelet.service'

root@node01 ~ ➜  systemctl daemon-reload

root@node01 ~ ➜  systemctl restart kubelet

root@node01 ~ ➜  exit
logout
Connection to node01 closed.
```
Remove the restriction and mark the worker node as schedulable again.


```
controlplane ~ ➜  kubectl uncordon node01
node/node01 uncordoned
```