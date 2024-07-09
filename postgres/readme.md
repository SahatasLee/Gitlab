# Postgres

```bash
#
export POSTGRES_PASSWORD=$(kubectl get secret --namespace psql psql-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)
#
kubectl get secret --namespace psql psql-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d ; echo
#
kubectl create secret generic gitlab-postgres --from-literal=psql-password=<your-password>
# Example
kubectl create secret generic gitlab-postgres --from-literal=psql-password=PiiGVju6ut -n gitlab
```

## Install

```bash
helm install psql bitnami/postgresql --namespace psql --create-namespace \
--set primary.resourcesPreset="large" \
--set primary.service.type="LoadBalancer" \
--version 15.5.14
```