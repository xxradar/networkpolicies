### Create a namespace
```
kubectl create ns dev-ns
```

### Create a service ServiceAccount
```
kubectl apply -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dev-user
  namespace: dev-ns
EOF
```

### Create a a role 
```
kubectl apply -f - <<EOF
kind: Role
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: dev-ns-user-full-access
  namespace: dev-ns
rules:
- apiGroups: ["", "extensions", "apps", "networking.k8s.io"]
  resources: ["*", "networkpolicies"]
  verbs: ["*"]
- apiGroups: ["batch"]
  resources:
  - jobs
  - cronjobs
  verbs: ["*"]
EOF
```

### Create a rolebinding 
```
kubectl apply -f - <<EOF
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: mynamespace-user-view
  namespace: dev-ns
subjects:
- kind: ServiceAccount
  name: dev-user
  namespace: dev-ns
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: dev-ns-user-full-access
EOF
```

### Create a kubeconfig file
```
kubectl describe sa dev-user -n dev-ns
```
```
kubectl get secret dev-user-token-t8l2v -n dev-ns -o "jsonpath={.data.token}" | base64 -d
eyJhbGciOiJSUzI1NiIsImtpZCI6ImVDWUtrckFXdm4xMVRMQURIMFhEZDQ1dUFWMGdZUUlPQWhhZncifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3xxxxxxxxXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZXYtbnMiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlY3JldC5uYW1lIjoiZGV2LXVzZXItdG9rZW4tdDhsMnYiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGV2LXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI2ZTBlMDI2YS0xOWU2LTQ0ZTAtYTRmMC1kYjkxNjJiZGQ3NzMiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGV2LW5zOmRldi11c2VyIn0.fr74ItNb3QCokHnu-TJ27AMooGGo9TC58_160LQuK1FU_Uhn5U_shJV5-iGFmJ3YgRqfxZ5735J_gR8uPS1-BuzK5u9d_bKzdKJKhX1C8aV7_DRI4_zJMDEG7d5Ntz5ui4xs5Tnag25FKp1c5rkvAynGMP4W7ZyJbBjGc7fkohCHb46cmcpBQqZWaJNorS2JeXpRznJZa-_4OuslpDUexVQO6zHWj4pTmYq0WBSjgHQ_q9gzoMdP764m190B5YEDv7lFuo4afcB2haBBkBCCc4NMEb4F6NYj-H3i2TD_ctX3doDXkY84gqf-7rvOKV4vpRRVLxnG-TNthyMRSK2JnrLOKi2hQGivJlkqFyjBL3LqsaARNugcRJuWXypFTR-qSZW5G9JwcyWsPjqs970a6mBZkv7VftDyQNRj3eeDkWM6Jzqhy-nO4WsLCfcqP20pG6kX19b7nv756IDfYUOoo7QR383MNvA-_P_Mo86u4fkdaNy1_OtoGuQAXD1xKlUBXH0c6r00mZPsLIk4L7_4gqXDsO9uj1y9jyadLT3dWc5XS8wQe1mFuUg684ONEKBe8-MRTmi6D9k8DfZ87rTU2ZyBUy48pJy3G3aIeZk8jjEclwlEgt0MRrhcfYO8825LlwLEbimehvbzJ3Gn0jB-_kROv7UfR5v7aqxs_ZJJaFs
```
```
kubectl get secret dev-user-token-t8l2v -n dev-ns -o "jsonpath={.data['ca\.crt']}"
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCxxxx9pNFpVczAvZ283ajVtV01FaTVPWjB3RFFZSktvWklodmNOQVFFTEJRQXcKRFRFTE1Ba0dBMVVFQXhNQxxxxxx
```

