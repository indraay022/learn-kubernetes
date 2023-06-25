```
FROM python:3.6-alpine

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]

CMD ["--color", "red"]
```

```
apiVersion: v1 
kind: Pod 
metadata:
  name: webapp-green
  labels:
      name: webapp-green 
spec:
  containers:
  - name: simple-webapp
    image: kodekloud/webapp-color
    command: ["--color","green"]
```

The ENTRYPOINT in the Dockerfile is overridden by the command in the pod definition file.
Since the entrypoint is overridden in the pod definition, the command that will be run is just --color green.

```
controlplane ~ ➜  kubectl run webapp-green --image kodekloud/webapp-color -o yaml --dry-run=client --command -- python.app --color=green
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: webapp-green
  name: webapp-green
spec:
  containers:
  - command:
    - python.app
    - --color=green
    image: kodekloud/webapp-color
    name: webapp-green
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

controlplane ~ ➜  kubectl run webapp-green --image kodekloud/webapp-color -o yaml --dry-run=client -- --color=greenapiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: webapp-green
  name: webapp-green
spec:
  containers:
  - args:
    - --color=green
    image: kodekloud/webapp-color
    name: webapp-green
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

controlplane ~ ➜  
```