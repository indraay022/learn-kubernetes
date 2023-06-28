We have a working Kubernetes cluster with a set of web applications running. Let us first explore the setup.  
```
controlplane ~ ➜  kubectl get all
NAME                        READY   STATUS    RESTARTS   AGE
pod/blue-6b478c8dbf-9fw9r   1/1     Running   0          35s
pod/blue-6b478c8dbf-lz7pk   1/1     Running   0          35s
pod/blue-6b478c8dbf-zjzmn   1/1     Running   0          35s
pod/red-6684f7669d-f5nct    1/1     Running   0          35s
pod/red-6684f7669d-jxf2z    1/1     Running   0          35s

NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/blue-service   NodePort    10.110.67.196    <none>        80:30082/TCP   35s
service/kubernetes     ClusterIP   10.96.0.1        <none>        443/TCP        2m24s
service/red-service    NodePort    10.104.230.195   <none>        80:30080/TCP   35s

NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/blue   3/3     3            3           35s
deployment.apps/red    2/2     2            2           35s

NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/blue-6b478c8dbf   3         3         3       35s
replicaset.apps/red-6684f7669d    2         2         2       35s
```
What is the version of ETCD running on the cluster?

```
controlplane ~ ➜  kubectl -n kube-system logs etcd-controlplane | grep -i 'etcd-version'
{"level":"info","ts":"2023-06-28T00:44:48.104Z","caller":"embed/etcd.go:306","msg":"starting an etcd server","etcd-version":"3.5.7","git-sha":"215b53cf3","go-version":"go1.17.13","go-os":"linux","go-arch":"amd64","max-cpu-set":36,"max-cpu-available":36,"member-initialized":false,"name":"controlplane","data-dir":"/var/lib/etcd","wal-dir":"","wal-dir-dedicated":"","member-dir":"/var/lib/etcd/member","force-new-cluster":false,"heartbeat-interval":"100ms","election-timeout":"1s","initial-election-tick-advance":true,"snapshot-count":10000,"max-wals":5,"max-snapshots":5,"snapshot-catchup-entries":5000,"initial-advertise-peer-urls":["https://192.3.72.9:2380"],"listen-peer-urls":["https://192.3.72.9:2380"],"advertise-client-urls":["https://192.3.72.9:2379"],"listen-client-urls":["https://127.0.0.1:2379","https://192.3.72.9:2379"],"listen-metrics-urls":["http://127.0.0.1:2381"],"cors":["*"],"host-whitelist":["*"],"initial-cluster":"controlplane=https://192.3.72.9:2380","initial-cluster-state":"new","initial-cluster-token":"etcd-cluster","quota-backend-bytes":2147483648,"max-request-bytes":1572864,"max-concurrent-streams":4294967295,"pre-vote":true,"initial-corrupt-check":true,"corrupt-check-time-interval":"0s","compact-check-time-enabled":false,"compact-check-time-interval":"1m0s","auto-compaction-mode":"periodic","auto-compaction-retention":"0s","auto-compaction-interval":"0s","discovery-url":"","discovery-proxy":"","downgrade-check-interval":"5s"}

controlplane ~ ➜  kubectl -n kube-system describe pod etcd-controlplane | grep Image:
    Image:         registry.k8s.io/etcd:3.5.7-0
```
At what address can you reach the ETCD cluster from the controlplane node?

```
controlplane ~ ➜  kubectl -n kube-system describe pod etcd-controlplane | grep '\--listen-client-urls'
      --listen-client-urls=https://127.0.0.1:2379,https://192.3.72.9:2379

```
Where is the ETCD server certificate file located?

```
controlplane ~ ➜  kubectl -n kube-system describe pod etcd-controlplane | grep '\--cert-file'
      --cert-file=/etc/kubernetes/pki/etcd/server.crt

```
Where is the ETCD CA Certificate file located?
```
controlplane ~ ➜  kubectl -n kube-system describe pod etcd-controlplane | grep '\--trusted-ca-file'
      --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt

```
Where is the ETCD key file located?
```
controlplane ~ ➜ kubectl -n kube-system describe pod etcd-controlplane | grep '\--key-file'
      --key-file=/etc/kubernetes/pki/etcd/server.key

```
Take a snapshot of the ETCD database using the built-in snapshot functionality.  
Store the backup file at location `/opt/snapshot-pre-boot.db`
```
controlplane ~ ➜  ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 \
> --cacert=/etc/kubernetes/pki/etcd/ca.crt \
> --cert=/etc/kubernetes/pki/etcd/server.crt \
> --key=/etc/kubernetes/pki/etcd/server.key \
> snapshot save /opt/snapshot-pre-boot.db
Snapshot saved at /opt/snapshot-pre-boot.db

```

```
controlplane ~ ➜  ETCDCTL_API=3 etcdctl  --data-dir /var/lib/etcd-from-backup \
> snapshot restore /opt/snapshot-pre-boot.db
2023-06-27 21:20:32.703889 I | mvcc: restore compact to 2480
2023-06-27 21:20:32.711353 I | etcdserver/membership: added member 8e9e05c52164694d [http://localhost:2380] to cluster cdf818194e3a8c32

```

