# vind Examples

This directory contains practical examples for using vind (vCluster in Docker).

## Examples

### [basic-cluster.yaml](./basic-cluster.yaml)
Minimal configuration to get started with vind. Load balancer and registry proxy are enabled by default.

**Usage:**
```bash
vcluster create my-cluster -f basic-cluster.yaml
```

### [multi-node-cluster.yaml](./multi-node-cluster.yaml)
Create a cluster with multiple worker nodes for testing distributed workloads.

**Usage:**
```bash
vcluster create my-multi-node-cluster -f multi-node-cluster.yaml
kubectl get nodes  # Should show control-plane + 3 workers
```

### [custom-cni.yaml](./custom-cni.yaml)
Configure a custom CNI plugin (like Calico) for advanced networking.

**Usage:**
```bash
vcluster create my-cni-cluster -f custom-cni.yaml
```

### [loadbalancer-service.yaml](./loadbalancer-service.yaml)
Example LoadBalancer service that works out of the box with vind.

**Usage:**
```bash
# Create cluster (load balancer is enabled by default)
vcluster create my-cluster

# Apply
kubectl apply -f loadbalancer-service.yaml

# Check the service - it will have an EXTERNAL-IP!
kubectl get svc nginx-loadbalancer
```

### [multi-node.yaml](./multi-node.yaml)
Minimal 3-worker-node cluster configuration.

**Usage:**
```bash
vcluster create my-cluster -f multi-node.yaml
kubectl get nodes
```

### [affinity.yaml](./affinity.yaml)
Node affinity example — schedule a workload only on worker nodes (no control-plane).

**Usage:**
```bash
# With a multi-node cluster running
kubectl apply -f affinity.yaml
kubectl get pods -o wide
```

### [antiaffinity.yaml](./antiaffinity.yaml)
Topology spread constraints — spread pods evenly across nodes using `topologySpreadConstraints`.

**Usage:**
```bash
# With a multi-node cluster running
kubectl apply -f antiaffinity.yaml
kubectl get pods -o wide
```

### [snapshot-restore.md](./snapshot-restore.md)
End-to-end guide for snapshot and restore — local files, OCI registries, S3, multi-node clusters, and cloning environments under a new name.

**Highlights:**
- Stateful stack backup and restore (PVCs survive)
- Share environments via OCI registry (`oci://ghcr.io/...`)
- Clone a cluster with a different name using `vcluster create --restore`
- S3/MinIO example with full URL params
- Multi-node snapshot walkthrough

### [external-node.md](./external-node.md)
Guide for joining external nodes (like EC2 instances) to your local vind cluster.

**Usage:**
Follow the step-by-step guide in the file.

For a working example with a GCP node, see: [Replacing KinD with vind - Deep Dive](https://www.vcluster.com/blog/replacing-kind-with-vind-deep-dive)

### GitHub Actions with setup-vind

Use the official [setup-vind GitHub Action](https://github.com/loft-sh/setup-vind) to run vind clusters in your CI/CD pipelines:

```yaml
name: Test with vind
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: loft-sh/setup-vind@main
      - run: |
          vcluster create test-cluster
          kubectl get nodes
```

## Contributing Examples

Have a useful example? Submit a pull request!

Examples should:
- Be well-documented
- Include usage instructions
- Be practical and real-world
- Follow best practices

## More Examples

For more examples and use cases, check out:
- [vCluster Documentation](https://www.vcluster.com/docs)
- [vCluster Examples](https://github.com/loft-sh/vcluster/tree/main/examples)
