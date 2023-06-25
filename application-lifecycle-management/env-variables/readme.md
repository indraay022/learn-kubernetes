Create a new ConfigMap for the webapp-color POD. Use the spec given below.
- ConfigMap Name: webapp-config-map  
- Data: APP_COLOR=darkblue  
- Data: APP_OTHER=disregard
```
kubectl create configmap webapp-config-map --from-literal=APP_COLOR=darkblue --from-literal=APP_OTHER=disregard
```

```
controlplane ~ ➜  kubectl create configmap webapp-config-map --from-literal=APP_COLOR=darkblue --from-literal=APP_OTHER=disregard
configmap/webapp-config-map created

controlplane ~ ➜  kubectl get configmaps webapp-config-map -o yaml
apiVersion: v1
data:
  APP_COLOR: darkblue
  APP_OTHER: disregard
kind: ConfigMap
metadata:
  name: webapp-config-map
  namespace: default
```