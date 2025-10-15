# EFK Stack Testing Guide

Follow these steps to test and validate your EFK stack deployment.

## Pre-deployment Validation

Before applying the configurations:

1. **Validate YAML files**:
   ```bash
   kubectl --dry-run=server -n argocd apply -f elasticsearch.yaml
   kubectl --dry-run=server -n argocd apply -f fluentd.yaml
   kubectl --dry-run=server -n argocd apply -f kibana.yaml
   ```

## Post-deployment Testing

After deploying the EFK stack through ArgoCD, verify each component:

### 1. Verify Elasticsearch

```bash
# Check if Elasticsearch pods are running
kubectl get pods -n efk -l app=elasticsearch-master

# Check Elasticsearch service
kubectl get svc -n efk -l app=elasticsearch-master

# Port-forward to test Elasticsearch directly
kubectl port-forward svc/elasticsearch-master -n efk 9200:9200

# In another terminal, test the connection
curl http://localhost:9200
```

Expected output should include Elasticsearch version and cluster information.

### 2. Verify Fluentd

```bash
# Check if Fluentd DaemonSet is running on all nodes
kubectl get pods -n efk -l app=fluentd -o wide

# Check Fluentd logs to ensure it's collecting and forwarding
kubectl logs -n efk $(kubectl get pods -n efk -l app=fluentd -o name | head -n 1)
```

Look for successful connections to Elasticsearch in the logs.

### 3. Verify Kibana

```bash
# Check if Kibana pod is running
kubectl get pods -n efk -l app=kibana

# Port-forward to access Kibana UI locally
kubectl port-forward svc/kibana-kibana -n efk 5601:5601
```

Access Kibana at http://localhost:5601 in your browser.

### 4. Test Log Collection

Generate some test logs to verify the collection:

```bash
# Create a test pod that generates logs
kubectl run log-generator -n default --image=busybox -- /bin/sh -c 'while true; do echo "$(date) - This is a test log entry"; sleep 5; done'

# Wait a few minutes for logs to be collected
```

Then check in Kibana:

1. Access Kibana UI
2. Go to Stack Management > Index Patterns
3. Create index pattern (typically `logstash-*`)
4. Go to Discover and search for "test log entry"

### 5. Test Ingress (if configured)

```bash
# Check if the Kibana ingress is created
kubectl get ingress -n efk

# Test the ingress URL (adjust domain as needed)
curl -H "Host: kibana.local" http://[INGRESS_IP]
```

## Troubleshooting Common Issues

### Elasticsearch won't start:
- Check PVC: `kubectl get pvc -n efk`
- Check events: `kubectl get events -n efk`

### Fluentd can't connect to Elasticsearch:
- Verify service names: `kubectl get svc -n efk`
- Check Fluentd config: `kubectl describe cm -n efk -l app=fluentd`

### Kibana shows no data:
- Verify Elasticsearch connection
- Ensure index patterns are correctly configured
- Check if data is reaching Elasticsearch: `curl http://localhost:9200/_cat/indices`
