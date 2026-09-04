# Wishlist Helm Chart (fork)

Original chart credit: [Deff](https://github.com/mddeff)/[wishlist-charts](https://github.com/mddeff/wishlist-charts)

> [!IMPORTANT]
> This whole thing is for my own use with Flux and Kustomize, I will try to keep it up-to-date, but I am not sure it will work with a different setup. The README is unedited besides this note and the title.  
> \- Love, L
> - Update Sept 2026: Decided to add an example config for flux and Kustomize under [Using this chart](#using-this-chart)

This helm chart installs the [Wishlist web application](https://github.com/cmintey/wishlist) (under [MIT license](https://github.com/cmintey/wishlist?tab=MIT-1-ov-file)) written by [cmintey](https://github.com/cmintey/).  This is an **unofficial chart** utilizing the official docker image. 

Please submit any *application* or container image issues with the upstream repository.  

Please do not submit any issues with the helm chart to the upstream repo as it is unofficial and unsupported.


## Using this chart

### Helm

```bash
helm repo add wishlist https://mddeff.github.io/wishlist-charts/
helm repo update
# make values-local.yaml locally from overrides of charts/wishlist/values.yaml 
helm upgrade --install my-cool-wishlist-release wishlist/wishlist --namespace wishlist --create-namespace -f values-local.yaml
```

### Flux with Kustomize

> [!NOTE]
> This is a config that works in my setup, mostly throwing it out there as inspo, ymmv, feel free to open issues if you find an improvement/more generic setup ˆˆ

helm.yaml
```yaml
---
apiVersion: source.toolkit.fluxcd.io/v1
kind: HelmRepository
metadata:
  name: wishlist
spec:
  interval: 1h
  url: https://fox1e-dev.github.io/wishlist-charts/ # https://github.com/fox1e-dev/wishlist-charts
---
apiVersion: source.toolkit.fluxcd.io/v1
kind: HelmChart
metadata:
  name: wishlist
spec:
  chart: wishlist
  interval: 10m0s
  sourceRef:
    kind: HelmRepository
    name: wishlist
  version: '0.0.6'
---
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: wishlist
spec:
  chartRef:
    kind: HelmChart
    name: wishlist
  install:
    createNamespace: true
  interval: 10m0s
  storageNamespace: servicemaster # you will probably want to change this to your apps/services namespace
  targetNamespace: servicemaster # you will probably want to change this to your apps/services namespace
  valuesFrom:
    - kind: ConfigMap
      name: wishlist-helm-values
```

kustomization.yaml
```yaml
---
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
configMapGenerator:
  - name: wishlist-helm-values
    files:
      - values.yaml
    options:
      disableNameSuffixHash: true
      labels:
        reconcile.fluxcd.io/watch: Enabled
namespace: servicemaster # you will probably want to change this to your apps/services namespace
resources:
  - helm.yaml
```

values.yaml for traefik (see also #example-wishlist-overrides from the original readme)
```yaml
ingress:
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt
    traefik.ingress.kubernetes.io/router.entrypoints: websecure # set this according to your traefik config
  class: traefik
  enabled: true
  host: your.domain # set this to your domain
  tls:
    - hosts:
      - your.domain # set this to your domain
      secretName: wishlist-tls
```

## example wishlist overrides
Defaults are an attempt at sanity. Only thing you will want to override is your ingress info.  Heres an example for using traefik:

```yaml
ingress:
  enabled: true
  # labels: {}
  annotations:
    traefik.ingress.kubernetes.io/router.entrypoints: websecure
    traefik.ingress.kubernetes.io/router.tls.certresolver: internalacme
  host: "wishlist.example.com"
  path: "/"
  tls:
    - hosts:
      - wishlist.example.com
      # secretName: 
```

> Note that in the example above, `websecure` and `internalacme` are dependent on my traefik config. YMMV. [RTFM](https://doc.traefik.io/traefik/). All that good stuff. 

## Warranty
TL;DR - Caveat Emptor. There is no warranty express or implied of fitness for a particular purpose.
