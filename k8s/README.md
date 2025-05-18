# Colanode Helm Chart

A Helm chart for deploying Colanode - an open-source & local-first collaboration workspace - on Kubernetes.

## Introduction

This Helm chart deploys Colanode and its dependencies (PostgreSQL with pgvector, Valkey, and MinIO) on a Kubernetes cluster. Colanode is an all-in-one platform for easy collaboration, built to prioritize data privacy and control. It helps teams communicate, organize, and manage projects with a local-first approach.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.2.0+
- PV provisioner support in the underlying infrastructure

## Installing the Chart

### From Helm Repository (Recommended)

```bash
# Add the Colanode Helm repository
helm repo add colanode https://colanode.github.io/colanode-k8s/

# Update repositories
helm repo update

# Install the chart
helm install colanode colanode/colanode
```

### From Local Clone

If you need to modify the chart before installing:

```bash
# Clone the repository
git clone https://github.com/colanode/colanode-k8s.git
cd colanode-k8s

# Add the required Bitnami repository for dependencies
helm repo add bitnami https://charts.bitnami.com/bitnami

# Update dependencies
helm dependency update ./charts/colanode

# Install the chart with your custom values
helm install colanode ./charts/colanode -f values.yaml
```

## Configuration

The following table lists the configurable parameters of the Colanode chart and their default values. Refer to `values.yaml` for the full structure and comments.

| Parameter                        | Description                                      | Default                           |
| -------------------------------- | ------------------------------------------------ | --------------------------------- |
| `colanode.image.repository`      | Colanode server image repository                 | `ghcr.io/colanode/server`         |
| `colanode.image.tag`             | Colanode server image tag                        | `latest`                          |
| `colanode.replicaCount`          | Number of Colanode server replicas               | `1`                               |
| `colanode.service.port`          | Colanode server service port                     | `3000`                            |
| `colanode.config.*`              | Colanode server application configuration        | See `values.yaml`                 |
| `postgresql.enabled`             | Enable PostgreSQL dependency chart deployment    | `true`                            |
| `postgresql.image.repository`    | PostgreSQL image (with pgvector)                 | `pgvector/pgvector`               |
| `postgresql.image.tag`           | PostgreSQL version                               | `pg17`                            |
| `postgresql.auth.username`       | PostgreSQL username                              | `colanode_user`                   |
| `postgresql.auth.password`       | PostgreSQL password (set directly or use secret) | `""`                              |
| `postgresql.auth.database`       | PostgreSQL database name                         | `colanode_db`                     |
| `postgresql.auth.existingSecret` | Existing secret name for PostgreSQL password     | `""`                              |
| `valkey.enabled`                 | Enable Valkey dependency chart deployment        | `true`                            |
| `valkey.image.repository`        | Valkey image repository                          | `valkey/valkey`                   |
| `valkey.image.tag`               | Valkey version                                   | `8.1`                             |
| `valkey.auth.enabled`            | Enable Valkey authentication                     | `true`                            |
| `valkey.auth.password`           | Valkey password (set directly or use secret)     | `""`                              |
| `valkey.auth.existingSecret`     | Existing secret name for Valkey password         | `""`                              |
| `minio.enabled`                  | Enable MinIO dependency chart deployment         | `true`                            |
| `minio.image.repository`         | MinIO image repository                           | `minio/minio`                     |
| `minio.image.tag`                | MinIO version                                    | `RELEASE.2025-04-08T15-41-24Z`    |
| `minio.auth.rootUser`            | MinIO root user                                  | `minioadmin`                      |
| `minio.auth.rootPassword`        | MinIO root password (set directly or use secret) | `""`                              |
| `minio.auth.existingSecret`      | Existing secret name for MinIO credentials       | `""`                              |
| `minio.defaultBuckets`           | Buckets to create at initialization              | `colanode-avatars,colanode-files` |

### Colanode Server Environment Variables

The `colanode.config` section in values.yaml contains all environment variables for the Colanode server. Major configuration categories include:

| Category                   | Description                        | Key Environment Variables                          |
| -------------------------- | ---------------------------------- | -------------------------------------------------- |
| General Server Config      | Basic server configuration         | `NODE_ENV`, `PORT`, `SERVER_NAME`, `SERVER_MODE`   |
| Account Configuration      | User account settings              | `ACCOUNT_VERIFICATION_TYPE`, `ACCOUNT_OTP_TIMEOUT` |
| User Configuration         | User storage limits                | `USER_STORAGE_LIMIT`, `USER_MAX_FILE_SIZE`         |
| PostgreSQL Configuration   | Database connection settings       | `POSTGRES_URL`, `POSTGRES_SSL_*`                   |
| Redis/Valkey Configuration | Message queue and caching          | `REDIS_URL`, `REDIS_DB`, `REDIS_JOBS_QUEUE_NAME`   |
| S3 Configuration           | File storage for avatars and files | `S3_AVATARS_*`, `S3_FILES_*`                       |
| SMTP Configuration         | Email settings for notifications   | `SMTP_ENABLED`, `SMTP_HOST`, `SMTP_PORT`           |
| AI Configuration           | AI feature toggle (experimental)   | `AI_ENABLED`                                       |

You can specify parameters using the `--set key=value[,key=value]` argument to `helm install`. For nested values, use dot notation (e.g., `--set colanode.replicaCount=2`).

For example:

```bash
helm install colanode ./charts/colanode \
  --set postgresql.auth.password=mypassword \
  --set valkey.auth.password=myvalkeypassword \
  --set minio.auth.rootPassword=myminioadminpassword
```

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart:

```bash
helm install colanode ./charts/colanode -f my_values.yaml
```

## Features of Colanode

Colanode offers:

- **Real-Time Chat:** Stay connected with instant messaging for teams
- **Rich Text Pages:** Create documents, wikis, and notes with intuitive editor
- **Customizable Databases:** Organize information with structured data and dynamic views
- **File Management:** Store, share, and manage files within secure workspaces

## Secrets Management

This chart supports two methods for handling sensitive information like database passwords:

1.  **Direct Value in `values.yaml` (or `--set`):** If you set `postgresql.auth.password`, `valkey.auth.password`, or `minio.auth.rootPassword` directly, the chart will automatically create Kubernetes secrets for you.
    _(Note: This is less secure as the password might be stored in Git if you commit your `values.yaml`)_

2.  **Existing Kubernetes Secrets:** For production use, it's recommended to create secrets beforehand and reference them in `values.yaml`:

    ```bash
    # Create secrets (replace placeholders)
    kubectl create secret generic colanode-postgresql-secret --from-literal=postgres-password='YOUR_POSTGRES_PASSWORD'
    kubectl create secret generic colanode-valkey-secret --from-literal=redis-password='YOUR_VALKEY_PASSWORD'
    kubectl create secret generic colanode-minio-secret --from-literal=root-user='minioadmin' --from-literal=root-password='YOUR_MINIO_PASSWORD'
    ```

    Then reference these secrets in your `values.yaml`:

    ```yaml
    postgresql:
      auth:
        existingSecret: colanode-postgresql-secret
        # Ensure secretKeys.userPasswordKey matches the key in the secret ('postgres-password')

    valkey:
      auth:
        existingSecret: colanode-valkey-secret
        # Ensure secretKeys.redisPasswordKey matches the key in the secret ('redis-password')

    minio:
      auth:
        existingSecret: colanode-minio-secret
        # Ensure rootUserKey ('root-user') and rootPasswordKey ('root-password') match keys in the secret
    ```

## Persistence

This chart uses PVC for PostgreSQL, Valkey, and MinIO. By default, the chart creates PVCs with the default storage class.