Lets say there is error on the cluster, and the deployment, services, and pods are not show up.
```
controlplane ~ ➜  kubectl get all -A
NAMESPACE      NAME                                       READY   STATUS    RESTARTS   AGE
kube-flannel   pod/kube-flannel-ds-p4wrn                  1/1     Running   0          8m29s
kube-system    pod/coredns-5d78c9869d-bp7wr               1/1     Running   0          8m35s
kube-system    pod/coredns-5d78c9869d-szwwv               1/1     Running   0          8m35s
kube-system    pod/etcd-controlplane                      1/1     Running   0          8m51s
kube-system    pod/kube-apiserver-controlplane            1/1     Running   0          8m52s
kube-system    pod/kube-controller-manager-controlplane   1/1     Running   0          8m49s
kube-system    pod/kube-proxy-xlhxp                       1/1     Running   0          8m35s
kube-system    pod/kube-scheduler-controlplane            1/1     Running   0          8m50s

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  2m12s
kube-system   service/kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   8m48s

NAMESPACE      NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-flannel   daemonset.apps/kube-flannel-ds   1         1         1       1            1           <none>                   8m29s
kube-system    daemonset.apps/kube-proxy        1         1         1       1            1           kubernetes.io/os=linux   8m48s

NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/coredns   2/2     2            2           8m48s

NAMESPACE     NAME                                 DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/coredns-5d78c9869d   2         2         2       8m35s
```

In this case, we are restoring the snapshot to a different directory but in the same server where we took the backup (the controlplane node) As a result, the only required option for the restore command is the `--data-dir`.  

Next, update the `/etc/kubernetes/manifests/etcd.yaml`:  
We have now restored the etcd snapshot to a new path on the controlplane - `/var/lib/etcd-from-backup`, so, the only change to be made in the YAML file, is to change the hostPath for the volume called etcd-data from old directory (`/var/lib/etcd`) to the new directory (`/var/lib/etcd-from-backup`).  

```
  volumes:
  - hostPath:
      path: /var/lib/etcd-from-backup
      type: DirectoryOrCreate
    name: etcd-data
```

```
controlplane ~ ➜  nano /etc/kubernetes/manifests/etcd.yaml 

controlplane ~ ➜  cat /etc/kubernetes/manifests/etcd.yaml 
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/etcd.advertise-client-urls: https://192.12.61.9:2379
  creationTimestamp: null
  labels:
    component: etcd
    tier: control-plane
  name: etcd
  namespace: kube-system
spec:
  containers:
  - command:
    - etcd
    - --advertise-client-urls=https://192.12.61.9:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
    - --client-cert-auth=true
    - --data-dir=/var/lib/etcd-from-backup 
    - --experimental-initial-corrupt-check=true
    - --experimental-watch-progress-notify-interval=5s
    - --initial-advertise-peer-urls=https://192.12.61.9:2380
    - --initial-cluster=controlplane=https://192.12.61.9:2380
    - --key-file=/etc/kubernetes/pki/etcd/server.key
    - --listen-client-urls=https://127.0.0.1:2379,https://192.12.61.9:2379
    - --listen-metrics-urls=http://127.0.0.1:2381
    - --listen-peer-urls=https://192.12.61.9:2380
    - --name=controlplane
    - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
    - --peer-client-cert-auth=true
    - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --snapshot-count=10000
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    image: registry.k8s.io/etcd:3.5.7-0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /health?exclude=NOSPACE&serializable=true
        port: 2381
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: etcd
    resources:
      requests:
        cpu: 100m
        memory: 100Mi
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 127.0.0.1
        path: /health?serializable=false
        port: 2381
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /var/lib/etcd-from-backup
      name: etcd-data
    - mountPath: /etc/kubernetes/pki/etcd
      name: etcd-certs
  hostNetwork: true
  priority: 2000001000
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd-from-backup
      type: DirectoryOrCreate
    name: etcd-data
status: {}
```

Note 1: As the ETCD pod has changed it will automatically restart, and also kube-controller-manager and kube-scheduler. Wait 1-2 to mins for this pods to restart. You can run the command: watch "crictl ps | grep etcd" to see when the ETCD pod is restarted.

Note 2: If the etcd pod is not getting Ready 1/1, then restart it by kubectl delete pod -n kube-system etcd-controlplane and wait 1 minute.

Note 3: This is the simplest way to make sure that ETCD uses the restored data after the ETCD pod is recreated. You don't have to change anything else.
```
controlplane ~ ➜  kubectl get pod -A
NAMESPACE      NAME                                   READY   STATUS    RESTARTS   AGE
default        blue-6b478c8dbf-jb2rn                  1/1     Running   0          4m49s
default        blue-6b478c8dbf-jz9xz                  1/1     Running   0          4m49s
default        blue-6b478c8dbf-qfgtj                  1/1     Running   0          4m49s
default        red-6684f7669d-gnvm2                   1/1     Running   0          4m49s
default        red-6684f7669d-n6cqx                   1/1     Running   0          4m49s
kube-flannel   kube-flannel-ds-gmjls                  1/1     Running   0          9m3s
kube-system    coredns-5d78c9869d-nrmmt               1/1     Running   0          9m3s
kube-system    coredns-5d78c9869d-t2p52               1/1     Running   0          9m3s
kube-system    etcd-controlplane                      1/1     Running   0          9m20s
kube-system    kube-apiserver-controlplane            1/1     Running   0          9m16s
kube-system    kube-controller-manager-controlplane   1/1     Running   0          9m14s
kube-system    kube-proxy-svqx2                       1/1     Running   0          9m3s
kube-system    kube-scheduler-controlplane            1/1     Running   0          9m18s
```