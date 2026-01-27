# Advanced Features

vind offers powerful advanced features that set it apart from other local Kubernetes solutions. This guide covers sleep/wake, load balancers, external nodes, and more.

## Sleep and Wake

One of vind's standout features is the ability to pause and resume clusters, saving resources when not in use.

### How It Works

When you pause a cluster:
- The Docker containers are stopped (not deleted)
- All cluster state is preserved
- Resources are freed on your host
- The cluster can be resumed instantly

### Usage

```bash
# Pause a cluster
vcluster pause my-cluster

# Resume a cluster
vcluster resume my-cluster
```

### Benefits

- **Save Resources**: Free up CPU, memory, and disk when clusters aren't in use
- **Instant Resume**: Clusters resume in seconds, not minutes
- **State Preservation**: All data, pods, and configurations are maintained
- **Perfect for Development**: Pause during breaks, resume when needed

### Example Workflow

```bash
# Morning: Start your development cluster
vcluster create dev-cluster
# ... do your work ...

# Lunch break: Pause to save resources
vcluster pause dev-cluster

# After lunch: Resume instantly
vcluster resume dev-cluster
# Everything is exactly as you left it!
```

## Automatic Load Balancers

vind provides automatic LoadBalancer service support out of the box. Load balancer is enabled by default.

### Create a LoadBalancer Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 8080
  selector:
    app: my-app
```

### Access Your Service

Once created, the service gets an IP address automatically:

```bash
kubectl get svc my-service
# NAME          TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)
# my-service    LoadBalancer   10.96.0.1     172.17.0.2    80:30000/TCP
```

You can access it directly via the EXTERNAL-IP!

### How It Works

- vind automatically configures the Docker network
- LoadBalancer IPs are assigned from the Docker network
- On macOS, ports are forwarded to localhost automatically
- No MetalLB or other load balancer setup required!

### Platform Requirements

- **Linux**: Works automatically
- **macOS (Docker Desktop)**: Requires privileged port access (may need sudo)
- **Windows**: Limited support

## Pull-through Cache (Registry Proxy)

Speed up image pulls by using your local Docker daemon's containerd image storage.

### Registry Proxy

Registry proxy is enabled by default.

### What It Does

The registry proxy allows the vCluster to pull images directly from the host Docker daemon's containerd image storage. This means:
- Images already pulled to your local Docker are available to the cluster
- No need to re-pull images that are already cached locally
- Can use purely local images without a registry
- Faster image pulls since they come from local storage

### Requirements

1. Docker must use containerd image storage
   - Check: `docker info | grep "Storage Driver"`
   - Should show: `containerd` or `overlay2` with containerd

2. Enable containerd storage in Docker:
   - See: https://docs.docker.com/engine/storage/containerd/

### How It Works

When enabled, vCluster mounts the containerd socket from the host Docker daemon. When the cluster needs an image:
1. It checks the local Docker daemon's containerd storage
2. If the image exists locally, it uses it directly
3. If not found, it pulls from the registry as normal

**Note:** This only works if Docker is using containerd as the image storage backend.

## External Node Joining (Private Nodes)

**Important:** Joining external nodes requires **Private Nodes** mode, not the Docker experimental nodes. Private Nodes is a different feature that allows joining real worker nodes (like EC2 instances) to your vCluster control plane.

### Use Cases

- **Hybrid Development**: Test with real cloud resources
- **GPU Access**: Use cloud GPUs from your local cluster
- **Cost Optimization**: Use local control plane with cloud workers
- **Testing**: Test multi-cloud scenarios

### Prerequisites

1. **Private Nodes enabled** in your vCluster
2. **vCluster Platform** running (required for VPN)
3. External node with network access
4. Join token from the cluster

### Configuration

To enable private nodes with VPN support:

```yaml
privateNodes:
  enabled: true
  vpn:
    enabled: true        # Enables node-to-control-plane VPN
    nodeToNode:
      enabled: true      # Enables node-to-node VPN (optional)
```

### Joining a Node

#### Step 1: Create Cluster with Private Nodes

```bash
vcluster create my-cluster \
  --set privateNodes.enabled=true \
  --set privateNodes.vpn.enabled=true
