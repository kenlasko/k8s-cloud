# Introduction
This folder contains the Argo CD application definitions for all the Kubernetes workloads.

The [00-disabled](/argocd-apps/00-disabled) folder is used to put applications that I don"t want to use anymore, but might want to in the future.

Most of the Helm chart managed applications are set to auto-update to newer versions by way of setting `spec.sources.targetRevision: "*"`. A few (like Argo CD), are set to only upgrade minor revisions. Manually managed apps (without Helm charts) are updated via [Keel](/keel). The exceptions include:
* [Cilium](/cilium)
* [MariaDB](/mariadb)

## Sync Wave -5
Apps that basically everything else depends on:
* [Cert Manager](/cert-manager)
* [Cilium](/cilium)
* [Kubelet Serving Cert Approver](https://github.com/alex1989hu/kubelet-serving-cert-approver)
* [NFS Provisioner](/nfs-provisioner)
* [Sealed Secrets](/sealed-secrets)

## Sync Wave 1
* [Cloudflare Tunnel](/cloudflare-tunnel)
* [Longhorn](/longhorn)
* [MariaDB](/mariadb)
* [PHPMyAdmin](/phpmyadmin)
* [Registry](/registry)

## Sync Wave 2
* [AdGuard Home](/adguard)
* [External DNS](/external-dns)
* [Home Assistant](/home-automation/homeassist)
* [Smarter Device Manager](/smarter-device-manager)
* [UCDialplans](/ucdialplans)
* [UPS Monitor](/home-automation/ups-monitor)
* [VaultWarden](/vaultwarden)
* [ZWave Admin](/home-automation/zwaveadmin)

## Sync Wave 10
* [Descheduler](/descheduler)
* [ESPHome](/home-automation/esphome)
* [Gitea](/gitea)
* [Headlamp](/headlamp)
* [Keel](/keel)
* [MariaDB Standalone](/mariadb-standalone)
* [Metrics Server](/metrics-server)
* [Portainer](/portainer)
* [Uptime Kuma](/uptime-kuma)

## Sync Wave 15
* [Alert Manager/Grafana/Prometheus/Loki](/promstack)

## Sync Wave 99
* [Media Tools](/media-tools)