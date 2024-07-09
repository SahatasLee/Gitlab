# Postgres

```bash
#
export POSTGRES_PASSWORD=$(kubectl get secret --namespace psql psql-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)
#
kubectl get secret --namespace psql psql-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d
#
kubectl create secret generic gitlab-postgres --from-literal=psql-password=<your-password>
# Example
kubectl create secret generic gitlab-postgres --from-literal=psql-password=PiiGVju6ut -n gitlab
```