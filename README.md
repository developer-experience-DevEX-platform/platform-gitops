# Platform GitOps

This repository contains the desired environment configuration for Kubernetes services deployed through the DevEx platform. It pins both the company Helm chart version and immutable service image versions for staging and production.

## Architecture and responsibilities

```text
Application repository
    source code, Dockerfile, CI, and container release
              ↓
platform-helm-charts
    reusable company Kubernetes templates, security defaults,
    and Kubernetes standards
              ↓
platform-gitops
    desired environment configuration, immutable image version,
    pinned chart version, and environment-specific Helm values
              ↓
Argo CD
    reads platform-gitops, combines its values with platform-helm-charts,
    and reconciles Kubernetes
```

Application developers should not manually construct these files during normal usage. Backstage and platform automation will eventually create initial service entries.

## Promotion model

```text
container release
    ↓
ECR image:<git-sha>
    ↓
staging values.yaml
    ↓
staging validation
    ↓
production values.yaml
```

The reusable Kubernetes GitOps CD workflow will later update the staging `image.tag` after a successful container release. Production promotion will update the production `image.tag` with the same staging-tested SHA. Images are never rebuilt between staging and production, and `latest` tags are not permitted.

## Secrets

Do not store application secrets directly in `platform-gitops`.

Future secrets will be delivered through a platform-supported mechanism such as External Secrets Operator, AWS Secrets Manager, and Kubernetes workload identity. None of these mechanisms are implemented yet.
