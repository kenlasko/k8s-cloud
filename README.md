# Kubernetes Install
Ensure that Omnictl/Talosctl is ready to go. Installation steps are [here](https://github.com/kenlasko/omni/).
## Install Kubernetes
1. Make sure all Talos nodes are in maintenance mode and are available in Omni, then create cluster:
```
omnictl cluster template sync -f ~/omni/cluster-template-cloud.yaml
```
2. Install Cilium, Cert-Manager, Sealed-Secrets and ArgoCD
```
ansible-playbook ~/k3s-cloud/_ansible/k3s-apps.yaml
```
3. Get initial ArgoCD password
```
kubectl -n argocd get secret argocd-initial-admin-secret \
          -o jsonpath="{.data.password}" | base64 -d; echo
```

## Get Kubernetes token
```
kubectl -n kube-system get secret kubeapi-service-account-secret \
          -o jsonpath="{.data.token}" | base64 -d; echo
```
