Create a configmap that the new scheduler will employ using the concept of ConfigMap as a volume
```
kubectl create -f my-scheduler-configmap.yaml
```

Deploy an additional scheduler to the cluster
```
kubectl create -f my-scheduler.yaml
```

Run 
```
kubectl create -f nginx-pod.yaml
```