# Snapshot and Restore

Save your entire vind cluster — deployments, services, configmaps, PVCs, and all persistent data — into a single portable file. Restore it on any machine with Docker. One command each way.

No other local Kubernetes tool supports this. Not KinD, not k3d, not minikube.

## Prerequisites

- Docker Desktop running
- vCluster CLI v0.34.0+
- Docker set as default driver: `vcluster use driver docker`

---

## What Gets Captured

| Data | Included? |
|------|-----------|
| All Kubernetes objects (Deployments, Services, ConfigMaps, Secrets) | Yes |
| PVC data (databases, files, anything on a PersistentVolumeClaim) | Yes |
| Container images in containerd cache | Yes |
| PKI certificates | Yes |
| vCluster config and tokens | Yes |

---

## Example 1: Backup and Restore a Stateful Stack

### Step 1: Create a cluster

```bash
vcluster create my-project
```

### Step 2: Deploy a stateful application

```bash
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: Namespace
metadata:
  name: production
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: production
data:
  APP_VERSION: "3.2.1"
  ENVIRONMENT: "production"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-storage
  namespace: production
spec:
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 100Mi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  namespace: production
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: app
        image: nginx:alpine
        ports:
        - containerPort: 80
        envFrom:
        - configMapRef:
            name: app-config
        volumeMounts:
        - name: storage
          mountPath: /data
        command:
        - sh
        - -c
        - |
          echo "App data created at $(date)" > /data/app.log
          echo "Active accounts: 1,247" >> /data/app.log
          nginx -g 'daemon off;'
      volumes:
      - name: storage
        persistentVolumeClaim:
          claimName: app-storage
---
apiVersion: v1
kind: Service
metadata:
  name: web-app
  namespace: production
spec:
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
EOF

kubectl wait --for=condition=ready pod -l app=web-app -n production --timeout=60s
```

Verify persistent data is written:

```bash
kubectl exec deploy/web-app -n production -- cat /data/app.log
# App data created at Sat Apr  5 12:00:00 UTC 2026
# Active accounts: 1,247
```

### Step 3: Take a snapshot

```bash
# Snapshot to a local file (auto-named with timestamp)
vcluster snapshot create my-project

# Or specify a path
vcluster snapshot create my-project ./backups/my-project.tar.gz
```

### Step 4: Delete the cluster

```bash
vcluster delete my-project
```

Everything is gone — containers, volumes, kubeconfig context.

### Step 5: Restore

```bash
vcluster restore my-project ./backups/my-project.tar.gz
```

### Step 6: Verify everything survived

```bash
# Wait for controllers to reconcile pods (~15s)
sleep 15

kubectl get all,configmap,pvc -n production
# All resources back, pods Running (1 restart is normal — kubelet detects stale containers)

kubectl exec deploy/web-app -n production -- cat /data/app.log
# App data created at Sat Apr  5 12:00:00 UTC 2026   <- original timestamp
# Active accounts: 1,247                              <- PVC data intact
```

---

## Example 2: Share an Environment via OCI Registry

Push a snapshot to GHCR (or any OCI registry) so teammates can restore it without file transfers.

```bash
# Team lead: set up the full stack, then push
vcluster create team-env
helm install prometheus prometheus-community/kube-prometheus-stack
kubectl apply -f our-app-manifests/

vcluster snapshot create team-env oci://ghcr.io/myorg/dev-envs:full-stack

# Every teammate — one command, no file download needed
vcluster restore team-env oci://ghcr.io/myorg/dev-envs:full-stack
```

Authentication uses your existing `docker login` credentials. Works with GHCR, AWS ECR, Azure ACR, Harbor, and any OCI-compliant registry.

---

## Example 3: Clone an Environment with a Different Name

The `--restore` flag on `vcluster create` lets you spin up a new cluster from an existing snapshot under a different name. Perfect for debugging a staging issue locally:

```bash
# Snapshot was from "staging"
vcluster snapshot create staging oci://ghcr.io/myorg/snapshots:staging-v1

# Create a local debug copy under a new name
vcluster create debug-copy --restore oci://ghcr.io/myorg/snapshots:staging-v1

# All data, deployments, and PVCs from staging — now in a cluster called debug-copy
kubectl get all --all-namespaces
```

---

## Example 4: S3 / S3-Compatible Storage (MinIO)

```bash
# Start MinIO locally for testing
docker run -d --name minio -p 9000:9000 -p 9001:9001 \
  -e MINIO_ROOT_USER=minioadmin \
  -e MINIO_ROOT_PASSWORD=minioadmin \
  minio/minio server /data --console-address ":9001"

# Create a bucket
docker run --rm --network host --entrypoint sh minio/mc -c "
  mc alias set local http://localhost:9000 minioadmin minioadmin &&
  mc mb local/vcluster-snapshots
"

# Push snapshot (credentials are base64-encoded in the URL)
ACCESS_KEY=$(echo -n "minioadmin" | base64)
SECRET_KEY=$(echo -n "minioadmin" | base64)
ENDPOINT=$(echo -n "http://localhost:9000" | base64)

vcluster snapshot create my-cluster \
  "s3://vcluster-snapshots/my-snapshot?access-key-id=${ACCESS_KEY}&secret-access-key=${SECRET_KEY}&url=${ENDPOINT}&region=us-east-1&force-path-style=true"

# Restore from MinIO
vcluster restore my-cluster \
  "s3://vcluster-snapshots/my-snapshot?access-key-id=${ACCESS_KEY}&secret-access-key=${SECRET_KEY}&url=${ENDPOINT}&region=us-east-1&force-path-style=true"
```

For AWS S3, omit the `url` and `force-path-style` params and use your actual credentials and region.

---

## Example 5: Multi-Node Cluster Snapshot

Snapshots capture all worker node volumes — including PVC data stored on workers.

```bash
# Create a multi-node cluster
vcluster create multi-node -f multi-node.yaml

# Verify 3 workers + control plane
kubectl get nodes

# Deploy something with PVCs spread across workers
kubectl apply -f affinity.yaml

# Snapshot — captures all node volumes automatically
vcluster snapshot create multi-node ./multi-node-backup.tar.gz

# Delete and restore
vcluster delete multi-node
vcluster restore multi-node ./multi-node-backup.tar.gz

# All nodes and PVC data come back
kubectl get nodes
```

On restore, worker nodes with existing volumes skip the `kubeadm join` step and re-register directly with the API server, preserving all PVC data.

---

## Backup Before Risky Changes

```bash
# Snapshot before anything potentially destructive
vcluster snapshot create my-cluster

# Try the experiment
kubectl apply -f experimental-crd.yaml
helm upgrade my-app ./new-chart-version

# Something broke? Roll back in seconds
vcluster delete my-cluster
vcluster restore my-cluster ./my-cluster-snapshot-*.tar.gz
```

---

## Tips and Limitations

- **Snapshot size**: A minimal cluster is ~300 MB compressed, mostly containerd image layers. Images re-pull on restore if needed, so you can delete them from the archive manually to reduce size.
- **Architecture**: Snapshots are architecture-specific. An ARM (Apple Silicon) snapshot won't restore on x86 Linux.
- **OCI auth**: Uses your existing `docker login` credentials. The macOS Keychain (`credsStore: osxkeychain`) can cause issues with Docker Hub — use GHCR or ECR instead.
- **Windows**: LoadBalancer services and the Docker driver have limited Windows support.
- **Check snapshot status**: `vcluster snapshot get my-cluster <target>` tracks async snapshot progress.
