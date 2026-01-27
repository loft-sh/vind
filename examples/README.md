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

### [external-node.md](./external-node.md)
Guide for joining external nodes (like EC2 instances) to your local vind cluster.

**Usage:**
Follow the step-by-step guide in the file.

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
