## EGRESS policies for Kubernetes and ProjectCalico
In kubernetes network policies are namespaced (so you have to apply them to every namespace). So let's create a namespace to investigate things.
```
kubectl create ns my-demo-app

 kubectl run -it --rm --image xxradar/hackon -l allow-internet-egress=true -n my-demo-app my-app -- bash
```

### 1. Default Deny Egress policy

```
cat >default-deny-egress.yaml <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all-egress
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    ports:
    - protocol: TCP
      port: 53
    - protocol: UDP
      port: 53
EOF
```
```
kubectl apply -n my-demo-app -f default-deny-egress.yaml 
```

### 2. Allow access to the internet on port 80 and 443 for selected pods
```
cat >allow-80-443-egress.yaml <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-80-443-egress
spec:
  podSelector:
    matchLabels:
      allow-internet-egress: "true"
  policyTypes:
  - Egress
  egress:
  - {}
EOF
```
```
kubectl apply -n my-demo-app -f allow-80-443-egress.yaml 
```
Please note that network policies are applied even on running pods labeled with **allow-internet-egress="true"**

### 3. Deny Egress access via Calico Network sets

```
cat >networkset-external-api.yaml <<EOF
apiVersion: projectcalico.org/v3
kind: NetworkSet
metadata:
  name: networkset-api-access
  labels:
    role: api
spec:
  nets:
  - 198.199.124.250/24
  - 95.85.54.44/32
EOF
```
```
calicoctl apply -n my-demo-app -f networkset-external-api.yaml
```
```
cat >deny-external-api.yaml <<EOF
apiVersion: projectcalico.org/v3
kind: NetworkPolicy
metadata:
  name: external-api-access
spec:
  order: 100
  selector:
    allow-internet-egress == "true"
  types:
    - Egress
  egress:    
    - action: Deny
      destination:
        selector: role == "api"
EOF
```
```
calicoctl apply -n my-demo-app -f deny-external-api.yaml
```

### 4. Deny Egress access via Calico Global Network sets
```
cat >global-networkset-external-api.yaml <<EOF
apiVersion: projectcalico.org/v3
kind: GlobalNetworkSet
metadata:
  name: global-networkset-external-api
  labels:
    role: api-global
spec:
  nets:
  - 198.199.124.250/24
  - 95.85.54.44/32
EOF
```
```
calicoctl apply -f global-networkset-external-api.yaml
```
```
cat >global-deny-external-api.yaml <<EOF
apiVersion: projectcalico.org/v3
kind: NetworkPolicy
metadata:
  name: global-external-api-access
spec:
  order: 80
  selector:
    allow-internet-egress == "true"
  types:
    - Egress
  egress:    
    - action: Deny
      destination:
        selector: role == "api-global"
EOF
```
```
calicoctl apply -f global-deny-external-api.yaml
```
### 5. Deny Egress access based on FQDN
```
cat >dns-deny-external-api.yaml <<EOF
apiVersion: projectcalico.org/v3
kind: NetworkPolicy
metadata:
  name: allow-egress-to-domains
spec:
  order: 1
    allow-internet-egress == "true"
  types:
  - Egress
  egress:
  - action: Deny
    destination:
      domains:
      - api.slack.com
      - "*.radarhack.com"
EOF
```
```
calicoctl apply -f dns-deny-external-api.yam
```
