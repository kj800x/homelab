# TODO

- I want an operator that makes it easy to deploy new versions of my custom images. "homelab orion". "DeployConfig" CRD maybe? It watches github for new builds and monitors them. It also provides a UI for rolling images forwards to newer commits (or rolling back I suppose). It can manage updates to anything that eventually makes pods (Jobs, CronJobs, Deployments, etc).
- I want to experiment with helm a bit more. Actually understand how helm charts are supposed to work, not just the way I'm half using them with argo.
- I want to take the manually deployed resources and make them managed in argo.
- I want to move all of the manually create secrets and move them into 1pw.
- I want discord notifications for when syncs start and finish and when deployments execute. CI/CD style info notifs.
- I want better kube metrics in victoria metrics generally. I want to know when a pod is failing.
- I want to set up better alerting in grafana.
- Is there a kube prometheus operator so that pod metrics are autoingested? Barring that, I could add the promscrape file as a config map instead of a file on disk?
- I need to fix ZFS metrics and configure Grafana alerting on that
- I need to set up cloud backups of the NAS
- I want to deploy a scan server to integrate with paperless-ngx
- I want to redeploy paperless-ngx
- I want to configure SSL with dns challenges and ACME certbot
- I want to configure an Ingress controller with SSL
- I want automatic updating of the *.home.coolkev.com route53 zone.
