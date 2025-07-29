# Traefik + ACME Configuration

This document describes how Traefik was configured as an Ingress Controller with Let's Encrypt (ACME) certificate management using DNS-01 challenges via Route53.

## Setup Summary

### Traefik Deployment

- Traefik is installed via Helm in the `traefik` namespace.
- Deployment is configured with:
  - `deployment.additionalVolumes` and `deployment.additionalVolumeMounts` to mount persistent ACME storage.
  - An NFS volume mounted at `/acme`:
    ```yaml
    deployment:
      additionalVolumeMounts:
        - name: acme-storage
          mountPath: /acme
      additionalVolumes:
        - name: acme-storage
          nfs:
            server: 10.60.1.15
            path: /data/services/acme
    ```
  - Verified that ACME cert data is being written to `acme.json` at the mounted path.

### ACME Resolver Configuration

- Traefik is configured to use Let's Encrypt via DNS challenge with Route53:
    ```yaml
    certificatesResolvers:
      letsencrypt:
        acme:
          caServer: https://acme-v02.api.letsencrypt.org/directory
          email: kevin@coolkev.com
          storage: /acme/acme.json
          dnsChallenge:
            provider: route53
            resolvers:
              - 1.1.1.1:53
              - 8.8.8.8:53
            delayBeforeCheck: 0
    ```

- AWS credentials are provided via a Kubernetes `Secret` referenced with `envFrom`.

### TLS Configuration

- Traefik is set to use wildcard TLS certs by default:
    ```yaml
    tls:
      options:
        default:
          sniStrict: false
      stores:
        default:
          defaultGeneratedCert:
            resolver: letsencrypt
    ```

### Ingress Resources

- Ingress objects reference `ingressClassName: traefik`.
- TLS section is **optional** in Ingress manifests — if omitted, Traefik uses the default wildcard cert.

    Example:
    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: example
      namespace: default
    spec:
      ingressClassName: traefik
      rules:
        - host: example.coolkev.com
          http:
            paths:
              - path: /
                pathType: Prefix
                backend:
                  service:
                    name: example-service
                    port:
                      number: 80
    ```

### Verification

- Certificates successfully issued by Let's Encrypt.
- Verified by `curl` output and valid TLS handshake.
- Confirmed `acme.json` updates and Traefik logs.
- Wildcard cert for `*.coolkev.com` was applied automatically to new Ingresses.

---

## Notes

- Use `external-dns` to manage DNS records automatically via annotations.
- Legacy `LoadBalancer` Services are no longer needed for HTTP(S) traffic.
- Traefik logs do not show certificate issuance verbosely; use the mounted storage to verify.