```

#### Step 2: Get Join Token

```bash
kubectl get secret join-token -n default -o jsonpath='{.data.token}' | base64 -d
```

#### Step 3: Install vCluster on External Node

On your external node (e.g., EC2 instance):

```bash
# Install vCluster binary
curl -L -o vcluster "https://github.com/loft-sh/vcluster/releases/latest/download/vcluster-linux-amd64"
chmod +x vcluster
sudo mv vcluster /usr/local/bin/

# Join the cluster (requires platform URL)
vcluster join my-cluster \
  --token <join-token> \
  --platform-url https://your-platform-url
```

#### Step 4: Verify

```bash
kubectl get nodes -o wide
# You should see your external node with a tailscale IP (100.64.x.x)!
```

### VPN Details

vCluster VPN uses Tailscale technology to create secure connections:
- **Node-to-Control-Plane VPN**: Connects nodes to the control plane
- **Node-to-Node VPN**: Connects nodes to each other (optional)
- Works across NAT and complex network setups
- Requires vCluster Platform to be accessible

**Note:** VPN is only available for Private Nodes mode, not for Docker experimental nodes. Docker experimental nodes are local Docker containers, while Private Nodes are real worker nodes that can be anywhere.

## Custom CNI and CSI

vind natively supports Flannel CNI, but you can install other CNI plugins manually.

### CNI Options

- **Flannel**: Default, simple overlay network (built-in)
- **Calico**: Advanced networking and policy (manual install)
- **Cilium**: eBPF-powered networking (manual install)
- **Weave**: Network encryption (manual install)

### CSI Options

- **Local Path Provisioner**: Default, simple storage
- **NFS**: Network file system
- **Rook**: Ceph storage
- **Longhorn**: Distributed block storage

### Configure CNI

vCluster only supports Flannel natively. To use other CNI plugins:

1. **Disable Flannel**:
```yaml
deploy:
  cni:
    flannel:
      enabled: false
```

2. **Create the cluster** and connect to it

3. **Install your preferred CNI** (e.g., Calico):
```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/custom-resources.yaml
```

Or for Cilium:
```bash
helm repo add cilium https://helm.cilium.io/
helm install cilium cilium/cilium --version 1.14.0
```

### Configure CSI

```yaml
storage:
  persistence: true
  storageClassName: "nfs-client"
```

## Multi-Node Clusters

Create clusters with multiple worker nodes:

```yaml
experimental:
  docker:
    nodes:
      - name: worker-1
      - name: worker-2
      - name: worker-3
```

```bash
vcluster create my-cluster -f config.yaml
```

Verify:

```bash
kubectl get nodes
# NAME           STATUS   ROLES    AGE   VERSION
# my-cluster     Ready    master   5m    v1.28.0
# worker-1       Ready    <none>   4m    v1.28.0
# worker-2       Ready    <none>   4m    v1.28.0
# worker-3       Ready    <none>   4m    v1.28.0
```

## Snapshots and Restore

**Coming Soon!** vind will support saving and restoring cluster snapshots.

This will allow you to:
- Save cluster state at any point
- Restore to a previous state
- Create templates from snapshots
- Share cluster configurations

Stay tuned for updates!

## Best Practices

### Sleep/Wake

- Pause clusters during breaks
- Resume before starting work
- Don't pause clusters with active workloads (if needed)

### Load Balancers

- Use LoadBalancer services for easy access
- Check EXTERNAL-IP after service creation
- On macOS, ensure privileged port access

### Registry Proxy

- Enable for faster development cycles
- Pre-pull common images to Docker
- Use for offline development

### External Nodes

- Use VPN for secure connections
- Test network connectivity first
- Monitor resource usage

## Troubleshooting

### Sleep/Wake Issues

```bash
# Check cluster status
vcluster list

# View container status
docker ps -a | grep vcluster

# Check control plane logs
docker exec vcluster.cp.my-cluster journalctl -u vcluster --nopager
```

### Load Balancer Not Working

1. Load balancer is enabled by default
2. Check Docker network connectivity
3. On macOS, ensure sudo access for port forwarding
4. Check service status: `kubectl get svc`

### Registry Proxy Issues

1. Verify containerd storage: `docker info`
2. Check containerd socket access
3. Registry proxy is enabled by default
4. Check logs for errors

### External Node Connection

1. Verify VPN is configured
2. Check network connectivity
3. Verify join token is valid
4. Check platform URL is accessible

For more help, see the [Troubleshooting Guide](./troubleshooting.md).
