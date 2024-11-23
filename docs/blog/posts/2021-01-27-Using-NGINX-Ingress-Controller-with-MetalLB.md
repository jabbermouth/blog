---
title: Using NGINX Ingress Controller with MetalLB
description: This document describes getting the NGINX Ingress Controller (the community version rather than the one from NGINX Inc) to work easily with MetalLB.
date: 2021-01-27
categories:
  - Kubernetes
  - Tutorials
---
# Using NGINX Ingress Controller with MetalLB

This document describes getting the NGINX Ingress Controller (the community version rather than the one from NGINX Inc) to work easily with MetalLB.

For the purposes of this guide, it's assumed that MetalLB is set up. In the example, we'll assume it's using an IP range of 192.168.10.x

Firstly, we'll download the manifest so we can make a change:

```bash
curl -o ingress-nginx.yaml https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.43.0/deploy/static/provider/baremetal/deploy.yaml
```

Once downloaded, we need to edit the file:

```bash
nano ingress-nginx.yaml
```

We now need to find the service that handles the ingress so press ++ctrl+"W"++ and search for:

```bash
name: ingress-nginx-controller
```

Next, replace the line type: NodePort with the following:

```yaml
  type: LoadBalancer
  loadBalancerIP: 192.168.10.254
```

You can use any IP within the MetalLB range but it's recommended you specify one to prevent it somehow changing in the future.

Another change you can make is update the deployment so it's a daemon set rather than a standard deployment. To do this, replace the line:

```yaml
kind: Deployment
```

With:

```yaml
kind: DaemonSet
```

When you're finished, save the document using `control`+`s` and then quit using `control`+`x`

Finally, we'll install everything by running the following command:

```bash
kubectl apply -f ingress-nginx.yaml
```

The ingress controller, once an ingress is configured, will be accessible on [http(s)://192.168.10.254/](http://192.168.10.254/)