```
vi mykube.conf

apiVersion: v1
kind: Config
preferences: {}

# Define the cluster
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSxxxxx9pNFpVczAvZ283ajVtV01FaTVPWjB3RFFZSktvWklodmNOQVxBMVVFQXhNQ1kyRXdJQmNOTWpBeE1USXdNRGcxTmpFeFdoZ1BNakExTURFeE1qQXdPVEEyTVRGYQpNQTB4Q3pBSkJnTlZCQU1UQW1OaE1JSUNJakFOQmdrcWhraUc5dzBCQVFFRkFBT0NBZzhBTUlJQ0NnS0NBZ0VBCnhmL0VENnE3aE9NQ09pNld6VHI3cXFmWWpKZ2FadG9wZldqVVVYSGdNcFZIVmNsNzVieGo0Skh1dWR3Y3ozb2QKYzFqblIrcERlKzFDSlJOMTd3Vit1dXpRTmNVUXpTQU96S3h4OEZaZ1FrdEFlckZNLzBHVFppeVNNcEh6V0JWbQpHOXlQVlJGUE16d0J5SGhvdTRFSjB2Skw0L2szSUZDRWtoN3hSN2JBY0g0TlFrSW0xSmQzNUtBajNvS1dVMzQ1CjJTNmgxWjlpNFZDTENoVFk0dk1pd1p5V25qTjFVVjdvWi9lbVNpRHFwTDF1Q1dIL0E4bk51UzBzNldTUTYzWHQKZVc4ajVqWHFXdVZTZUQ4SU1jYmUyVUZmVFRQUm9ZMkxjNnJTSnh4K0Z2YTVqbmYxcXJJTW9aNzRjdjhMMFVmYwpORHd5OTdodWtQK1lWY2syWXM5SFAwcGV6M0h6NExsZGs2bXZjUXVRVHpmdURzbU45eG1HdGUxSDROa1BxekNRCnpwdXIzRWw0Z3p2bU54VkxpUHpIc1hxKy9nQ2oyb1dCa1QzKzB2eTZ2N3ZSZVFobFJMZGIwWmtvT2p1Mk5MdUEKekdCZnMyVmtlOGxpbVZ6TXJBbjVWRE1Ea1VYT0lRL3h1SENwVFVYM094Y3RlVld2M09WcjVuUHdaVTJuRThhNgpLYXBqTHFKUW5QVHlBTFBYelFrZlc3cEcxR1M0TEJjb1FYZHMvZCtUYXQxb1pmWlNWQk1DVHVpVHdxNkNFRERmCnhLc1FLN2g5R0kyZHZoU3B6NTVod3pCcitkREJZMDZ2cHRNcW9ySXlwWDBMOFRxVzRuZVNGM0dTMU8vdlNaOVUKWVV0WGpJY0RHYWZYemh4U3ZCeFFWUUtsbHBXN0lBTzNOeU53ZFYxQU43a0NBd0VBQWFNak1DRXdEZ1lEVlIwUApBUUgvQkFRREFnS2tNQThHQTFVZEV3RUIvd1FGTUFNQkFmOHdEUVlKS29aSWh2Y05BUUVMQlFBRGdnSUJBRnpBClREYjFyay8vZW0rVjFKVHhKTTUyK3lMWUc0NnpicUxabHdvRCtIYkcvVmVNYnpnSVJ3VEFoNVRUVTRXVVRKNWEKQ2pNdXlyM1FhcnBDL0JpcUxhT3BlRy9ZL2J3YmpIT1Q3S1IreElEQkFXbkFLY1puUVlZM0x5VVl5bThVNXVWdgpoTUg5R2FHNzBTOTE5UWl3alJnaFNEQnF3L2ttQy9jZkJhbTYrWWZLR1NuK2xtYTBOQjFiNTd2ZzUzQjdoK1A0ClFxQ0U2ZTkzTHhJc3Rla2JONmQxcndnTjBBSVBxQWthNlhNWEN6U2taOE16VkxUbnpIUWd5RDdiakZVeDVFNHkKbjJzR0hXTnd3VDFPUXQvc0I3OTNsclEzVlRRZXZhS0VwTzY1MFVtYkZhNGFLZW1Ddy9nNkwvMVJxdGRTSXFjKwoxN2FoaENIUEJpQ2tKeVprbnVHeU1EM2pQbFB1UldjRHlHeHNRN2VmSEVOaUlmdDRuZ0YwMXA3ZFVLd2ZFUDkwCjZWRDhiSFY5UkNuMXg0cHN0azJtbWZIVjNXSDdFay9uelVpRGhzSys4dUJ4aUsyU2tpV3huUUVwSGpVSWZlamIKTnZRZUNsTDJ1emlDQUJLenNobndXOHpmSi80bDFWYnA1bU82ZHRWZ0hsd0p3K1o3QWVMaXlScUVxSEtrTDRCawp0YTYwTTJQQTZHaVJKRTI0Z1F2MEU5V3BWSTlNRFNnNzd6MWdwZmEvRTBkeldjdHd3T1RwY2N6dnlYSzFtMnJXCjFBRkdvRkYxa2FKSElGNlEzNVRYYndKYlE1b3pwSng0czdCOTBoZnUyWnFXcTloaHJaYS9mSWtPVDlqQWEwc2cKbllVajRsNmFFZ1NBUjVkcStJVWxVOENsRndZUU1uZXcrMkQ5bHBQUgotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    # You'll need the API endpoint of your Cluster here:
    server:  https://mycluster:6443
  name:  mycluster

# Define the user
users:
- name: dev-user
  user:
    as-user-extra: {}
    token: eyJhbGciOiJSUzI1NiIsImtpZCI6ImVDWUtrckFXdxxxxZncifQ.eyJpc3MiOiJrxxxxYWNjb3VudC51aWQiOiI2ZTBlMDI2YS0xOWU2LTQ0ZTAtYTRmMCxIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGV2LW5zOmRldi11c2VyIn0.fr74ItNb3QCokHnu-TJ27AMooGGo9TC58_160LQuK1FU_Uhn5U_shJV5-iGFmJ3YgRqfxZ5735J_gR8uPS1-BuzK5u9d_bKzdKJKhX1C8aV7_DRI4_zJMDEG7d5Ntz5ui4xs5Tnag25FKp1c5rkvAynGMP4W7ZyJbBjGc7fkohCHb46cmcpBQqZWaJNorS2JeXpRznJZa-_4OuslpDUexVQO6zHWj4pTmYq0WBSjgHQ_q9gzoMdP764m190B5YEDv7lFuo4afcB2haBBkBCCc4NMEb4F6NYj-H3i2TD_ctX3doDXkY84gqf-7rvOKV4vpRRVLxnG-TNthyMRSK2JnrLOKi2hQGivJlkqFyjBL3LqsaARNugcRJuWXypFTR-qSZW5G9JwcyWsPjqs970a6mBZkv7VftDyQNRj3eeDkWM6Jzqhy-nO4WsLCfcqP20pG6kX19b7nv756IDfYUOoo7QR383MNvA-_P_Mo86u4fkdaNy1_OtoGuQAXD1xKlUBXH0c6r00mZPsLIk4L7_4gqXDsO9uj1y9jyadLT3dWc5XS8wQe1mFuUg684ONEKBe8-MRTmi6D9k8DfZ87rTU2ZyBUy48pJy3G3aIeZk8jjEclwlEgt0MRrhcfYO8825LlwLEbimehvbzJ3Gn0jB-_kROv7UfR5v7aqxs_ZJJaFs

# Define the context: linking a user to a cluster
contexts:
- context:
    cluster:  mycluster
    namespace: dev-ns
    user: dev-user
  name: dev-ns

# Define current context
current-context: dev-ns

```

### Switch context
```
export KUBECONFIG=$PWD/mykube.conf
```
```
kubectl run  --image nginx nginxwww
```
This will fail ...
```
kubectl apply -n default -f - <<EOF
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
This will work ...
```
kubectl apply -n dev-ns -f - <<EOF
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

