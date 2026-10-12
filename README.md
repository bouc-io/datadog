# Datadog Kubernetes Integration

This repository implements Datadog monitoring and observability for Kubernetes clusters using Helm charts and GitOps practices.

## Architecture

Datadog provides comprehensive monitoring and observability for Kubernetes clusters through its agent deployment. The integration includes:
- Datadog Agent deployment
- Metrics collection
- Log collection
- APM (Application Performance Monitoring)
- Infrastructure monitoring

## Repository Structure

```shell
infrastructure/datadog/
├── lcl.values.yaml    # Local development environment configuration
├── snbx.values.yaml  # Sandbox environment configuration
└── README.md
```

## Installation

### Prerequisites
- Kubernetes cluster
- Helm 3.x
- Datadog API key

### Installation Steps

1. Create the Datadog namespace:
```shell
kubectl create namespace datadog --cluster docker-desktop
```

2. Add the Datadog Helm repository:
```shell
helm repo add datadog https://helm.datadoghq.com
helm repo update
```

3. The Datadog API key secret is delivered by External Secrets, not created by hand.

   `datadog-secret` (key `api-key`) in this namespace is produced by the
   `boucio-datadog-secret` ExternalSecret in
   `infrastructure/external-secrets/externalsecrets/infra/<env>/`. The same key is
   fanned out to the `opentelemetry` namespace as well, because Secrets are
   namespace-scoped. Set the source value once:

   - **local:** add `boucio-datadog-api-key` to `boucio-local-external-credentials.yaml`
     (gitignored; copy from the `.example.yaml`) and apply it.
   - **sandbox:** `gcloud secrets create boucio-datadog-api-key`.

   See `infrastructure/external-secrets/README.md`. For a standalone `helm install`
   outside this GitOps setup, create it manually:
```shell
kubectl create secret generic datadog-secret -n datadog --from-literal api-key=<API KEY>
```

4. Install Datadog using Helm:

For the local cluster:
```shell
helm upgrade -install datadog-agent -f base.values.yaml -f lcl.values.yaml datadog/datadog -n datadog --kube-context docker-desktop
```

Or for the sandboxy cluster:
```shell
helm upgrade -install datadog-agent -f base.values.yaml -f snbx.values.yaml datadog/datadog -n datadog --kube-context $BOUCIO_CLSTR_FULLNAME
```


## Configuration

### Environment-Specific Configuration

- **Local Environment**: Use `lcl.values.yaml` for local development
- **Sandbox Environment**: Use `snbx.values.yaml` for sandbox environment

### Key Configuration Parameters

- `datadog.apiKey`: Your Datadog API key
- `datadog.site`: Datadog site (e.g., datadoghq.com)
- `datadog.logs.enabled`: Enable log collection
- `datadog.apm.enabled`: Enable APM
- `datadog.processAgent.enabled`: Enable process monitoring

## Monitoring Features

### Available Monitoring Components
- Infrastructure metrics
- Container metrics
- Custom metrics
- APM traces
- Log collection
- Network performance monitoring

## Troubleshooting

### Common Issues and Solutions

1. **Agent not reporting data**
   - Verify API key is correct
   - Check agent pod logs
   - Ensure network connectivity to Datadog

2. **High resource usage**
   - Adjust resource limits in values.yaml
   - Review enabled features
   - Check for duplicate metrics

### How to check Datadog Agent status:
```shell
kubectl get pods -n datadog
kubectl logs -n datadog -l app.kubernetes.io/name=datadog
```

## References

1. [Datadog Kubernetes Integration](https://docs.datadoghq.com/agent/kubernetes/?tab=helm)
2. [Datadog Helm Charts](https://github.com/DataDog/helm-charts/tree/master/charts/datadog)
3. [Datadog Agent Configuration](https://docs.datadoghq.com/agent/configuration/)
4. [Datadog Log Collection](https://docs.datadoghq.com/logs/log_collection/)
5. [Datadog APM](https://docs.datadoghq.com/tracing/)

## License

[Elastic License 2.0](./LICENSE) — source-available; not OSI open source.
