# DevOps Infrastructure for Microservices

This repository contains production-grade Kubernetes infrastructure for a microservices platform, including Helm charts and CI/CD workflows for automated deployment.

## Architecture

The infrastructure supports the following microservices:
- **backend-api**: Main API service
- **webrtc-service**: WebRTC communication service
- **payments-service**: Payment processing service
- **moderation-service**: Content moderation service
- **design-system**: Static design system assets

## Directory Structure

```
├── k8s/
│   └── namespaces.yaml          # Kubernetes namespaces for staging and production
├── helm-charts/
│   ├── values.global.yaml       # Global Helm values
│   ├── backend-api/             # Helm chart for backend API
│   ├── webrtc-service/          # Helm chart for WebRTC service
│   ├── payments-service/        # Helm chart for payments service
│   ├── moderation-service/      # Helm chart for moderation service
│   └── design-system/           # Helm chart for design system
└── .github/workflows/
    ├── staging-deploy.yml       # CI/CD workflow for staging deployment
    └── production-deploy.yml    # CI/CD workflow for production deployment
```

## Helm Charts

Each microservice has its own Helm chart with the following configuration:

- **Replica Count**: 2 instances for high availability
- **Resource Limits**: 200m CPU, 256Mi memory
- **Resource Requests**: 100m CPU, 128Mi memory
- **Service Type**: ClusterIP
- **Ingress**: Enabled with service-specific hostnames

### Global Values

The `helm-charts/values.global.yaml` file contains shared configuration:

```yaml
image:
  tag: "v1.0.0"
DOMAIN: example.com
```

### Overriding Helm Global Values per Environment

To override Helm global values for different environments, you can create separate values files for each environment:

- `values-staging.yaml` for staging environment
- `values-prod.yaml` for production environment

Example deployment with custom values:

```bash
helm upgrade --install my-release my-chart \
  --namespace staging \
  --values helm-charts/values.global.yaml \
  --values values-staging.yaml
```

## CI/CD Workflows

### Staging Deployment

The staging deployment workflow (`staging-deploy.yml`) is triggered on git tags matching `v*.*.*` pattern:

1. Sets up Helm and kubectl
2. Authenticates to Kubernetes using staging credentials
3. Deploys all Helm charts to the staging namespace
4. Performs smoke tests

### Production Deployment

The production deployment workflow (`production-deploy.yml`) is triggered automatically after successful staging deployment:

1. Waits for staging deployment to complete successfully
2. Authenticates to Kubernetes using production credentials
3. Deploys all Helm charts to the production namespace

## Required GitHub Secrets

Configure the following secrets in your GitHub repository:

- `KUBE_CONFIG_STAGING`: Base64-encoded Kubernetes configuration for staging environment
- `KUBE_CONFIG_PROD`: Base64-encoded Kubernetes configuration for production environment
- `DOCKER_USERNAME`: Docker Hub username for image authentication
- `DOCKER_PASSWORD`: Docker Hub password for image authentication

## Deployment Process

1. **Tag a release**: Create and push a git tag
   ```bash
   git tag v1.0.0
   git push --tags
   ```

2. **Monitor staging deployment**: Watch the "Deploy to Staging" workflow in GitHub Actions

3. **Automatic production promotion**: Upon successful staging deployment, the "Promote to Production" workflow will automatically deploy to production

## Customization

### Service Configuration

Each service can be customized by modifying its respective `values.yaml` file in the helm chart directory. Key configuration options include:

- Docker image repository and tag
- Resource limits and requests
- Ingress hostname configuration
- Service port configuration

### Adding New Services

To add a new microservice:

1. Create a new Helm chart: `helm create helm-charts/new-service`
2. Customize the chart following the existing pattern
3. Update CI/CD workflows to include the new service
4. Update this README documentation

## RBAC Manifests

If your services require special cluster roles, add RBAC manifests under `k8s/rbac.yaml`. Review and apply them as necessary for your deployment configuration.

## Monitoring and Health Checks

The staging deployment includes a smoke test that verifies the health endpoint. Ensure your services implement health check endpoints for proper monitoring.

## Security Considerations

- All services run with minimal resource allocations
- Ingress is configured for each service with proper path routing
- Secrets management is handled through Kubernetes secrets
- Docker image authentication is configured through GitHub secrets