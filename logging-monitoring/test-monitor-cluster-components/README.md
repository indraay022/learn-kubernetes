# kubernetes-metrics-server
Development/Learning environment deployment of metrics-server

NOTE: DO NOT USE THIS FOR PRODUCTION USE CASES.
 This is an insecure deployment for quick deployment in a learning environment.

```
root@controlplane:~# cd kubernetes-metrics-server/
root@controlplane:~/kubernetes-metrics-server# kubectl create -f .
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
serviceaccount/metrics-server created
deployment.apps/metrics-server created
service/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
root@controlplane:~/kubernetes-metrics-server# 
```

To check resource usage
```
controlplane ~ ➜  kubectl top node
NAME           CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
controlplane   356m         0%     1166Mi          0%        
node01         283m         0%     309Mi           0%
```

To identify the node that consumes the most CPU(cores).
```
controlplane ~ ➜  kubectl top node --sort-by='cpu'
NAME           CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
controlplane   316m         0%     1175Mi          0%        
node01         25m          0%     308Mi           0%
```

To identify the node that consumes the most Memory(bytes).
```
controlplane ~ ➜  kubectl top node --sort-by='memory'
NAME           CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
controlplane   294m         0%     1175Mi          0%        
node01         29m          0%     308Mi           0%    
```

To identify the POD that consumes the most Memory(bytes).
```
controlplane ~ ➜  kubectl top pod --sort-by='memory'
NAME       CPU(cores)   MEMORY(bytes)   
rabbit     131m         252Mi           
elephant   20m          32Mi            
lion       1m           18Mi 
```

To identify the POD that consumes the least CPU(cores).
```
controlplane ~ ➜  kubectl top pod --sort-by='cpu' --no-headers | tail -1
lion       1m     18Mi  
```