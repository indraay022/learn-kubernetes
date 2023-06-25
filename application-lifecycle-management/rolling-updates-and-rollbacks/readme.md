```
controlplane ~ ➜  kubectl replace -f deploy.yaml --force 
deployment.apps "frontend" deleted
deployment.apps/frontend replaced

controlplane ~ ➜  kubectl get deployment
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
frontend   4/4     4            4           16s
```