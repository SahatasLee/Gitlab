# Gitlab 

Install gitlab with external postgres and minio.

create database before install gitlab

create minio bucket before install gitlab

## Pre-install

create psql database name `gitlabhq_production`

```bash
#
kubectl get secret --namespace psql psql-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d ; echo
# secret for password
kubectl create secret generic gitlab-postgres --from-literal=psql-password=<your-password>
```

create minio bucket

https://min.io/docs/minio/linux/reference/minio-mc.html

```bash
# mc mb local/gitlab-registry-storage 
# mc mb local/gitlab-lfs-storage 
# mc mb local/gitlab-artifacts-storage 
# mc mb local/gitlab-uploads-storage 
# mc mb local/gitlab-packages-storage 
# mc mb local/gitlab-backup-storage

mc mb local/gitlab-registry-storage  local/gitlab-lfs-storage  local/gitlab-artifacts-storage  local/gitlab-uploads-storage  local/gitlab-packages-storage  local/gitlab-backup-storage
```

create minio secret

```bash
# https://gitlab.com/gitlab-org/charts/gitlab/blob/master/doc/charts/globals.md#connection
kubectl create secret generic gitlab-rails-storage \
    --from-file=connection=rails.minio.yaml

# https://gitlab.com/gitlab-org/charts/gitlab/tree/master/doc/charts/registry/#storage
# Example using S3
kubectl create secret generic registry-storage \
    --from-file=config=registry-storage.minio.yaml
```

create crt secret 

```bash
kubectl create secret tls gitlab-tls -n gitlab --key app.key --cert app.crt
```

## Installation

```bash
#
helm search repo gitlab/gitlab --versions | head -n 5

#
helm install gitlab gitlab/gitlab -n gitlab -f <CONFIG_VALUES_FILE>
```

## Password

```bash
kubectl get secret gitlab-gitlab-initial-root-password -o jsonpath="{.data.password}" -n gitlab | base64 --decode ; echo
```