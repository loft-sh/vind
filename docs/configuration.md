# vind Configuration Guide

vind supports extensive configuration options to customize your Kubernetes clusters. This guide covers all available configuration parameters.

## Configuration Methods

You can configure vind in several ways:

1. **Command-line flags** - Quick options
2. **Values file** - YAML configuration file
3. **Helm values** - Using `--set` flags

## Basic Configuration

### Using a Values File

Create a `vcluster.yaml` file:

```yaml
experimental:
  docker:
    # Network configuration
    network: "my-custom-network"
    
    # Container image
    image: "ghcr.io/loft-sh/vm-container"
    
    # Load balancer and registry proxy are enabled by default
    # No need to explicitly enable them
```

Then create the cluster:

```bash
vcluster create my-cluster -f vcluster.yaml
```

## Configuration Options

### Network Configuration

```yaml
experimental:
  docker:
    # Use a custom Docker network
    # If not specified, a network will be created automatically
    network: "my-custom-network"
```

**Example:**
```bash
vcluster create my-cluster \
  --set experimental.docker.network=my-network
```

### Additional Nodes

Add worker nodes to your cluster:

```yaml
experimental:
  docker:
    nodes:
      - name: worker-1
        env:
          - "NODE_LABEL=worker"
        ports:
          - "30080:80"
        volumes:
          - "/host/data:/container/data"
        args:
          - "--cap-add=SYS_ADMIN"
      
      - name: worker-2
        # ... more configuration
```

**Example:**
```bash
vcluster create my-cluster \
  --set experimental.docker.nodes[0].name=worker-1
```

### Port Mappings

Expose additional ports:

```yaml
experimental:
  docker:
    ports:
      - "8080:80"      # Host:Container
      - "8443:443"
      - "30000-30100:30000-30100"  # Port ranges
```

**Example:**
```bash
vcluster create my-cluster \
  --set experimental.docker.ports[0]="8080:80" \
  --set experimental.docker.ports[1]="8443:443"
```

### Volume Mounts

Mount host directories or volumes:

```yaml
experimental:
  docker:
    volumes:
      - "/host/path:/container/path:ro"  # Read-only
      - "/host/data:/container/data:rw"  # Read-write
      - "volume-name:/container/path"     # Named volume
```

**Example:**
```bash
vcluster create my-cluster \
  --set experimental.docker.volumes[0]="/host/data:/container/data"
```

### Environment Variables

Set environment variables:

```yaml
experimental:
  docker:
    env:
      - "KEY1=value1"
      - "KEY2=value2"
      - "NODE_NAME=my-node"
```

**Example:**
```bash
vcluster create my-cluster \
  --set experimental.docker.env[0]="CUSTOM_VAR=value"
```

### Extra Docker Arguments

Pass additional arguments to `docker run`:

```yaml
experimental:
  docker:
    args:
      - "--cap-add=SYS_ADMIN"
      - "--cap-add=NET_ADMIN"
      - "--device=/dev/fuse"
      - "--security-opt=apparmor=unconfined"
```

**Example:**
```bash
vcluster create my-cluster \
  --set experimental.docker.args[0]="--cap-add=SYS_ADMIN"
```

## Load Balancer Configuration

Load balancer is enabled by default. LoadBalancer services work out of the box with automatic IP assignment.

**Benefits:**
- LoadBalancer services work out of the box
- No need for MetalLB or other load balancer solutions
- Automatic IP assignment

**Note:** On macOS with Docker Desktop, privileged port binding may require sudo access.

## Registry Proxy Configuration

Registry proxy is enabled by default. It provides pull-through caching via local Docker daemon.

**Benefits:**
- Faster image pulls
- Reduced bandwidth usage
- Use purely local images

**Requirements:**
- Docker must use containerd image storage
- See: https://docs.docker.com/engine/storage/containerd
```

## Node-Specific Configuration

Configure individual nodes:

```yaml
experimental:
  docker:
    nodes:
      - name: worker-1
        # Node-specific ports
        ports:
          - "30080:80"
        
        # Node-specific volumes
        volumes:
          - "/host/data:/data"
        
        # Node-specific environment variables
        # Note: These are Docker container env vars, not Kubernetes node labels
        env:
          - "CUSTOM_VAR=value"
        
        # Node-specific Docker args
        args:
          - "--cap-add=SYS_ADMIN"
```

## Complete Example

Here's a complete configuration example:

```yaml
experimental:
  docker:
    # Network
    network: "vind-network"
    
    # Container image
    image: "ghcr.io/loft-sh/vm-container"
    
    # Port mappings
    ports:
      - "8080:80"
      - "8443:443"
      - "30000-30100:30000-30100"
    
    # Volume mounts
    volumes:
      - "/host/data:/container/data"
      - "vind-storage:/var/lib/data"
    
    # Environment variables
    env:
      - "CLUSTER_NAME=my-cluster"
      - "ENVIRONMENT=development"
    
    # Extra Docker args
    args:
      - "--cap-add=SYS_ADMIN"
    
    # Load balancer and registry proxy are enabled by default
    
    # Additional nodes
    nodes:
      - name: worker-1
        env:
          - "NODE_LABEL=worker"
        ports:
          - "30080:80"
      
      - name: worker-2
        env:
          - "NODE_LABEL=worker"
        ports:
          - "30081:80"
```

Create the cluster:

```bash
vcluster create my-cluster -f vcluster.yaml
```

## Kubernetes Version

Specify the Kubernetes version (defaults to v1.34.0):

```yaml
controlPlane:
  distro:
    k8s:
      version: "v1.34.0"  # or any supported Kubernetes version
```

Or use a custom image tag:

```yaml
controlPlane:
  distro:
    k8s:
      image:
        tag: "v1.34.0"
```

**Note:** The default Kubernetes version is v1.34.0. You can specify any supported version.

## Advanced Configuration

### Custom CNI

vind natively supports Flannel CNI. To use other CNI plugins (Calico, Cilium, etc.):

1. **Disable Flannel** in your configuration:
```yaml
deploy:
  cni:
    flannel:
      enabled: false
```

2. **Create the cluster** with Flannel disabled

3. **Manually install** your preferred CNI plugin after cluster creation

For example, to use Calico:
```bash
# After creating cluster and connecting
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/custom-resources.yaml
```

### Custom CSI

Configure Container Storage Interface:

```yaml
storage:
  persistence: true
  size: "20Gi"
  storageClassName: "local-path"
```

## Configuration Reference

For the complete configuration reference, see the [vCluster configuration documentation](https://www.vcluster.com/docs/vcluster/config).

## Best Practices

1. **Use values files** for complex configurations
2. **Version control** your configuration files
3. **Test configurations** in a development cluster first
4. **Document** custom configurations for your team
5. **Use environment variables** for sensitive data

## Troubleshooting Configuration

### Invalid Configuration

If you get configuration errors:

1. Validate your YAML syntax
2. Check the configuration reference
3. Use `--debug` flag for detailed errors:
   ```bash
   vcluster create my-cluster -f config.yaml --debug
   ```

### Configuration Not Applied

If changes aren't applied:

1. Delete and recreate the cluster
2. Check for typos in configuration keys
3. Verify the configuration format

For more help, see the [Troubleshooting Guide](./troubleshooting.md).
