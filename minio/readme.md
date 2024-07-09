# Gitlab Minio

https://docs.gitlab.com/charts/advanced/external-object-storage/minio.html

```bash
# https://gitlab.com/gitlab-org/charts/gitlab/blob/master/doc/charts/globals.md#connection
kubectl create secret generic gitlab-rails-storage \
    --from-file=connection=rails.minio.yaml

# https://gitlab.com/gitlab-org/charts/gitlab/tree/master/doc/charts/registry/#storage
# Example using S3
kubectl create secret generic registry-storage \
    --from-file=config=registry-storage.minio.yaml
```

## Installation

minio operator version 5.0.15

[Deploy Operator With Helm — MinIO Object Storage for Kubernetes](https://min.io/docs/minio/kubernetes/upstream/operations/install-deploy-manage/deploy-operator-helm.html)

```bash
# Add repo to helm
helm repo add minio-operator https://operator.min.io
# Install
helm install minio-operator minio-operator/operator --version 5.0.15 -n minio 
```

## Service

change service to LoadBalancer

```bash
# LoadBalancer
kubectl patch service -n minio console -p '
{
    "spec": {
        "type": "LoadBalancer"
    }
}'
```

## Access Token

minio operator console

```bash
kubectl -n minio-operator  get secret console-sa-secret -o jsonpath="{.data.token}" | base64 --decode
```

## Access key & Secret key

```bash
# Access key
kubectl -n minio-tenants get secrets demo-user-0 -o jsonpath="{.data.CONSOLE_ACCESS_KEY}" | base64 --decode

# Secret key
kubectl -n minio-tenants get secrets demo-user-0 -o jsonpath="{.data.CONSOLE_SECRET_KEY}" | base64 --decode
```

## Python SDK

[Python SDK](https://min.io/docs/minio/linux/developers/python/minio-py.html)

Example

[main.py](main.py)