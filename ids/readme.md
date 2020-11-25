To remove the tracker block rule
```
 kubectl  delete  GlobalNetworkPolicy security.block-feodo
 ```


Create a security tier
```
kubectl apply -f security-tier.yaml
```
Apply the quarantine rule
```
kubectl apply -f quarantine.yaml
```
Create the threatfeed
```
kubectl apply -f feodo-tracker.yaml
```
Apply the  threatfeed block
```
kubectl apply -f feodo-tracker-block.yaml
```
Create a pass rule for next tier
```
kubectl apply -f security-pass.yaml 
```
