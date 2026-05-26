# Helm Charts

Helm helps you manage Kubernetes applications. While it is similar to using `kubectl apply -f /path/to/manifests`, Helm offers templating and packaging capabilities to define, install, and upgrade even the most complex Kubernetes applications.

- **Packaging**: Helm packages all manifest files (like `deployment.yaml`, `service.yaml`, `config.yaml`, `secret.yaml`, etc.) into a single bundle called a **Helm Chart**.
- **Templating**: Helm works by templating a single set of deployment manifests to configure them easily for different environments (e.g., dev, staging, prod).

## Key Concepts

- **Chart** — A package of K8s resources
- **Release** — A running instance of a chart
- **Repository** — A collection of charts
- **Values** — Configuration to customize a chart

## Commands

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm search repo nginx
helm install my-release bitnami/nginx
helm list
helm upgrade my-release bitnami/nginx --set replicaCount=3
helm uninstall my-release
```

## Creating Your Own Chart

```bash
helm create my-chart

# Structure
my-chart/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
└── charts/
```
