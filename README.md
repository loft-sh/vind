# vind - vCluster in Docker

<div align="center">

![vind Logo](https://img.shields.io/badge/vind-vCluster%20in%20Docker-orange?style=for-the-badge&logo=docker&logoColor=white)

**Next-Level Kubernetes Development - Run Kubernetes clusters as Docker containers**

[![GitHub stars](https://img.shields.io/github/stars/loft-sh/vind?style=social)](https://github.com/loft-sh/vind/stargazers)
[![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](LICENSE)

**[Quick Start](#-quick-start)** • **[Features](#-key-features)** • **[Documentation](#-documentation)** • **[vs KinD](#-vind-vs-kind)** • **[Examples](./examples/)**

</div>

---

## 🎯 What is vind?

**vind (vCluster in Docker)** is a revolutionary way to run Kubernetes clusters directly as Docker containers. Built on top of [vCluster](https://github.com/loft-sh/vcluster), vind combines the power of virtual Kubernetes clusters with the simplicity of Docker, creating isolated Kubernetes environments that are perfect for development, testing, and CI/CD pipelines.

**Note:** vind uses vCluster's Private Nodes mode internally. This is automatically enabled when using the Docker driver and is required for proper operation. This is expected behavior, not a configuration issue.

### Why vind?

- 🚀 **Faster than KinD** - Optimized container-based architecture
- 💤 **Sleep & Wake** - Pause clusters to save resources, resume instantly
- 🎨 **Built-in UI** - Free vCluster Platform UI for cluster management
- ⚡ **Load Balancers OOB** - Automatic LoadBalancer services without extra setup
- 🐳 **Docker Native** - Leverages Docker's networking and storage
- 🔄 **Pull-through Cache** - Faster image pulls via local Docker daemon
- 🌐 **Hybrid Nodes** - Join external nodes (even cloud instances) via VPN
- 📸 **Snapshots** - Save and restore cluster state (coming soon)

---

## 🚀 Quick Start

### Prerequisites

- Docker installed and running
- vCluster CLI v0.31.0 or later

### Installation

```bash
# Upgrade vCluster CLI to the latest version
vcluster upgrade --version v0.31.0

# Set Docker as the default driver
vcluster use driver docker
```

### Optional: Start vCluster Platform UI

```bash
# Start vCluster Platform (optional but recommended)
vcluster platform start
```

This gives you a beautiful web UI to manage your clusters!

### Create Your First Cluster

```bash
# Create a vCluster in Docker (automatically connects)
vcluster create my-cluster

# Verify it's working
kubectl get nodes
kubectl get namespaces
```

---

## ✨ Key Features

### 🎮 Kubernetes UI via vCluster Platform
Integrated management and visibility with a beautiful web interface - no need for external tools!

### 💤 Sleep and Wakeup
Pause your clusters when not in use and resume them instantly. Perfect for saving resources during development breaks.

```bash
# Pause a cluster
vcluster pause my-cluster

# Resume a cluster
vcluster resume my-cluster
```

### ⚡ Automatic Load Balancers
Hassle-free service exposure with automatic LoadBalancer support - works out of the box! (Enabled by default)

### 🐳 Pull-through Cache via Local Docker Daemon
Faster image pulls and reduced bandwidth by leveraging your local Docker daemon's image cache. (Enabled by default)

### 🌐 Join External Nodes
Connect real cloud instances (like EC2) as nodes to your local cluster - how cool is that!

### 🔧 Flexible CNI and CSI
Choose your own Container Network Interface and Container Storage Interface plugins.

---

## 📖 Documentation

- **[Getting Started Guide](./docs/getting-started.md)** - Detailed setup and first steps
- **[Configuration Guide](./docs/configuration.md)** - All configuration options explained
- **[Advanced Features](./docs/advanced-features.md)** - Sleep/wake, load balancers, external nodes
- **[vind vs KinD](./docs/vind-vs-kind.md)** - Detailed comparison
- **[Examples](./examples/)** - Real-world examples and use cases
- **[Troubleshooting](./docs/troubleshooting.md)** - Common issues and solutions

---

## 🎨 vind vs KinD

| Feature | vind | KinD |
|---------|------|------|
| **UI Platform** | ✅ Built-in vCluster Platform UI | ❌ Command-line only |
| **Sleep/Wake** | ✅ Native pause & resume | ❌ Must delete & recreate |
| **Load Balancers** | ✅ Automatic, works OOB | ❌ Manual setup required |
| **Image Caching** | ✅ Pull-through via Docker daemon | ❌ Direct registry pulls |
| **External Nodes** | ✅ Join cloud instances via VPN | ❌ Local only |
| **CNI/CSI Choice** | ✅ Your choice | ⚠️ Limited |
| **Snapshots** | ✅ Coming soon | ❌ Not available |

👉 **[See detailed comparison](./docs/vind-vs-kind.md)**

---

## 🔧 Configuration

vind supports extensive configuration options. Here's a quick example:

```yaml
experimental:
  docker:
    # Custom network
    network: "my-custom-network"
    
    # Additional nodes (only name required)
    nodes:
      - name: worker-1
        env:
          - "CUSTOM_VAR=value"
    
    # Load balancer and registry proxy are enabled by default
    
    # Custom ports
    ports:
      - "8080:80"
    
    # Custom volumes
    volumes:
      - "/host/path:/container/path"
    
    # Extra Docker args
    args:
      - "--cap-add=SYS_ADMIN"
```

👉 **[Full configuration reference](./docs/configuration.md)**

---

## 📚 Examples

Check out our [examples directory](./examples/) for:

- [Basic cluster setup](./examples/basic-cluster.yaml)
- [Multi-node cluster](./examples/multi-node-cluster.yaml)
- [With custom CNI](./examples/custom-cni.yaml)
- [Load balancer services](./examples/loadbalancer-service.yaml)
- [External node joining](./examples/external-node.md)

---

## 🎯 Use Cases

### Development Environments
Create isolated Kubernetes environments for each developer or feature branch.

### CI/CD Pipelines
Spin up temporary clusters for testing and tear them down when done.

### Learning Kubernetes
Perfect for learning K8s without the overhead of managing a full cluster.

### Local Testing
Test your applications in a real Kubernetes environment before deploying.

### Hybrid Development
Join cloud resources to your local cluster for hybrid development scenarios.

---

## 🛠️ Common Commands

```bash
# Create a cluster (automatically connects)
vcluster create my-cluster

# List clusters
vcluster list

# Connect to a cluster
vcluster connect my-cluster

# Disconnect from a cluster
vcluster disconnect my-cluster

# Pause a cluster (save resources)
vcluster pause my-cluster

# Resume a cluster
vcluster resume my-cluster

# Delete a cluster
vcluster delete my-cluster

# Describe cluster
vcluster describe my-cluster

# View control plane logs
docker exec vcluster.cp.my-cluster journalctl -u vcluster --nopager

# View node logs (for node named worker-1)
docker exec vcluster.node.my-cluster.worker-1 journalctl -u kubelet --nopager
```

---

## 🔗 Related Projects

- **[vCluster](https://github.com/loft-sh/vcluster)** - The underlying virtual cluster technology
- **[vCluster Platform](https://github.com/loft-sh/loft)** - Management platform for vClusters
- **[KinD](https://github.com/kubernetes-sigs/kind)** - Kubernetes in Docker (alternative)

---

## 🤝 Contributing

vind is built on top of vCluster. The core code lives in the [vCluster repository](https://github.com/loft-sh/vcluster). 

This repository serves as:
- Documentation and guides for the Docker driver feature
- Examples and use cases
- Community resources

To contribute to the core feature, please see the [vCluster contributing guide](https://github.com/loft-sh/vcluster/blob/main/CONTRIBUTING.md).

---

## 📝 License

This project is licensed under the Apache 2.0 License - see the [LICENSE](LICENSE) file for details.

---

## 📞 Support

- 📖 [Documentation](https://www.vcluster.com/docs)
- 💬 [Slack Community](https://slack.vcluster.com/)
- 🐛 [Issue Tracker](https://github.com/loft-sh/vcluster/issues)
- 💡 [Discussions](https://github.com/loft-sh/vcluster/discussions)

---

<div align="center">

**Made with ❤️ by the Kubernetes community**

[⭐ Star us on GitHub](https://github.com/saiyam1814/vind) • [📖 Read the Docs](./docs/) • [💬 Join Slack](https://slack.loft.sh/)

</div>
