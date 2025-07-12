# Game Server Helm Chart

A Helm chart for deploying an MMO game server architecture on Kubernetes.

## Overview

This chart deploys a multi-zone game server setup using a containerized Bevy game server. Each zone runs as a separate deployment instance with its own service and HTTP routing configuration.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+
- Gateway API CRDs installed (for HTTPRoute support)
- Traefik Gateway or compatible gateway controller

## Installation

```bash
# Add the chart repository (if applicable)
helm repo add <repo-name> <repo-url>

# Install the chart
helm install game-server ./game-server-chart

# Or with custom values
helm install game-server ./game-server-chart -f custom-values.yaml
```

## Configuration

### Default Values

| Parameter | Description | Default |
|-----------|-------------|---------|
| `game-server.hostname` | Hostname for external access | `mmo.alanchiu.net` |
| `game-server.zones` | List of game zones to deploy | `[{id: "zone-0"}]` |
| `game-server.wsPort` | WebSocket port | `5000` |
| `game-server.httpPort` | HTTP port for health checks | `3000` |
| `game-server.deployment.name` | Base name for deployments | `mmo-game-server` |
| `game-server.deployment.requests.cpu` | CPU resource requests | `250m` |
| `game-server.deployment.requests.memory` | Memory resource requests | `256Mi` |
| `game-server.deployment.limits.cpu` | CPU resource limits | `1` |
| `game-server.deployment.limits.memory` | Memory resource limits | `1Gi` |
| `game-server.autoscaler.name` | Base name for HPA resources | `mmo-game-autoscaler` |
| `game-server.autoscaler.minReplicas` | Minimum number of replicas | `1` |
| `game-server.autoscaler.maxReplicas` | Maximum number of replicas | `3` |
| `game-server.service.name` | Base name for services | `mmo-game-server-lb` |
| `game-server.image.repository` | Container image repository | `alanyau1613/mmo-game-server` |
| `game-server.image.tag` | Container image tag | `latest` |
| `game-server.image.pullPolicy` | Image pull policy | `Always` |

### Adding Multiple Zones

To deploy multiple game zones, modify the `zones` configuration:

```yaml
game-server:
  zones:
    - id: zone-0
    - id: zone-1
    - id: zone-2
```

This will create separate deployments, services, and routes for each zone.

## Architecture

The chart creates the following resources for each zone:

1. **Deployment**: Runs the game server container with zone-specific arguments and resource limits
2. **Service**: Exposes both WebSocket and HTTP ports for the zone
3. **HTTPRoute**: Routes traffic to the appropriate zone based on path prefix
4. **HorizontalPodAutoscaler**: Automatically scales pods based on CPU utilization (80% threshold)

### Zone Routing

Each zone is accessible via a path prefix:
- Zone 0: `https://mmo.alanchiu.net/zone-0`
- Zone 1: `https://mmo.alanchiu.net/zone-1`

The HTTPRoute configuration automatically strips the zone prefix before forwarding to the backend service.

### Resource Management

Each zone deployment includes:
- **Resource Requests**: 250m CPU and 256Mi memory per pod
- **Resource Limits**: 1 CPU core and 1Gi memory per pod
- **Horizontal Pod Autoscaling**: Scales from 1-3 replicas based on 80% CPU utilization

### Health Checks

The deployment includes readiness probes that check the `/healthz` endpoint on the HTTP port. The container also implements graceful shutdown with a 60-second drain period.

## Upgrading

```bash
helm upgrade game-server ./game-server-chart
```

## Uninstalling

```bash
helm uninstall game-server
```

## Troubleshooting

### Common Issues

1. **Pods not starting**: Check that the container image is accessible and the image tag exists
2. **Gateway not routing**: Verify that the Traefik Gateway exists in the default namespace
3. **Health check failures**: Ensure the game server exposes the `/healthz` endpoint on the HTTP port

### Viewing Logs

```bash
# View logs for a specific zone
kubectl logs -l app=game-server,zone=zone-0

# View all game server logs
kubectl logs -l app=game-server
```

## Development

To modify this chart:

1. Update templates in the `templates/` directory
2. Modify default values in `values.yaml`
3. Test changes with `helm template` or `helm install --dry-run`

## License

[Add your license information here]
