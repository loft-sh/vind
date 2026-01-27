# Joining External Nodes to vind (Private Nodes)

**Important:** This guide is for **Private Nodes** mode, which is different from Docker experimental nodes. Private Nodes allows you to join real worker nodes (like EC2 instances) to your vCluster control plane using VPN.

## Prerequisites

1. **Private Nodes enabled** in your vCluster
2. **vCluster Platform** running (required for VPN)
3. External node with network access
4. vCluster binary installed on external node

## Step 1: Start vCluster Platform

```bash
# Start the platform
vcluster platform start --version v4.7.0-alpha.0
```

## Step 2: Create Your vind Cluster with Private Nodes

```bash
# Create cluster with private nodes and VPN enabled
vcluster create my-cluster \
  --set privateNodes.enabled=true \
  --set privateNodes.vpn.enabled=true \
  --set privateNodes.vpn.nodeToNode.enabled=true
```

## Step 3: Get Join Token

```bash
# Get the join token
kubectl get secret join-token -n default -o jsonpath='{.data.token}' | base64 -d
```

Save this token - you'll need it on the external node.

## Step 4: Install vCluster on External Node

On your external node (e.g., EC2 instance):

```bash
# Download vCluster binary
curl -L -o vcluster "https://github.com/loft-sh/vcluster/releases/latest/download/vcluster-linux-amd64"
chmod +x vcluster
sudo mv vcluster /usr/local/bin/

# Verify installation
vcluster version
```

## Step 5: Join the Node

On the external node:

```bash
# Join the cluster
vcluster join my-cluster \
  --token <your-join-token> \
  --platform-url https://your-platform-url:10443
```

Replace:
- `<your-join-token>` with the token from Step 3
- `your-platform-url` with your platform URL

## Step 6: Verify

Back on your local machine:

```bash
# Check nodes
kubectl get nodes

# You should see your external node listed!
# NAME           STATUS   ROLES    AGE   VERSION
# my-cluster     Ready    master   5m    v1.28.0
# ec2-instance   Ready    <none>   2m    v1.28.0
```

## Example: EC2 Instance

### Launch EC2 Instance

```bash
# Launch an EC2 instance (example with AWS CLI)
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t3.medium \
  --key-name my-key \
  --security-group-ids sg-xxxxxxxxx
```

### Configure Security Group

Allow inbound traffic:
- Port 8443 (vCluster API)
- Port 10250 (kubelet)
- Port 10259 (kube-scheduler)
- Port 10257 (kube-controller-manager)

### SSH to Instance

```bash
ssh -i my-key.pem ec2-user@<ec2-ip>
```

### Install and Join

```bash
# Install vCluster
curl -L -o vcluster "https://github.com/loft-sh/vcluster/releases/latest/download/vcluster-linux-amd64"
chmod +x vcluster
sudo mv vcluster /usr/local/bin/

# Join cluster
vcluster join my-cluster \
  --token <token> \
  --platform-url https://your-platform-url:10443
```

## Troubleshooting

### Node Not Appearing

1. Check VPN connection:
   ```bash
   # On external node
   ping <control-plane-ip>
   ```

2. Verify join token is valid:
   ```bash
   # Get new token if needed
   kubectl get secret join-token -n default -o jsonpath='{.data.token}' | base64 -d
   ```

3. Check platform URL is accessible:
   ```bash
   curl https://your-platform-url:10443/health
   ```

### Connection Issues

1. Verify network connectivity:
   ```bash
   # From external node
   telnet <control-plane-ip> 8443
   ```

2. Check firewall rules:
   ```bash
   # Allow required ports
   sudo ufw allow 8443/tcp
   sudo ufw allow 10250/tcp
   ```

3. Verify VPN is enabled:
   ```bash
   vcluster describe my-cluster | grep -i vpn
   ```

## Use Cases

### Hybrid Development
- Local control plane
- Cloud worker nodes
- Test with real cloud resources

### GPU Access
- Local cluster
- Cloud GPU nodes
- Access GPUs without full cloud cluster

### Cost Optimization
- Lightweight local control plane
- On-demand cloud workers
- Pay only for what you use

### Multi-Cloud Testing
- Single control plane
- Nodes in different clouds
- Test multi-cloud scenarios

## Security Considerations

1. **VPN Encryption**: All traffic is encrypted via VPN
2. **Token Security**: Keep join tokens secure
3. **Network Isolation**: Use security groups/firewalls
4. **Access Control**: Limit who can join nodes

## Next Steps

- Configure node labels and taints
- Set up node autoscaling
- Configure storage for external nodes
- Set up monitoring and logging
