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

### Apply a INGRESS (1) Deny All netpol on the nginx deployment and (2) allow port 80
```
kubectl apply -n laser -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector:
    matchLabels:
      app: nginx
  policyTypes:
   - Ingress
   - Egress
EOF
```
```
kubectl apply -n laser -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-http
spec:
  podSelector:
    matchLabels:
      app: nginx
  ingress:
  - from:
    - podSelector:
        matchLabels: {}
    ports:
    - protocol: TCP
      port: 80
EOF
```
```
kubectl get netpol -n laser
```

### Apply a INGRESS (1) Default Deny All netpol and (2) allow port 80 for debug pod and (3) control egress from pods
```
kubectl delete netpol -n laser allow-http
kubectl delete netpol -n laser default-deny
```
```
kubectl apply -n laser -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector:
    matchLabels: {}
  policyTypes:
   - Ingress
   - Egress
EOF
```
```
kubectl apply -n laser -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-http-debug
spec:
  podSelector:
    matchLabels:
      app: nginx
  ingress:
  - from:
    - podSelector:
        matchLabels:
          debug: "true"
    ports:
    - protocol: TCP
      port: 80
EOF
```
```
kubectl apply -n laser -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-http-egress
spec:
  podSelector:
    matchLabels:
      debug: "true"
  egress:
  - to:
    - podSelector:
        matchLabels: {}
EOF
```
### Create a global quarantine rule
```
kubectl apply -f - <<EOF
apiVersion: projectcalico.org/v3
kind: GlobalNetworkPolicy
metadata:
  name: default.quarantine
spec:
  tier: default
  order: 490
  selector: quarantine == "true"
  namespaceSelector: ''
  serviceAccountSelector: ''
  ingress:
    - action: Deny
      source: {}
      destination: {}
  egress:
    - action: Deny
      source: {}
      destination: {}
  doNotTrack: false
  applyOnForward: false
  preDNAT: false
  types:
    - Ingress
    - Egress
EOF
```
```
kubectl get po -n laser --show-labels
kubectl exec -it debug -n laser -- bash 
```
```
kubectl label po debug -n laser quarantine="true" --overwrite
kubectl exec -it debug -n laser -- bash 
```
```
