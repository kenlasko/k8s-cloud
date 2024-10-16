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
