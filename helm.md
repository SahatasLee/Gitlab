# Installation by helm

The GitLab chart is intended to fit in a cluster with at least 8 vCPU and 30 GB of RAM. If you are trying to deploy a non-production instance, you can reduce the defaults to fit into a smaller cluster.

## Using Static IP address

https://docs.gitlab.com/charts/installation/tools.html

## Deploy Gitlab By Helm

Change `global.hosts.domain` to your domain.
If you use `gitlab.com`, you will get this `https://gitlab.gitlab.com`.

```bash
helm repo add gitlab https://charts.gitlab.io/
helm repo update
```

```bash
#
helm upgrade --install gitlab gitlab/gitlab \
  --timeout 600s \
  --set global.hosts.domain=sahatas.com \
  --set global.hosts.externalIP=10.111.0.117 \
  --set certmanager-issuer.email=sahatasnutlee@gmail.com \
  --set postgresql.image.tag=13.6.0 \
  --set global.edition=ce \
  --set global.time_zone=Asia/Bangkok \
  --namespace gitlab

# 
helm install gitlab gitlab/gitlab -n gitlab -f <CONFIG_VALUES_FILE>
```

Example

```bash
root@ansible-server:~# helm upgrade --install gitlab gitlab/gitlab \
>   --timeout 600s \
>   --set global.hosts.domain=sahatas.com \
>   --set global.hosts.externalIP=10.111.0.117 \
>   --set postgresql.image.tag=13.6.0 \
>   --set global.edition=ce \
>   --set global.time_zone=Asia/Bangkok
Release "gitlab" does not exist. Installing it now.
NAME: gitlab
LAST DEPLOYED: Mon Oct 23 17:09:29 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
=== CRITICAL
The following charts are included for evaluation purposes only. They will not be supported by GitLab Support
for production workloads. Use Cloud Native Hybrid deployments for production. For more information visit
https://docs.gitlab.com/charts/installation/index.html#use-the-reference-architectures.
- PostgreSQL
- Redis
- Gitaly
- MinIO

=== NOTICE
The minimum required version of PostgreSQL is now 13. See https://gitlab.com/gitlab-org/charts/gitlab/-/blob/master/doc/installation/upgrade.md for more details.

=== NOTICE
You've installed GitLab Runner without the ability to use 'docker in docker'.
The GitLab Runner chart (gitlab/gitlab-runner) is deployed without the `privileged` flag by default for security purposes. This can be changed by setting `gitlab-runner.runners.privileged` to `true`. Before doing so, please read the GitLab Runner chart's documentation on why we
chose not to enable this by default. See https://docs.gitlab.com/runner/install/kubernetes.html#running-docker-in-docker-containers-with-gitlab-runners
Help us improve the installation experience, let us know how we did with a 1 minute survey:https://gitlab.fra1.qualtrics.com/jfe/form/SV_6kVqZANThUQ1bZb?installation=helm&release=16-5

=== NOTICE
The in-chart NGINX Ingress Controller has the following requirements:
    - Kubernetes version must be 1.20 or newer (unless NGINX version is set
      to v1.2.1 via `nginx-ingress.controller.image.tag=v1.2.1`, which will
      restore Kubernetes 1.19 compatibility).
    - Ingress objects must be in group/version `networking.k8s.io/v1`.
```

Check Gitlab URL.

```bash
export GITLAB_HOSTNAME=$(kuexport GITLAB_HOSTNAME=$(kubectl get ingresses.extensions gitlab-unicorn \
    -o jsonpath='{.spec.rules[0].host}')
echo "Your GitLab URL is: https://${GITLAB_HOSTNAME}"
 
Your GitLab URL is: https://gitlab.35.225.196.151.xip.io
```

Ref.

```url
https://www.dbi-services.com/blog/deploy-gitlab-on-kubernetes-using-helm/
```

## Get Password

username: `root`

```bash
kubectl get secret gitlab-gitlab-initial-root-password -o jsonpath="{.data.password}" -n gitlab | base64 --decode ; echo
```

## Namespace remove stuck

```bash
(
NAMESPACE=gitlab
kubectl proxy &
kubectl get namespace $NAMESPACE -o json |jq '.spec = {"finalizers":[]}' >temp.json
curl -k -H "Content-Type: application/json" -X PUT --data-binary @temp.json 127.0.0.1:8001/api/v1/namespaces/$NAMESPACE/finalize
)
```

```bash
root@ansible-server:~# kubectl api-resources --verbs=list --namespaced -o name | xargs -n 1 kubectl get --show-kind --ignore-not-found -n gitlab
NAME                                                                            STATE     DOMAIN                       AGE
challenge.acme.cert-manager.io/gitlab-gitlab-tls-vt6r4-1330819766-2392614702    pending   gitlab.khaolakonline.com     3h2m
challenge.acme.cert-manager.io/gitlab-kas-tls-hmk2f-600222928-2177299830        pending   kas.khaolakonline.com        5h12m
challenge.acme.cert-manager.io/gitlab-minio-tls-vqjr8-2852618193-2582324073     pending   minio.khaolakonline.com      5h12m
challenge.acme.cert-manager.io/gitlab-registry-tls-mt79k-2903392422-127572445   pending   registry.khaolakonline.com   5h12m
```

## Storage class

Set default storage class

```bash
kubectl patch storageclass zpool-ext4 -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

If not set, will get this error.

```bash
Warning  FailedMount  5m23s (x2 over 5m25s)  kubelet            MountVolume.SetUp failed for volume "webservice-config" : failed to sync configmap cache: timed out waiting for the condition
Warning  FailedMount  5m23s (x2 over 5m25s)  kubelet            MountVolume.SetUp failed for volume "workhorse-config" : failed to sync configmap cache: timed out waiting for the condition
Warning  FailedMount  5m23s                  kubelet            MountVolume.SetUp failed for volume "init-webservice-secrets" : failed to sync secret cache: timed out waiting for the condition
```