# Troubleshooting Guide

Common issues and solutions when using vind.

## Cluster Creation Issues

### Cluster Won't Start

**Symptoms:**
- `vcluster create` fails
- Container exits immediately
- No cluster appears in `vcluster list`

**Solutions:**

1. **Check Docker is running:**
   ```bash
   docker ps
   ```

2. **Check Docker resources:**
   ```bash
   docker system df
   docker system prune  # If needed
   ```

3. **Check available disk space:**
   ```bash
   df -h
   ```

4. **View detailed logs:**
   ```bash
   vcluster create my-cluster --debug
   ```

5. **Check Docker logs:**
   ```bash
   docker exec vcluster.cp.my-cluster journalctl -u vcluster --nopager
   ```

### Port Already in Use

**Symptoms:**
- Error: `port is already allocated`
- Cannot bind to port

**Solutions:**

1. **Find what's using the port:**
   ```bash
   # macOS/Linux
   lsof -i :8443
   
   # Or
   netstat -an | grep 8443
   ```

2. **Use a different port:**
   ```bash
   vcluster create my-cluster \
     --set experimental.docker.ports[0]="8444:8443"
   ```

3. **Stop conflicting service:**
   ```bash
   # Find and stop the service using the port
   ```

### Insufficient Resources

**Symptoms:**
- Container fails to start
- Out of memory errors
- CPU throttling

**Solutions:**

1. **Check Docker resources:**
   ```bash
   docker stats
   ```

2. **Increase Docker resources:**
   - Docker Desktop: Settings → Resources
   - Increase CPU and Memory allocation

3. **Pause other clusters:**
   ```bash
   vcluster list
   vcluster pause other-cluster
   ```

## Connection Issues

### Cannot Connect to Cluster

**Symptoms:**
- `vcluster connect` fails
- `kubectl` commands fail
- Connection refused errors

**Solutions:**

1. **Verify cluster is running:**
   ```bash
   vcluster list
   docker ps | grep vcluster
   ```

2. **Check cluster status:**
   ```bash
   vcluster describe my-cluster
   ```

3. **Reconnect:**
   ```bash
   vcluster connect my-cluster --update-current
   ```

4. **Check kubeconfig:**
   ```bash
   kubectl config get-contexts
   kubectl config use-context vcluster-docker_my-cluster
   ```

5. **Verify port is accessible:**
   ```bash
   # Get the port
   docker port vcluster-my-cluster
   
   # Test connection
   curl -k https://localhost:<port>
   ```

### Kubeconfig Not Found

**Symptoms:**
- `kubectl` says context not found
- Cannot switch contexts

**Solutions:**

1. **Reconnect to cluster:**
   ```bash
   vcluster connect my-cluster
   ```

2. **Manually add context:**
   ```bash
   vcluster connect my-cluster --update-current
   ```

3. **Disconnect from cluster:**
   ```bash
   vcluster disconnect my-cluster
   ```

3. **Check kubeconfig location:**
   ```bash
   echo $KUBECONFIG
   kubectl config view
   ```

## Load Balancer Issues

### LoadBalancer Service Has No External IP

**Symptoms:**
- Service created but EXTERNAL-IP is `<pending>`
- Cannot access service

**Solutions:**

1. **Load balancer is enabled by default**

2. **Check Docker network:**
   ```bash
   docker network inspect vcluster-my-cluster
   ```

4. **On macOS, check port forwarding:**
   ```bash
   # May need sudo for privileged ports
   # Load balancer is enabled by default
   ```

### Cannot Access LoadBalancer Service

**Symptoms:**
- Service has EXTERNAL-IP but cannot access
- Connection refused

**Solutions:**

1. **Check service status:**
   ```bash
   kubectl get svc my-service
   ```

2. **Verify IP is in Docker network:**
   ```bash
   docker network inspect vcluster-my-cluster | grep -A 5 "IPAM"
   ```

3. **Test connectivity:**
   ```bash
   curl http://<EXTERNAL-IP>
   ping <EXTERNAL-IP>
   ```

4. **Check firewall rules:**
   ```bash
   # macOS
   sudo pfctl -sr
   ```

## Registry Proxy Issues

### Registry Proxy Not Working

**Symptoms:**
- Images still pull from registry
- No caching benefit

**Solutions:**

1. **Verify containerd storage:**
   ```bash
   docker info | grep "Storage Driver"
   # Should show containerd or overlay2 with containerd
   ```

2. **Registry proxy is enabled by default**

3. **Enable containerd storage:**
   - See: https://docs.docker.com/engine/storage/containerd/
   - Restart Docker after enabling

4. **Verify containerd socket:**
   ```bash
   ls -la /var/run/docker/containerd/containerd.sock
   ```

5. **Check logs:**
   ```bash
   docker exec vcluster.cp.my-cluster journalctl -u vcluster --nopager | grep -i registry
   ```

## Sleep/Wake Issues

### Cannot Pause Cluster

**Symptoms:**
- `vcluster pause` fails
- Cluster doesn't stop

**Solutions:**

