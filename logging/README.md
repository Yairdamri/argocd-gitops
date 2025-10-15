# EFK Stack for Kubernetes

This directory contains ArgoCD Application manifests for deploying the Elasticsearch, Fluentd, and Kibana (EFK) stack to your Kubernetes cluster.

## Components

The EFK stack consists of the following components:

1. **Elasticsearch**: Distributed search and analytics engine for storing logs
2. **Fluentd**: Log collector that runs as a DaemonSet on each node to collect logs from all pods
3. **Kibana**: Web UI for visualizing and searching logs

## Deployment

The EFK stack is deployed and managed by ArgoCD. The components are deployed in the `efk` namespace.

### Prerequisites

- Kubernetes cluster with ArgoCD installed
- Storage class available for Elasticsearch persistent volumes
- Ingress controller (like NGINX) for accessing Kibana

### Accessing Kibana

After deployment, Kibana will be available at the following URL:

```
http://kibana.local
```

You may need to add this hostname to your hosts file or configure your ingress/DNS settings accordingly.

## Configuration

### Elasticsearch

Elasticsearch is configured with the following settings:

- Single node deployment (for development/testing)
- 10GB of persistent storage
- Resource limits: 1 CPU core, 2GB memory

To scale for production:
- Increase `replicas` value
- Adjust `minimumMasterNodes` accordingly
- Increase resource allocations

### Fluentd

Fluentd is configured to:

- Run as a DaemonSet on all nodes
- Collect logs from container runtime
- Forward logs to Elasticsearch

### Kibana

Kibana is configured with:

- Connection to Elasticsearch
- NGINX ingress for web access
- Default configuration for log visualization

## Customization

To modify the configuration, edit the respective YAML files:

- `elasticsearch.yaml`: Elasticsearch settings
- `fluentd.yaml`: Log collection configuration
- `kibana.yaml`: UI and access settings

## Troubleshooting

Common issues:

1. **Elasticsearch fails to start**: Check persistent volume availability
2. **Fluentd not collecting logs**: Check permissions and mounted volumes
3. **Kibana can't connect to Elasticsearch**: Verify service names and ports

## Accessing Logs

1. Access Kibana UI
2. Create an index pattern (typically `logstash-*`)
3. Navigate to "Discover" to search and filter logs
4. Create dashboards for visualization

## Adding Custom Log Parsing

To add custom log parsing for specific applications:

1. Create a ConfigMap with Fluentd configuration
2. Add it to the Fluentd application's `configMapConfigs` list
