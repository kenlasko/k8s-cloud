# Introduction
[Argo CD](https://github.com/argoproj/argo-cd) is the tool that declaratively manages every application running in the Kubernetes cluster. Every configuration setting is stored in this Github repo, which means that the entire cluster can be rapidly rebuilt without much work.

This deployment makes use of the Argo CD "app of apps" approach, which theoretically means that once Argo CD is bootstrapped and connected to the repository, every other application can automatically be installed and configured. In actual fact, this is only partially true. Once Argo CD is running, the remaining applications have to be installed in a rather specific order, owing to the requirement of restoring [MariaDB databases](/manifests/database/mariadb) before many workloads can function. I am working towards full automation, but am not there yet.

The app-of-apps approach has two "sections": 

## argocd-apps
The [argocd-apps](/argocd-apps) folder contains the application definitions for all the applications that do not depend on Longhorn volumes. This also includes Longhorn, ironically. This is "generated" by the [argocd-apps](/argocd/argocd-apps.yaml) application. Each application has to be manually triggered and generally follows the pattern of:
* app prerequisites such as [sealed-secrets](/manifests/system/sealed-secrets), [cert-manager](/manifests/system/cert-manager), [cloudflare-tunnel](/manifests/network/cloudflare-tunnel) etc.
* [MariaDB](/manifests/database/mariadb) databases 
* other applications such as [VaultWarden](/manifests/apps/vaultwarden), [UCDialplans](/manifests/apps/ucdialplans) etc which rely on databases but can start without the databases present without much ill-effect (other than a possible pod restart once the database is restored)

I am experimenting with [Argo CD sync-waves](https://argo-cd.readthedocs.io/en/stable/user-guide/sync-waves/) to control the order of installation of these tools, so the installation can be automated.

# Authentication
ArgoCD is configured to use Github authentication instead of the built-in admin account. Github authentication uses [Github OAuth Apps](https://github.com/settings/developers), which is found under `User - Settings - Developer settings - OAuth Apps`. Steps to configure: https://argo-cd.readthedocs.io/en/stable/operator-manual/user-management/#dex

If local authentication needs to be re-enabled, first set `data.admin.enabled = "true"` in [configmap.yaml](/manifests/argocd/configmap.yaml), then login with the normal credentials. If this is the first time logging in, get the initial admin password via
```
kubectl -n kube-system get secret kubeapi-service-account-secret -o jsonpath="{.data.token}" | base64 -d; echo
```

# Adding a new Helm chart installation
1. Edit [values.yaml](/argocd/values.yaml) and add a Helm repository under `configs.repositories`
2. Create an Argo CD application in [argocd-apps](/argocd-apps) using an existing yaml as a template

# Troubleshooting
## ArgoCD apps showing as `Unknown`
Restart ArgoCD pods in this order:
1. argocd-dex-server
2. argocd-application-controller
3. argocd-server

## ArgoCD seems very slow to sync
Probably due to Unifi blocking SSH traffic due to `ET SCAN Potential SSH Scan OUTBOUND`. Resolve by adding the node IP to the Signature Suppression for this rule: https://unifi.ucdialplans.com/network/default/settings/security/cybersecure

