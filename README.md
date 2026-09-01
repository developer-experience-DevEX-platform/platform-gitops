# Platform GitOps

This repository contains small environment- and service-specific Helm value overrides for Kubernetes services managed by the DevEx platform. Only staging and production are supported.

## Architecture

```text
platform-helm-charts
    company-wide Kubernetes defaults and templates
              ↓
platform-gitops
    environment/service-specific overrides
              ↓
ApplicationSet
    automatically discovers environments/*/*
              ↓
Argo CD
    combines the central Helm chart with the appropriate values.yaml
```

The directory path is the deployment metadata. For example, `environments/staging/nodejs-ci-test` represents the `nodejs-ci-test` service in staging. A future ApplicationSet will derive the environment from path segment 2 and the service from path segment 3.

The ApplicationSet will use Argo CD multiple sources:

1. `platform-helm-charts` for the central service chart.
2. `platform-gitops` for the matching environment values.

The ApplicationSet will be added after the EKS and Argo CD destination-cluster model is established. This repository does not invent cluster names, Kubernetes API endpoints, or per-service Application manifests.

## Scale and developer experience

The structure scales linearly. Fifty services across two environments produce up to 100 small values files rather than copied Kubernetes manifests.

Application developers should not manually create these files. Backstage will eventually create:

```text
environments/staging/<service>/values.yaml
environments/production/<service>/values.yaml
```

The CD workflow will update the staging `image.tag` after container release. Production promotion will update the production `image.tag` with the same staging-tested immutable Git SHA. The image is not rebuilt between environments, and `latest` tags are never used.

## Secrets

Do not store Kubernetes secrets, AWS credentials, IAM role ARNs, or Argo CD credentials here. A platform-supported secrets mechanism will be introduced separately.
