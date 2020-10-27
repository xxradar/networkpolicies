```
kubectl create ns laser-sa
```
```
kubectl apply -n laser-sa -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: debugger
EOF
```
```
kubectl apply -n laser-sa -f - <<EOF
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
  serviceAccountName: debugger
EOF
```
```
kubectl apply -n laser-sa -f - <<EOF
apiVersion: projectcalico.org/v3
kind: NetworkPolicy
metadata:
  name: demo-calico-sa
spec:
  selector: all()
  order: 500
  egress:
    - action: Allow
      source:
        serviceAccounts:
          names:
            - debugger
EOF
```
```
kubectl apply -n laser-sa -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: debug2
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
