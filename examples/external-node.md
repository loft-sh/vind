# Joining External Nodes to vind (Private Nodes)

**Important:** This guide is for **Private Nodes** mode, which is different from Docker experimental nodes. Private Nodes allows you to join real worker nodes (like EC2 instances) to your vCluster control plane using VPN.

## Prerequisites

1. **Private Nodes enabled** in your vCluster
2. **vCluster Platform** running (required for VPN)
3. External node with network access

## Step 1: Start vCluster Platform

```bash
# Start the platform
vcluster platform start
```

## Step 2: Create Your vind Cluster with Private Nodes

```bash
# Create cluster with private nodes and VPN enabled
vcluster create my-cluster \
  --set privateNodes.vpn.enabled=true \
  --set privateNodes.vpn.nodeToNode.enabled=true
```

## Step 3: Get Join Token

```bash
# Get the join token
vcluster token create
```

Save this command.

## Step 4: Install vCluster on External Node

On your external node (e.g., EC2 instance) run the command from the previous step:

```bash
curl ...
```

## Step 5: Verify

Back on your local machine:

```bash
# Check nodes
kubectl get nodes

# You should see your external node listed!
# NAME           STATUS   ROLES    AGE   VERSION
# my-cluster     Ready    master   5m    v1.28.0
# ec2-instance   Ready    <none>   2m    v1.28.0
```
