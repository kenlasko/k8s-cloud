# Configuring Replication from Home PostgreSQL Cluster
Before creating the cluster, you must first create a secret that contains the following certificates from the parent cluster:
- ```tls.crt``` and ```tls.key``` from ```postgres-replication``` secret
- ```ca.crt``` from ```postgres-ca```

```
kubectl get secret -n postgresql home-replication -o json | yq -P '.data'
kubectl get secret -n postgresql home-ca -o json | yq -P '.data'
```


All three can be in a single secret that looks like this:
```
---
apiVersion: v1
kind: Secret
metadata:
  name: replication-certs
  namespace: postgresql
type: Opaque
data:
  ca.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JS...
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1J...
  tls.key: LS0tLS1CRUdJTiBFQyBQUklWQVRFIEtFWS0tLS0tC...
```
Then use ```kubeseal``` to generate the sealed-secret:
```
kubeseal -f secret.yaml -w sealed-secrets.yaml --cert sealed-secret-signing-key.crt
```
Can be appended to any other existing sealed-secret yaml file in the namespace.