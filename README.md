# Jellyfin Kubernetes Deployment

This directory contains Kubernetes manifests for deploying Jellyfin media server.

## Components

- **Namespace**: `jellyfin`
- **Deployment**: Single replica Jellyfin server using `jellyfin/jellyfin:preview`
- **Service**: LoadBalancer service exposing HTTP (8096) and HTTPS (8920)
- **PersistentVolumeClaims**:
  - `jellyfin-config`: 10Gi for configuration files
  - `jellyfin-cache`: 5Gi for cache data
  - `jellyfin-media`: 100Gi for media files

## Deployment

Deploy Jellyfin to your cluster:

```bash
kubectl apply -f jellyfin-deployment.yaml
```

## Verify Deployment

Check the deployment status:

```bash
kubectl get pods -n jellyfin
kubectl get svc -n jellyfin
```

## Access Jellyfin

After deployment, get the service endpoint:

```bash
kubectl get svc jellyfin -n jellyfin
```

Access Jellyfin at `http://<EXTERNAL-IP>:8096` or `http://localhost:8096` if using port-forward:

```bash
kubectl port-forward -n jellyfin svc/jellyfin 8096:8096
```

## Customization

### Timezone
Update the `TZ` environment variable in the deployment:
```yaml
- name: TZ
  value: "America/New_York"  # Change to your timezone
```

### Storage
Adjust PVC sizes based on your needs:
- `jellyfin-config`: Configuration and database
- `jellyfin-cache`: Transcoding cache
- `jellyfin-media`: Your media library

### Resources
Modify CPU/Memory limits based on your workload:
```yaml
resources:
  requests:
    memory: "512Mi"
    cpu: "500m"
  limits:
    memory: "4Gi"
    cpu: "2000m"
```

## Cleanup

Remove Jellyfin from your cluster:

```bash
kubectl delete -f jellyfin-deployment.yaml
```

**Note**: This will also delete the PVCs and all data. Backup your data before deleting!
