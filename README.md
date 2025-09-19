# BookStack Helm Chart

A Helm chart for deploying BookStack - a simple, self-hosted, easy-to-use wiki platform - on Kubernetes.

## Features

- **External Database Support**: Designed to work with external MySQL/MariaDB databases
- **Persistent Storage**: Configurable PersistentVolumeClaim for file storage
- **Ingress with SSL**: Full support for Traefik ingress controller with cert-manager integration
- **Flexible Configuration**: Comprehensive values.yaml for easy customization
- **Production Ready**: Includes health checks, resource limits, and security contexts

## Prerequisites

- Kubernetes 1.19+
- Helm 3.2.0+
- External MySQL/MariaDB database
- Ingress controller (Traefik recommended)
- cert-manager (for automatic SSL certificates)

## Installation

1. **Generate APP_KEY**: First, generate a unique APP_KEY for BookStack:
   ```bash
   docker run -it --rm --entrypoint /bin/bash lscr.io/linuxserver/bookstack:latest appkey
   ```

2. **Create values file**: Copy and modify the example values:
   ```bash
   helm show values ./bookstack > my-values.yaml
   ```

3. **Configure the values**: Edit `my-values.yaml` with your specific settings:
   ```yaml
   bookstack:
     appKey: "base64:YOUR_GENERATED_KEY_HERE"
     appUrl: "https://bookstack.yourdomain.com"
   
   database:
     host: "mysql.example.com"
     name: "bookstack"
     username: "bookstack"
     password: "your_secure_password"
   
   ingress:
     hosts:
       - host: bookstack.yourdomain.com
         paths:
           - path: /
             pathType: Prefix
     tls:
       - secretName: bookstack-tls
         hosts:
           - bookstack.yourdomain.com
   ```

4. **Install the chart**:
   ```bash
   helm install bookstack ./bookstack -f my-values.yaml
   ```

## Configuration

### Key Configuration Options

| Parameter | Description | Default |
|-----------|-------------|---------|
| `bookstack.appKey` | Unique application key (required) | `""` |
| `bookstack.appUrl` | Base URL for BookStack | `"https://bookstack.example.com"` |
| `database.host` | Database host | `"mysql.example.com"` |
| `database.name` | Database name | `"bookstack"` |
| `database.username` | Database username | `"bookstack"` |
| `database.password` | Database password | `"changeme"` |
| `persistence.size` | Storage size for files | `"8Gi"` |
| `ingress.enabled` | Enable ingress | `true` |
| `ingress.className` | Ingress class name | `"traefik"` |

### Database Configuration

BookStack requires an external MySQL/MariaDB database. Make sure to:

1. Create a database for BookStack
2. Create a user with full privileges on that database
3. Configure the database connection in your values.yaml

### Storage Configuration

The chart creates a PersistentVolumeClaim for BookStack's file storage which includes:
- Uploaded files and images
- Themes and customizations  
- Backups
- Cache and session data

### Ingress and SSL

The chart includes Traefik-specific annotations and cert-manager integration:

```yaml
ingress:
  annotations:
    traefik.ingress.kubernetes.io/router.entrypoints: websecure
    traefik.ingress.kubernetes.io/router.tls: "true"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
```

## Upgrading

To upgrade BookStack:

1. Update the image tag in your values file
2. Run the upgrade command:
   ```bash
   helm upgrade bookstack ./bookstack -f my-values.yaml
   ```

## Uninstalling

To uninstall the chart:

```bash
helm uninstall bookstack
```

**Note**: This will not delete the PersistentVolumeClaim by default. To also delete the storage:

```bash
kubectl delete pvc bookstack-storage
```

## Troubleshooting

### Common Issues

1. **APP_KEY not set**: Make sure to generate and set a unique APP_KEY
2. **Database connection failed**: Verify database credentials and connectivity
3. **File permissions**: Check that PUID/PGID values match your storage requirements
4. **SSL certificate issues**: Ensure cert-manager is properly configured

### Accessing Logs

```bash
kubectl logs -l app.kubernetes.io/name=bookstack
```

## Support

- [BookStack Documentation](https://www.bookstackapp.com/docs/)
- [LinuxServer.io BookStack Image](https://docs.linuxserver.io/images/docker-bookstack)
- [GitHub Issues](https://github.com/BookStackApp/BookStack/issues)