## Important links

Argo: http://10.60.2.7/
Grafana: http://10.60.2.5:3000/
Kube dashboard: https://10.60.2.3:443/

## Converting from docker

ChatGPT is pretty good so far:

```
Can I get a Kube loadbalancer service and deployment for this docker compose. Ensure that revisionHistoryLimit is set to 1:

INSERT COMPOSE HERE

Two changes:
- POSTGRES_PASSWORD should be a secret now named miniflux-postgres-secret with key POSTGRES_PASSWORD
- The data is now stored at /data/services/miniflux-postgres via NFS server 10.60.1.15
```

## Adding secrets

First push a commit to add the new namespace and sync that in Argo first.

In order to make a secret that can be consumed like this:

```yaml
- name: ADMIN_PASSWORD
  valueFrom:
    secretKeyRef:
      name: miniflux-admin-secret
      key: ADMIN_PASSWORD
- name: DATABASE_URL
  valueFrom:
    secretKeyRef:
      name: miniflux-admin-secret
      key: DATABASE_URL
```

You need to run a command like this:

```sh
kubectl create secret generic miniflux-admin-secret \
  --from-literal=ADMIN_PASSWORD='some secret value' \
  --from-literal=DATABASE_URL='postgres://miniflux:some secret value@miniflux-postgres/miniflux?sslmode=disable' \
  --namespace=miniflux
```

Obviously we don't check our secrets into git, so those need to be manually configured outside of argo.

Testing:

```sh
kubectl get secrets -n miniflux
```

See secret resource (values base64 encoded)

```sh
kubectl get secret miniflux-admin-secret -n miniflux -o yaml
```

Decode a value

```sh
kubectl get secret miniflux-admin-secret -n miniflux -o jsonpath="{.data.ADMIN_PASSWORD}" | base64 --decode
```
