# Getting Started with vind

This guide will walk you through setting up and using vind (vCluster in Docker) for the first time.

## Quickstart

### Create Your First Cluster

The simplest way to create a cluster:

```bash
vcluster create my-first-cluster
```

This will:
- Create a Docker container with vCluster standalone
- Install Kubernetes inside the container
- Set up networking and storage
- Generate a kubeconfig and connect automatically

### Verify the Cluster

After creation, verify it's working:

```bash
kubectl get nodes
kubectl get namespaces
kubectl get pods --all-namespaces
```

You should see:
- A control plane node
- Default namespaces (default, kube-system, etc.)
- System pods running

## Prerequisites

Before you begin, ensure you have:

1. **Docker** installed and running
   ```bash
   docker --version
   docker ps  # Should work without errors
   ```

2. **vCluster CLI** v0.31.0 or later
   ```bash
   vcluster version
   ```

3. **kubectl** (optional but recommended)
   ```bash
   kubectl version --client
   ```

## Installation

### Step 1: Install/Upgrade vCluster CLI

If you don't have vCluster CLI installed:

```bash
# macOS
brew install loft-sh/tap/vcluster

# Linux
curl -L -o vcluster "https://github.com/loft-sh/vcluster/releases/latest/download/vcluster-linux-amd64" && \
sudo install -c -m 0755 vcluster /usr/local/bin && \
rm -f vcluster

# Windows
# Download from https://github.com/loft-sh/vcluster/releases
```

Upgrade to the required version:

```bash
vcluster upgrade --version v0.31.0
```

### Step 2: Set Docker as Default Driver

```bash
vcluster use driver docker
```

This sets Docker as your default driver for all vCluster operations.

## Optional: Start vCluster Platform UI

For a better management experience, start the vCluster Platform:

```bash
vcluster platform start --version v4.7.0-alpha.0
```

This provides:
- Web-based UI for cluster management
- Visual cluster overview
- Resource monitoring
- Easy cluster operations

Access the UI at `https://localhost:10443` (default port).

## Basic Operations

### List Clusters

```bash
vcluster list
```

### Connect to a Cluster

```bash
vcluster connect my-first-cluster
```

This updates your kubeconfig to use the cluster.

### Disconnect from a Cluster

```bash
vcluster disconnect my-first-cluster
```

This removes the cluster from your kubeconfig.

### Pause a Cluster

Save resources by pausing a cluster:

```bash
vcluster pause my-first-cluster
```

The cluster state is preserved, but resources are freed.

### Resume a Cluster

Resume a paused cluster:

```bash
vcluster resume my-first-cluster
```

The cluster returns to its previous state.

### Delete a Cluster

```bash
vcluster delete my-first-cluster
```

⚠️ **Warning**: This permanently deletes the cluster and all its data.

## Next Steps

Now that you have a basic cluster running:

1. **[Configure your cluster](./configuration.md)** - Learn about advanced configuration options
2. **[Explore advanced features](./advanced-features.md)** - Sleep/wake, load balancers, external nodes
3. **[Check out examples](../examples/)** - See real-world use cases
4. **[Compare with KinD](./vind-vs-kind.md)** - Understand the differences

## Troubleshooting

### Cluster won't start

1. Check Docker is running: `docker ps`
2. Check available resources: `docker system df`
3. View cluster logs: `docker exec vcluster.cp.my-first-cluster journalctl -u vcluster --nopager`

### Can't connect to cluster

1. Ensure cluster is running: `vcluster list`
2. Reconnect: `vcluster connect my-first-cluster --update-current`
3. Check kubeconfig: `kubectl config get-contexts`

### Port conflicts

If you get port conflicts, specify custom ports:

```bash
vcluster create my-cluster \
  --set experimental.docker.ports[0]="8080:80" \
  --set experimental.docker.ports[1]="8443:443"
```

For more troubleshooting help, see the [Troubleshooting Guide](./troubleshooting.md).

## What's Next?

Congratulations! You've created your first vind cluster. Explore the documentation to learn more about:

- [Configuration options](./configuration.md)
- [Advanced features](./advanced-features.md)
- [Real-world examples](../examples/)

Happy clustering! 🚀
