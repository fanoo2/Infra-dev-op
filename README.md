## Overriding Helm Global Values per Environment

To override Helm global values for different environments, you can create separate values files for each environment. For example:

- `values-staging.yaml` for staging environment
- `values-prod.yaml` for production environment

You can then install or upgrade your Helm chart using the `-f` flag to specify the value files like so:

```bash
helm upgrade --install my-release my-chart -f values-staging.yaml
```

## Required GitHub Secrets

When deploying to different environments, ensure that the following GitHub Secrets are configured:

- `KUBE_CONFIG_STAGING`: Kubernetes configuration for the staging environment.
- `KUBE_CONFIG_PROD`: Kubernetes configuration for the production environment.
- `DOCKER_USERNAME`: Docker Hub username for image authentication.
- `DOCKER_PASSWORD`: Docker Hub password for image authentication.

## RBAC Manifests

If required, RBAC manifests can be found under `k8s/rbac.yaml`. Make sure to review and apply them as necessary for your deployment configuration.