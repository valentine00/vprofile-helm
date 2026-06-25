# vprofile-helm

A GitOps Kubernetes deployment for the vprofile application.

## Structure

- [argocd/](argocd/) - Argo CD application and project manifests
- [helm/vprofile/](helm/vprofile/) - Helm charts for the vprofile application as well as values.yaml file to control various variables
- [kubedefs/](kubedefs/) - plain Kubernetes manifests for manual apply

## Helm chart

The Helm chart is located in `helm/vprofile` and includes Deployment and Service templates plus a ServiceMonitor for Prometheus Operator.

### Key templates

- Service ports are defined in `helm/vprofile/templates/services.yaml` and include a named metrics port `vproapp-metrics`, to allow metrics to be gathered from the exporter.
- The ServiceMonitor template is `helm/vprofile/templates/servicemonitor.yaml` and is controlled by chart values.

### Values (ServiceMonitor)

You can control the ServiceMonitor from `helm/vprofile/values.yaml` using the `serviceMonitor` block. Example:

```yaml
serviceMonitor:
	enabled: true        # set false to disable creating a ServiceMonitor
	interval: 30s
	path: /metrics
```

### Render and verify

Render the entire chart:

```bash
helm template ./helm/vprofile
```

Render only the ServiceMonitor to verify templating:

```bash
helm template ./helm/vprofile -s templates/servicemonitor.yaml
```

Ensure the rendered ServiceMonitor references the named port `vproapp-metrics` and correct namespace.

### Install or upgrade

```bash
helm upgrade --install vprofile ./helm/vprofile --namespace <namespace> --create-namespace
```

## Argo CD

To manage the chart with Argo CD, reference the chart path or package in your Argo CD Application manifest (see `argocd/apps/vprofile-app.yaml`). Argo CD will sync the Helm chart and manage upgrades.

## Manual apply

Use plain manifests if you prefer not to use Helm:

```bash
kubectl apply -f kubedefs/
```

## Notes

- The application exposes HTTP on the port configured at `.Values.app.servicePort` and metrics on `.Values.app.metricsPort`.
- The ServiceMonitor (when enabled) scrapes the named port `vproapp-metrics` on the app service.

