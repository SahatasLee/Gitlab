# Gitlab without tls

## Installations

```bash
helm install gitlab gitlab/gitlab -n gitlab -f <CONFIG_VALUES_FILE>
```

## Use Local DNS for poc

if you don't have DNS, you can use local dns.

on window `C:\Windows\System32\drivers\etc\hosts`

on linux `/etc/hosts`

add entries to the host files

```bash
10.111.0.117 gitlab.gitlab.com
10.111.0.117 kas.gitlab.com
10.111.0.117 minio.gitlab.com
10.111.0.117 registry.gitlab.com
```

## HTTP

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| https	| Boolean	| true	| If set to true, you will need to ensure the NGINX chart has access to the certificates. In cases where you have TLS-termination in front of your Ingresses, you probably want to look at global.ingress.tls.enabled. Set to false for external URLs to use http:// instead of https. |
| tls.enabled	| Boolean	| true | When set to false, this disables TLS in GitLab. This is useful for cases in which you cannot use TLS termination of Ingresses, such as when you have a TLS-terminating proxy before the Ingress Controller. If you want to disable https completely, this should be set to false together with global.hosts.https

```yml
# values.yaml
global:
  ingress:
    annotations:
      "nginx.ingress.kubernetes.io/ssl-redirect": "false"
```

## Disable TLS Verification

https://gitlab.com/gitlab-org/gitlab-runner/-/issues/36895

1. Create cert

```bash
openssl s_client -showcerts -connect {your.gitlab.hostname}:443 -servername {your.gitlab.hostname} < /dev/null 2>/dev/null | openssl x509 -outform PEM > {your.gitlab.hostname}.crt
```

```bash
openssl s_client -showcerts -connect gitlab.khaolakonline.com:443 -servername gitlab.khaolakonline.com < /dev/null 2>/dev/null | openssl x509 -outform PEM > gitlab.khaolakonline.com.crt
```

2. Create secret


```bash
kubectl create secret generic gitlab-cert-name \
--namespace <NAMESPACE> \
--from-file={your.gitlab.hostname}.crt
```

```bash
kubectl create secret generic gitlab-cert \
--namespace gitlab \
--from-file=gitlab.khaolakonline.com.crt
```

3. In runner values.yaml

```bash
certsSecretName: gitlab-cert-name
```

## ContainerD unsecure registry

