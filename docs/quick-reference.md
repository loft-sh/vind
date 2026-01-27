# vind Quick Reference

Quick reference guide for common vind operations.

## Installation

```bash
# Upgrade vCluster CLI
vcluster upgrade --version v0.31.0

# Set Docker driver
vcluster use driver docker
```

## Basic Commands

```bash
# Create cluster (automatically connects)
vcluster create my-cluster

# List clusters
vcluster list

# Connect to cluster
vcluster connect my-cluster

# Disconnect from cluster
vcluster disconnect my-cluster

# Pause cluster
vcluster pause my-cluster

# Resume cluster
vcluster resume my-cluster

# Delete cluster
vcluster delete my-cluster

# Describe cluster
vcluster describe my-cluster
```

## Platform UI

```bash
# Start platform
vcluster platform start --version v4.7.0-alpha.0

# Stop platform
vcluster platform stop
```

## Configuration

### Basic Config

```yaml
experimental:
  docker:
    # Load balancer and registry proxy are enabled by default
```

### Multi-Node

```yaml
experimental:
  docker:
    nodes:
      - name: worker-1
      - name: worker-2
```

### Custom Ports

```yaml
experimental:
  docker:
    ports:
      - "8080:80"
      - "8443:443"
```

## Common kubectl Commands

```bash
# Get nodes
kubectl get nodes

# Get pods
kubectl get pods --all-namespaces

# Get services
kubectl get svc

# Describe resource
kubectl describe pod <pod-name>

# View logs
kubectl logs <pod-name>

# Execute command
kubectl exec -it <pod-name> -- /bin/bash
```

## LoadBalancer Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: LoadBalancer
  ports:
    - port: 80
  selector:
    app: my-app
```

## Troubleshooting

```bash
# Check cluster status
vcluster list

# View control plane logs
docker exec vcluster.cp.my-cluster journalctl -u vcluster --nopager

# View node logs (for node named worker-1)
docker exec vcluster.node.my-cluster.worker-1 journalctl -u kubelet --nopager
# or for containerd
docker exec vcluster.node.my-cluster.worker-1 journalctl -u containerd --nopager

# Check Docker resources
docker stats

# Debug mode
vcluster create my-cluster --debug
```

## Useful Links

- [Full Documentation](./getting-started.md)
- [Configuration Guide](./configuration.md)
- [Advanced Features](./advanced-features.md)
- [Troubleshooting](./troubleshooting.md)
- [Examples](../examples/)
