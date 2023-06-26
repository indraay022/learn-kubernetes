```
controlplane ~ ➜  kubectl get nodes
NAME           STATUS   ROLES           AGE     VERSION
controlplane   Ready    control-plane   6m58s   v1.27.0
node01         Ready    <none>          6m37s   v1.27.0

controlplane ~ ➜  kubectl get pod -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
blue-6b478c8dbf-8bfqc   1/1     Running   0          87s   10.244.1.3   node01         <none>           <none>
blue-6b478c8dbf-fvz8w   1/1     Running   0          87s   10.244.1.2   node01         <none>           <none>
blue-6b478c8dbf-n7l57   1/1     Running   0          87s   10.244.0.4   controlplane   <none>           <none>

controlplane ~ ➜  kubectl drain node01 --ignore-daemonsets
node/node01 cordoned
Warning: ignoring DaemonSet-managed Pods: kube-flannel/kube-flannel-ds-94css, kube-system/kube-proxy-hwr2w
evicting pod default/blue-6b478c8dbf-fvz8w
evicting pod default/blue-6b478c8dbf-8bfqc
pod/blue-6b478c8dbf-fvz8w evicted
pod/blue-6b478c8dbf-8bfqc evicted
node/node01 drained

controlplane ~ ➜  kubectl get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
blue-6b478c8dbf-2tfs7   1/1     Running   0          7m37s   10.244.0.5   controlplane   <none>           <none>
blue-6b478c8dbf-5r2ps   1/1     Running   0          7m37s   10.244.0.6   controlplane   <none>           <none>
blue-6b478c8dbf-n7l57   1/1     Running   0          13m     10.244.0.4   controlplane   <none>           <none>

# The pod got Evicted from node01 and then it moved to controlplane (because the controlplane have no taint)

controlplane ~ ➜  kubectl describe node controlplane | grep Taint
Taints:             <none>

controlplane ~ ➜  kubectl uncordon node01
node/node01 uncordoned

# Configure the node node01 to be schedulable again.

controlplane ~ ➜  kubectl get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
blue-6b478c8dbf-2tfs7   1/1     Running   0          10m   10.244.0.5   controlplane   <none>           <none>
blue-6b478c8dbf-5r2ps   1/1     Running   0          10m   10.244.0.6   controlplane   <none>           <none>
blue-6b478c8dbf-n7l57   1/1     Running   0          16m   10.244.0.4   controlplane   <none>           <none>

# Running the uncordon command on a node will not automatically schedule pods on the node. When new pods are created, they will be placed on node01.

controlplane ~ ➜  kubectl get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
blue-6b478c8dbf-2tfs7   1/1     Running   0          13m   10.244.0.5   controlplane   <none>           <none>
blue-6b478c8dbf-5r2ps   1/1     Running   0          13m   10.244.0.6   controlplane   <none>           <none>
blue-6b478c8dbf-n7l57   1/1     Running   0          19m   10.244.0.4   controlplane   <none>           <none>
hr-app                  1/1     Running   0          22s   10.244.1.4   node01         <none>           <none>

# hr-app pod manually scheduled without using replicaset / controllers on node01

controlplane ~ ➜  kubectl drain node01 --ignore-daemonsets
node/node01 cordoned
error: unable to drain node "node01" due to error:cannot delete Pods declare no controller (use --force to override): default/hr-app, continuing command...
There are pending nodes to be drained:
 node01
cannot delete Pods declare no controller (use --force to override): default/hr-app

# that kind of pod only can be evicted using --force command

controlplane ~ ✖ kubectl drain node01 --ignore-daemonsets --force
node/node01 already cordoned
Warning: deleting Pods that declare no controller: default/hr-app; ignoring DaemonSet-managed Pods: kube-flannel/kube-flannel-ds-94css, kube-system/kube-proxy-hwr2w
evicting pod default/hr-app
pod/hr-app evicted
node/node01 drained

# the pod will lost forever

controlplane ~ ➜  kubectl cordon node01
node/node01 cordoned

# Instead draining the pod, we use cordon command. This will ensure that no new pods are scheduled on this node and the existing pods will not be affected by this operation.
```