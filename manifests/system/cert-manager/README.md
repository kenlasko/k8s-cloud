# Introduction
[Cert-Manager](https://github.com/cert-manager/cert-manager) is used to generate certificates for web services provided by [Cilium Gateway API](/manifests/network/cilium)

## Web Services Certificates
Cert-Manager creates LetsEncrypt certificates for web services provided by the Cilium Gateway API. Generally, there are two certificates widely in use:
* A `*.ucdialplans.com` wildcard certificate for all web services
* A `*.dns.ucdialplans.com` wildcard certificate for [AdGuard Home](/manifests/apps/adguard) private DNS

The `ucdialplans.com` domain is hosted on [Cloudflare](https://dash.cloudflare.com/login) and utilizes the [Cloudflare DNS01 challenge provider](https://cert-manager.io/docs/configuration/acme/dns01/cloudflare/) for proving ownership and approving certificates.

