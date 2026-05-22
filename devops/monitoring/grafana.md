# Grafana

Analytics and visualization platform.

## Key Features

- **Dashboards** — Rich visualizations
- **Data Sources** — Prometheus, InfluxDB, Elasticsearch, etc.
- **Alerting** — Built-in alert rules
- **Templating** — Dynamic dashboards with variables

## Dashboard as Code

```json
{
  "dashboard": {
    "title": "Application Metrics",
    "panels": [
      {
        "title": "Request Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])",
            "legendFormat": "{{method}} {{status}}"
          }
        ]
      }
    ]
  }
}
```

> **Tip:** Use Grafana provisioning to manage dashboards as code in Git.
