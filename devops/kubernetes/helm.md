# Helm Charts

Helm is the **package manager for Kubernetes**.

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
