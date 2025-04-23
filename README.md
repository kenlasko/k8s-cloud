# Introduction
This is the Git repository that contains all the configuration for my cloud-based single-node Kubernetes cluster. The cluster is hosted on a single node in [Oracle Cloud](https://cloud.oracle.com). 

This cluster is used as a disaster-recovery solution for my [home-based Kubernetes cluster](https://github.com/kenlasko/k8s). It replicates the function of some critical services:
* MariaDB
* AdGuard Home
* VaultWarden
* UCDialplans website

Most of the services are in warm-standby mode. AdGuard Home is the only actively used service for when I am away from home as it responds to requests from *.dns.ucdialplans.com. However, it is very lightly used, since my phone is usually connected to home via Wireguard. 

## Related Repositories
Links to my other repositories mentioned or used in this repo:
- [K8s Bootstrap](https://github.com/kenlasko/k8s-bootstrap): Bootstraps Kubernetes clusters with essential apps using Terraform/OpenTofu
- [K8s Cluster Configuration](https://github.com/kenlasko/k8s): Manages Kubernetes cluster manifests and workloads.
- [NixOS](https://github.com/kenlasko/nixos-wsl): A declarative OS modified to support my Kubernetes cluster
- [Omni](https://github.com/kenlasko/omni): Creates and manages the Kubernetes clusters.

## Software Updates
All software updates (excluding Kubernetes and OS) are managed via [Renovate](https://github.com/renovatebot/renovate). Renovate watches the Github repo and checks for software version updates on any Helm chart, ArgoCD application manifest or deployment manifest. If an update is found, Renovate will update the version in the repo and let ArgoCD handle the actual upgrade. All updates are logged in the repo as commits.

The configuration for Renovate is stored in [renovate.json](/renovate.json). The dashboard is available at https://developer.mend.io/github/kenlasko

Renovate is set to automatically and silently upgrade every software package EXCEPT for the following:
* [Cilium](/manifests/network/cilium)

When upgrades for the above packages are found, Renovate will create a pull request that has to be manually approved (or denied). Once approved, ArgoCD manages the actual upgrade as with any other software.

# Cluster Installation Prerequisites
SideroLabs Omni must be ready to go. Installation steps are in the repository link below:
[Omni On-Prem installation and configuration](https://github.com/kenlasko/omni/)

You will need a workstation (preferably Linux-based) with several tools to get things rolling:
- [Workstation Prep Instructions for Ubuntu-based distributions](/docs/WORKSTATION.md)
- [NixOS Workstation Build](https://github.com/kenlasko/nixos-wsl/)


# Node Prep
1. Download ARM64 Talos Oracle image from https://omni.ucdialplans.com and place in /home/ken/
2. Create image metadata file and save as ```image_metadata.json```
```
{
    "version": 2,
    "externalLaunchOptions": {
        "firmware": "UEFI_64",
        "networkType": "PARAVIRTUALIZED",
        "bootVolumeType": "PARAVIRTUALIZED",
        "remoteDataVolumeType": "PARAVIRTUALIZED",
        "localDataVolumeType": "PARAVIRTUALIZED",
        "launchOptionsSource": "PARAVIRTUALIZED",
        "pvAttachmentVersion": 2,
        "pvEncryptionInTransitEnabled": true,
        "consistentVolumeNamingEnabled": true
    },
    "imageCapabilityData": null,
    "imageCapsFormatVersion": null,
    "operatingSystem": "Talos",
    "operatingSystemVersion": "1.8.1",
    "additionalMetadata": {
        "shapeCompatibilities": [
            {
                "internalShapeName": "VM.Standard.A1.Flex",
                "ocpuConstraints": null,
                "memoryConstraints": null
            }
        ]
    }
}
```
3. Create .oci image for Oracle
```
xz --decompress oracle-arm64-omni-onprem-omni-v1.8.1.qcow2.xz
tar zcf talos-oracle-arm64.oci oracle-arm64.qcow2 image_metadata.json
```
4. Copy image to [Oracle storage bucket](https://cloud.oracle.com/object-storage/buckets?region=ca-toronto-1)
5. [Import custom image](https://cloud.oracle.com/compute/images?region=ca-toronto-1) to Oracle cloud
6. Edit image capabilities and set ```Boot volume type``` and ```Local data volume``` to ```PARAVIRTUALIZED```
7. [Create instance](https://cloud.oracle.com/compute/instances?region=ca-toronto-1) using custom image
8. Open all inbound firewall ports from home network
9. Set Oracle public IP address on [Unifi port forwarding](https://unifi.ucdialplans.com/network/default/settings/security/port-forwarding)

# Kubernetes Install
Ensure that Omnictl/Talosctl is ready to go. Installation steps are [here](https://github.com/kenlasko/omni/).

## Install Kubernetes
This guide assumes you're using a NixOS distribution that is configured to securely store and present all required certificates. For more information, see [my NixOS repo](https://github.com/kenlasko/nixos-wsl/).

1. Make sure all Talos nodes are in maintenance mode and appearing in [Omni](https://omni.ucdialplans.com). Use network boot via [NetBootXYZ](https://github.com/kenlasko/pxeboot/) to boot nodes into Talos maintenance mode.
2. Create cluster via `omnictl`:
```
omnictl cluster template sync -f ~/omni/cluster-template-cloud.yaml
```
3. Set the proper context with kubectl and verify you see the expected nodes
```
kubectl config use-context omni-cloud
kubectl get nodes
```
4. [Bootstrap cluster](https://github.com/kenlasko/k8s-bootstrap) by installing Cilium, Cert-Manager, Sealed-Secrets and ArgoCD via OpenTofu/Terraform
```
cd ~/terraform
tf workspace new cloud
tf workspace select cloud
tf init
tf apply
```

## Get Kubernetes token for token-based authentication
Cluster connectivity can be done via OIDC through Omni, but its a good idea to have secondary access through standard token-based authentication. The cluster is configured for this using [Talos Shared VIP](https://www.talos.dev/v1.9/talos-guides/network/vip/), which makes cluster API access via a shared IP that is advertised by one of the control plane nodes. The address for this is `https://cloud-kube.ucdialplans.com:6443`.

The service account is configured via [kubeapi-serviceaccount.yaml](/manifests/argocd/kubeapi-serviceaccount.yaml) and gets its token when the service account is created.

For configuring your `kubeconfig` file with the token-based authentication information, you need the service account token as well as the certificate authority public certificate. To get these, run:
```
echo
echo "Service Account Token:"
kubectl -n kube-system get secret kubeapi-service-account-secret -o jsonpath="{.data.token}" | base64 -d; echo
echo
echo "Certificate Authority:"
kubectl -n kube-system get secret kubeapi-service-account-secret -o jsonpath="{.data.ca\.crt}"
```

Add these to your `.kube/config` file like this:
```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: 
    *********************************** ADD-CERT-AUTH-BASE64-STRING-HERE ***********************************
    server: https://cloud-kube.ucdialplans.com:6443
  name: cloud
- cluster:
    server: https://omni.ucdialplans.com:8100/
  name: omni-cloud
contexts:
- context:
    cluster: cloud
    user: cloud-user
  name: cloud
- context:
    cluster: omni-cloud
    user: onprem-omni-cloud-ken.lasko@gmail.com
  name: omni-cloud
current-context: cloud
kind: Config
preferences: {}
users:
- name: cloud-user
  user:
    token: *********************************** ADD-SERVICE-ACCOUNT-TOKEN-HERE ***********************************
- name: onprem-omni-cloud-ken.lasko@gmail.com
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1beta1
      args:
      - oidc-login
      - get-token
      - --oidc-issuer-url=https://omni.ucdialplans.com/oidc
      - --oidc-client-id=native
      - --browser-command=wslview
      - --oidc-extra-scope=cluster:cloud
      command: kubectl
      env: null
      interactiveMode: IfAvailable
      provideClusterInfo: false
```

Switch context between the OIDC-based authenticated access and the token-based access with:
```
# To use OIDC
kubectl config use-context omni-cloud

# To use the token
kubectl config use-context cloud
```

# Commit Pre-Check
This repository makes use of [pre-commit](https://pre-commit.com) to guard against accidental secret commits. When you attempt a commit, Pre-Commit will check for secrets and block the commit if one is found. It is currently using [GitGuardian](https://dashboard.gitguardian.com) [ggshield](https://www.gitguardian.com/ggshield) for secret validation. Requires a GitGuardian account, which does offer a free tier for home use.

## Requirements
Requires installation of the following programs:
* ggshield
* pre-commit

In my case, this is handled by [NixOS](https://github.com/kenlasko/nixos)

## Configuration
1. Create a file called `.pre-commit-config.yaml` and place in the root of your repository
2. Populate the file according to your desired platform (ggshield shown):
```
repos:
  - repo: https://github.com/GitGuardian/ggshield
    rev: v1.37.0
    hooks:
      - id: ggshield
```
3. Run the following command:
```
pre-commit install
```
4. For ggshield, login to your GitGuardian account. Only required once.
```
ggshield auth login
```