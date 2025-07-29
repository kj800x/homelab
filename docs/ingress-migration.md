# Migrating a Service to Traefik Ingress

This guide explains how to migrate an internal Kubernetes service from a LoadBalancer type to using Traefik as an Ingress controller with automatic TLS via Let's Encrypt (ACME).

## Prerequisites

- Traefik deployed via Helm with ACME DNS challenge (e.g., Route53) and wildcard cert configured
- Traefik exposed via LoadBalancer or other reachable method
- External DNS integration (e.g., `external-dns.alpha.kubernetes.io/hostname` annotations working)
- Your DNS wildcard (e.g. `*.home.coolkev.com`) points to the Traefik LoadBalancer IP

---

## Migration Steps

### 1. Remove LoadBalancer from the Service

Change the service from:

```yaml
spec:
  type: LoadBalancer
```

To:

```yaml
spec:
  type: ClusterIP
```

This keeps the service accessible inside the cluster without exposing it directly.

### 2. Create an Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: timelapse-service-rs
  namespace: home-sensors
  annotations:
    external-dns.alpha.kubernetes.io/hostname: timelapse.home.coolkev.com
spec:
  ingressClassName: traefik
  rules:
    - host: timelapse.home.coolkev.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: timelapse-service-rs
                port:
                  number: 80
```

To use the default wildcard TLS cert from Traefik’s ACME config, omit the `tls:` block.

### 3. Wait for DNS and Cert Propagation

- `external-dns` will update the A record
- Cert issuance via Let's Encrypt may take ~30–60 seconds
- Use `dig`, `nslookup`, or `curl -v` to verify changes

DNS TTL may cause delays in seeing updated A records. Use:

```bash
dig @1.1.1.1 timelapse.home.coolkev.com
```

to bypass local caching.

---

## Known Issues and Weirdness

### Self-signed cert error

If you access the hostname before Traefik has completed ACME issuance, you’ll see a self-signed cert:

```
curl: (60) SSL certificate problem: self-signed certificate
```

Try again in a minute.

### Stale DNS records

If the service used to be a LoadBalancer, your local resolver may still point to the old IP. Use:

```bash
sudo resolvectl flush-caches
# or
sudo systemd-resolve --flush-caches  # depending on distro
```

You can also test via a public DNS resolver:

```bash
dig @1.1.1.1 yourhost.home.coolkev.com
```

---

## Optional: TLS Section for Specific Cert

If you want Traefik to request a cert per-host (instead of using the wildcard):

```yaml
  tls:
    - hosts:
        - timelapse.home.coolkev.com
      secretName: timelapse-tls
```

This forces Traefik to do an ACME challenge for that domain and persist the result in the secret. Use this sparingly if you're already covering everything via a wildcard.

---

## Result

You now have:

- ClusterIP-only internal service
- Externally accessible via hostname
- Managed TLS from Let's Encrypt
- Automatic renewals (via Traefik + ACME)
