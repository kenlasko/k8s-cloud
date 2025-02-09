# Introduction
This folder contains the Argo CD application definitions for all the Kubernetes workloads.

The [00-disabled](/argocd-apps/00-disabled) folder is used to put applications that I don"t want to use anymore, but might want to in the future.

Most of the Helm chart managed applications are set to auto-update to newer versions by way of setting `spec.sources.targetRevision: "*"`. A few (like Argo CD), are set to only upgrade minor revisions. Manually managed apps (without Helm charts) are updated via [Keel](/keel). The exceptions include:
* [Cilium](/manifests/cilium)
* [MariaDB](/manifests/mariadb)

## Sync Wave -5
Apps that basically everything else depends on:
* [Cert Manager](/manifests/cert-manager)
* [Cilium](/manifests/cilium)
* [Kubelet Serving Cert Approver](https://github.com/alex1989hu/kubelet-serving-cert-approver)
* [Sealed Secrets](/manifests/sealed-secrets)

## Sync Wave 1
* [Cloudflare Tunnel](/manifests/cloudflare-tunnel)
* [Longhorn](/manifests/longhorn)
* [MariaDB](/manifests/mariadb)
* [PHPMyAdmin](/manifests/phpmyadmin)
* [Registry](/manifests/registry)

## Sync Wave 2
* [AdGuard Home](/manifests/adguard)
* [External DNS](/manifests/external-dns)
* [Smarter Device Manager](/manifests/smarter-device-manager)
* [UCDialplans](/manifests/ucdialplans)
* [VaultWarden](/vaultwarden)

## Sync Wave 10
* [Descheduler](/manifests/descheduler)
* [Headlamp](/headlamp)
* [MariaDB Standalone](/manifests/mariadb)
* [Metrics Server](/manifests/metrics-server)
* [Uptime Kuma](/manifests/uptime-kuma)

## Sync Wave 15
* [Alert Manager/Grafana/Prometheus/Loki](/manifests/promstack)
