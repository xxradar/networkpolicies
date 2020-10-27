### Create a namespace
```
kubectl create ns laser
```

### Create a nginx-deployment
```
kubectl apply -n laser -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
      nodeSelector:
        beta.kubernetes.io/os: linux
EOF
```

### Create a demo pod
```
kubectl apply -n laser -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: debug
spec:
  containers:
  - name: debug
    image: docker.io/xxradar/hackon
    command: ["bash"]
    args: ["-c", "sleep 100000"]
  nodeSelector:
    kubernetes.io/os: linux
EOF
```
### Get IP address pod
```
kubectl get po -n laser -o wide
```
### Check connectivity
```
kubectl exec -it debug -n laser -- bash
```
