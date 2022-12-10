# MollySocket for Kubernetes

## Helm chart for deploying MollySocket in Kubernetes cluster

Prepare your configuration of the MollySocket helm release by creating your values file e.g. `my_values.yml`
for all options see default values.yaml from the chart: [/charts/mollysocket/values.yaml](/charts/mollysocket/values.yaml)

```yaml

image:
  ## if the image under same tag is updated
  pullPolicy: Always
  ## There is not yet an version tagged container image (just latest):
  ## https://github.com/MollySocket/mollysocket/pkgs/container/mollysocket/versions?filters%5Bversion_type%5D=tagged
  tag: "latest"

mollysocket:
  log: "warn"

## here is the sqlite file stored
persistence:
  enabled: true

## you should not change, till you have support for
## a. your cluster support accessMode=ReadWriteMany  or b. MollySocket support external databases
replicaCount: 1


ingress:
  enabled: true
  hosts:
    - host: chart-example.local
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls:
    - secretName: chart-local-cert
      hosts:
        - chart-example.local
```

Installation with following commands:

```bash
helm repo add mollysocket-repo https://mollysocket.github.io/helm/
helm install --create-namespace --namespace mollysocket my-mollysocket mollysocket-repo/mollysocket -f my_values.yml
```

## Testing

```bash
curl "https://chart-example.local/"
```

result:
  
```json
{"mollysocket":{"version":"0.1.0"}}
```

### Monitoring

Works well with [Prometheus Operator](https://prometheus-operator.dev/) ([Helmchart](https://artifacthub.io/packages/helm/prometheus-community/kube-prometheus-stack)) by enabling following values:

```yaml
prometheus:
  servicemonitor:
    enabled: true
    labels:
      release: "kube-prometheus-stack"
  rules:
    enabled: true # will deploy alert rules
    additionalLabels:
      release: "kube-prometheus-stack"
```

PS: The labels `release=kube-prometheus-stack` is setup with the helmchart of the Prometheus Operator.

PSS: Not yet supported in regulary mollysocket image, see:
https://github.com/MollySocket/mollysocket/issues/3


## TODO
- Working with [Prometheus Operator](https://prometheus-operator.dev/) ([Helmchart](https://artifacthub.io/packages/helm/prometheus-community/kube-prometheus-stack))
  - Waiting for Metric Support of MollySocket: https://github.com/MollySocket/mollysocket/issues/3
  - [x] ServiceMonitor to scrape metrics
  - [x] default set of PrometheusRules for Alerting on unespected behavour
  - [ ] deploy [Grafana](https://grafana.com/grafana/)-Dashboards

- Working with [Banzai Logging Operator](https://banzaicloud.com/docs/one-eye/logging-operator/)
  - Waiting for Serizable-Log Support for MollySocket: https://github.com/MollySocket/mollysocket/issues/5
  - [ ] Flow with Serilizer
  - [ ] optional Output (if not ClusterOutput)