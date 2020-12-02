### Enable hostEndpoint autocreation
```
calicoctl patch kubecontrollersconfiguration default --patch='{"spec": {"controllers": {"node": {"hostEndpoint": {"autoCreate": "Enabled"}}}}}'
```
Verify
```
calicoctl get heps -o wide
```

### Label a node for testing

```
kubectl get node
```
```
kubectl label nodes --all kubernetes-host=
kubectl label node ip-10-11-2-73 environment=dev
```
Verify
```
kubectl get no --show-labels
````



### Create a policy 
```
calicoctl apply -f - << EOF
apiVersion: projectcalico.org/v3
kind: GlobalNetworkPolicy
metadata:
  name: default.some-nodes-policy
spec:
  tier: default
  selector: (has(kubernetes-host)&&environment == "dev")
  namespaceSelector: ''
  serviceAccountSelector: ''
  ingress:
    - action: Allow
      protocol: TCP
      source: {}
      destination:
        ports:
          - '9999'
          - '10250'
    - action: Allow
      source: {}
      destination:
        nets:
          - 127.0.0.1/32
    - action: Deny
      protocol: TCP
      source: {}
      destination:
        ports:
          - '9998'
  doNotTrack: false
  applyOnForward: true
  preDNAT: true
  types:
    - Ingress
EOF
```

Create a testing pod
```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: debug
spec:
  hostNetwork: true
  containers:
  - name: debug
    image: docker.io/xxradar/hackon
    command: ["bash"]
    args: ["-c", "sleep 100"]
    ports:
    - containerPort: 9999
  nodeSelector:
    kubernetes.io/hostname: ip-10-11-2-73
EOF
```

Create a testing nginx deployment 
```
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: radarhack-deployment
  labels:
    app: radarhack
spec:
  replicas: 3
  selector:
    matchLabels:
      app: radarhack
  template:
    metadata:
      labels:
        app: radarhack
    spec:
      hostNetwork: true
      containers:
      - name: radarhack
        image: docker.io/xxradar/naxsi5
        ports:
         - containerPort: 9999
           protocol: TCP
           name: http
EOF
```
