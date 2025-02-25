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