1. **Check cluster status:**
   ```bash
   vcluster list
   docker ps | grep vcluster
   ```

2. **Manually stop containers:**
   ```bash
   docker stop vcluster-my-cluster
   docker stop vcluster-node-my-cluster-*
   ```

3. **Check for stuck processes:**
   ```bash
   docker ps -a | grep vcluster
   ```

### Cannot Resume Cluster

**Symptoms:**
- `vcluster resume` fails
- Cluster doesn't start

**Solutions:**

1. **Check container status:**
   ```bash
   docker ps -a | grep vcluster
   ```

2. **Manually start containers:**
   ```bash
   docker start vcluster-my-cluster
   docker start vcluster-node-my-cluster-*
   ```

3. **Check Docker resources:**
   ```bash
   docker system df
   ```

4. **View container logs:**
   ```bash
   docker exec vcluster.cp.my-cluster journalctl -u vcluster --nopager
   ```

## Network Issues

### Cannot Reach Services

**Symptoms:**
- Pods cannot communicate
- Services unreachable
- DNS resolution fails

**Solutions:**

1. **Check network configuration:**
   ```bash
   docker network inspect vcluster-my-cluster
   ```

2. **Verify CNI is working:**
   ```bash
   kubectl get pods -n kube-system | grep cni
   ```

3. **Check pod networking:**
   ```bash
   kubectl get pods -o wide
   kubectl describe pod <pod-name>
   ```

4. **Test DNS:**
   ```bash
   kubectl run -it --rm debug --image=busybox -- nslookup kubernetes.default
   ```

### DNS Not Working

**Symptoms:**
- Cannot resolve service names
- DNS queries fail

**Solutions:**

1. **Check CoreDNS:**
   ```bash
   kubectl get pods -n kube-system | grep coredns
   kubectl logs -n kube-system <coredns-pod>
   ```

2. **Verify DNS service:**
   ```bash
   kubectl get svc -n kube-system kube-dns
   ```

3. **Check DNS config:**
   ```bash
   kubectl get configmap -n kube-system coredns -o yaml
   ```

## External Node Issues

### Cannot Join External Node

**Symptoms:**
- Join command fails
- Node doesn't appear in cluster

**Solutions:**

1. **Verify VPN is enabled:**
   ```bash
   vcluster describe my-cluster | grep -i vpn
   ```

2. **Check join token:**
   ```bash
   # Get valid join token
   kubectl get secret join-token -o jsonpath='{.data.token}' | base64 -d
   ```

3. **Verify network connectivity:**
   ```bash
   # From external node
   ping <control-plane-ip>
   curl -k https://<control-plane-ip>:8443
   ```

4. **Check platform URL:**
   ```bash
   # Ensure platform is accessible
   curl https://<platform-url>/health
   ```

5. **View join logs:**
   ```bash
   # On external node
   journalctl -u vcluster-join
   ```

## Performance Issues

### Slow Cluster Operations

**Symptoms:**
- Commands take long time
- High CPU/memory usage

**Solutions:**

1. **Check resource usage:**
   ```bash
   docker stats
   ```

2. **Pause unused clusters:**
   ```bash
   vcluster list
   vcluster pause unused-cluster
   ```

3. **Clean up Docker:**
   ```bash
   docker system prune -a
   ```

4. **Increase Docker resources:**
   - Docker Desktop: Settings → Resources

### High Memory Usage

**Symptoms:**
- System running out of memory
- Docker containers killed

**Solutions:**

1. **Check memory usage:**
   ```bash
   docker stats
   ```

2. **Limit cluster resources:**
   ```yaml
   experimental:
     docker:
       args:
         - "--memory=2g"
         - "--cpus=2"
   ```

3. **Pause unused clusters:**
   ```bash
   vcluster pause my-cluster
   ```

## Getting Help

### Debug Mode

Enable debug logging:

```bash
vcluster create my-cluster --debug
```

### View Logs

```bash
# Control plane logs
docker exec vcluster.cp.my-cluster journalctl -u vcluster --nopager

# Node logs (for node named worker-1)
docker exec vcluster.node.my-cluster.worker-1 journalctl -u kubelet --nopager
# or for containerd
docker exec vcluster.node.my-cluster.worker-1 journalctl -u containerd --nopager
```

### Collect Information

When reporting issues, collect:

1. **vCluster version:**
   ```bash
   vcluster version
   ```

2. **Docker version:**
   ```bash
   docker version
   ```

3. **System info:**
   ```bash
   uname -a
   docker info
   ```

4. **Cluster status:**
   ```bash
   vcluster list
   vcluster describe my-cluster
   ```

5. **Logs:**
   ```bash
   docker exec vcluster.cp.my-cluster journalctl -u vcluster --nopager > vcluster.log
   ```

### Community Support

- 📖 [Documentation](https://www.vcluster.com/docs)
- 💬 [Slack Community](https://slack.loft.sh/)
- 🐛 [GitHub Issues](https://github.com/loft-sh/vcluster/issues)
- 💡 [Discussions](https://github.com/loft-sh/vcluster/discussions)
