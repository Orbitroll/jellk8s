# Jellyfin Media Stack - Kubernetes Deployment

Complete media automation stack running on Kubernetes with Jellyfin, Sonarr, Radarr, Bazarr, Jackett, qBittorrent, Jellyseerr, and FlareSolverr.

## Components

### Media Server
- **Jellyfin**: Media server (preview image)
  - HTTP: 8096, HTTPS: 8920
  - Config: `C:\jellyfin-config`
  - Cache: `C:\jellyfin-cache`
  - Media: D/E/F drives (Movies, Tv Shows, heb-dub)

### Download Management
- **qBittorrent**: Torrent client
  - Port: 8080
  - Config: `C:\qbittorrent-config`
  - Downloads: `C:\downloads`

### Content Management
- **Sonarr**: TV show management (Port: 8989)
  - Config: `C:\sonarr-config`
  - TV Shows: D/E/F drives
  
- **Radarr**: Movie management (Port: 7878)
  - Config: `C:\radarr-config`
  - Movies: D/E/F drives
  
- **Bazarr**: Subtitle management (Port: 6767)
  - Config: `C:\bazarr-config`

### Request & Indexing
- **Jellyseerr**: Media request management (Port: 5055)
  - Config: `C:\jellyseerr-config`
  
- **Jackett**: Torrent indexer proxy (Port: 9117)
  - Config: `C:\jackett-config`
  
- **FlareSolverr**: Cloudflare bypass (Ports: 8191, 8192)

## Prerequisites

- Docker Desktop for Windows with Kubernetes enabled
- Windows drives: C:, D:, E:, F:
- Storage structure:
  - D:\Movies, D:\Tv Shows, D:\heb-dub
  - E:\Movies, E:\Tv Shows, E:\heb-dub
  - F:\Movies, F:\Tv Shows, F:\heb-dub
  - C:\downloads (for completed downloads)

## Deployment

### 1. Deploy Storage
```powershell
kubectl apply -f namespace.yaml
kubectl apply -f Jellyfin/storage.yaml
kubectl apply -f jackett/storage.yaml
kubectl apply -f sonarr/storage.yaml
kubectl apply -f radarr/storage.yaml
kubectl apply -f bazarr/storage.yaml
kubectl apply -f jellyseerr/storage.yaml
kubectl apply -f qbittorrent/storage.yaml
```

### 2. Deploy Applications
```powershell
kubectl apply -f Jellyfin/deployment.yaml
kubectl apply -f jackett/deployment.yaml
kubectl apply -f flarresolver/deployment.yaml
kubectl apply -f sonarr/deployment.yaml
kubectl apply -f radarr/deployment.yaml
kubectl apply -f bazarr/deployment.yaml
kubectl apply -f jellyseerr/deployment.yaml
kubectl apply -f qbittorrent/deployment.yaml
```

### 3. Deploy Services
```powershell
kubectl apply -f Jellyfin/services.yaml
kubectl apply -f jackett/services.yaml
kubectl apply -f flarresolver/services.yaml
kubectl apply -f sonarr/services.yaml
kubectl apply -f radarr/services.yaml
kubectl apply -f bazarr/services.yaml
kubectl apply -f jellyseerr/services.yaml
kubectl apply -f qbittorrent/services.yaml
```

### 4. Deploy Ingress (Optional)
```powershell
# Install nginx ingress controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml

# Apply ingress
kubectl apply -f ingress.yaml

# Port-forward to access via hostnames
kubectl port-forward -n ingress-nginx service/ingress-nginx-controller 80:80
```

## Access Services

### Via LoadBalancer (Direct)
- Jellyfin: http://localhost:8096
- qBittorrent: http://localhost:8080
- Sonarr: http://localhost:8989
- Radarr: http://localhost:7878
- Bazarr: http://localhost:6767
- Jackett: http://localhost:9117
- Jellyseerr: http://localhost:5055
- FlareSolverr: http://localhost:8191

### Via Ingress (With port-forward)
- http://jellyfin.local
- http://qbittorrent.local
- http://sonarr.local
- http://radarr.local
- http://bazarr.local
- http://jackett.local
- http://jellyseerr.local
- http://flaresolverr.local

Add to `C:\Windows\System32\drivers\etc\hosts`:
```
127.0.0.1 jellyfin.local
127.0.0.1 qbittorrent.local
127.0.0.1 sonarr.local
127.0.0.1 radarr.local
127.0.0.1 bazarr.local
127.0.0.1 jackett.local
127.0.0.1 jellyseerr.local
127.0.0.1 flaresolverr.local
```

## Configuration

### qBittorrent
- Get temporary password: `kubectl logs -n jellyfin deployment/qbittorrent | Select-String password`
- Set downloads to: `/downloads`

### Radarr/Sonarr
- Add qBittorrent:
  - Host: `qbittorrent` (or `qbittorrent.jellyfin.svc.cluster.local`)
  - Port: `8080`
  - Downloads: `/downloads`

### Jellyseerr
- Configure Radarr:
  - Host: `radarr`
  - Port: `7878`
  - API Key: From Radarr Settings → General
  
- Configure Sonarr:
  - Host: `sonarr`
  - Port: `8989`
  - API Key: From Sonarr Settings → General

### Path Mappings
All services see the same paths:
- `/downloads` → C:\downloads
- `/movies/D`, `/movies/E`, `/movies/F` → Movie folders
- `/tv/D`, `/tv/E`, `/tv/F` → TV show folders

## Verify Deployment

```powershell
# Check all pods
kubectl get pods -n jellyfin

# Check all services
kubectl get svc -n jellyfin

# Check storage
kubectl get pv,pvc -n jellyfin
```

## Troubleshooting

### Pods not starting
```powershell
kubectl describe pod <pod-name> -n jellyfin
kubectl logs <pod-name> -n jellyfin
```

### Storage issues
- Ensure directories exist on Windows host
- Check PV/PVC status: `kubectl get pv,pvc -n jellyfin`

### Service connectivity
```powershell
# Test from one pod to another
kubectl exec -n jellyfin deployment/jellyseerr -- wget -O- http://radarr:7878/api/v3/system/status
```

## Cleanup

```powershell
# Delete all resources
kubectl delete namespace jellyfin

# Delete PVs
kubectl delete pv -l app=jellyfin

# Remove ingress controller
kubectl delete namespace ingress-nginx
```

**Note**: This will delete all configurations. Backup data from C:\ config folders before deleting!
