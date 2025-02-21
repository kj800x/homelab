Cluster is a k3s cluster with nodes 10.60.1.15 and 10.60.1.4

Traefik and ServiceLB are disabled (for now at least). ServiceLB conflicts with MetalLB.

MetalLB is deployed and has been given 10.60.2.0 - 10.60.1.254. The router was changed to make 10.60.0.0/16 the subnet, but still only DHCP 10.60.1.0/24

ArgoCD has been deployed from scratch

```sh
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

Everything else has been configured inside of this repo using Argo and the App of Apps pattern. One app was manually created that points at /apps and everything else should take care of itself automatically.

The main problems (as of this writing):
- While services can be accessed from the LAN with their MetalLB IPs and those IPs seem generally stable, it would be better for services to have DNS hostnames. I started to set up k8s-gateway, but unfortunately I can't point my router exclusively at it because I don't want to accidentally create a DNS resolver infinite loop and I'm not sure if there's an easy way to prevent that. I could possibly try that some day when nobody else is on the network. The recommendation I've been hearing so far is to set up a pihole which can be configured to forward anything for the .kube TLD to k8s-gateway, and send anything else directly to 1.1.1.1
- ArgoCD isn't managed in ArgoCD
- Haven't figured out jobs yet
- Haven't figured out the proper way to version deployments yet
- ArgoCD isn't actually doing the changes in the right order, and there's some order dependent stuff. In practice, spamming the sync button a bunch of times seems to work fine and it makes more progress each time.
