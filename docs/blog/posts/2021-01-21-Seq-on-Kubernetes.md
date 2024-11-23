---
title: Seq on Kubernetes
description: This guide will tell you how to set up an HTTPS protected sec instance. This is using LetsEncrypt with cert-manager to get an SSL certificate so it assumes your Seq instance is public facing. It also uses Longhorn for storage.
date: 2021-01-21
categories:
  - Seq
  - Kubernetes
  - Tutorials
---
# Seq on Kubernetes

This guide will tell you how to set up an HTTPS protected sec instance. This is using LetsEncrypt with cert-manager to get an SSL certificate so it assumes your Seq instance is public facing. It also uses Longhorn for storage.

First, create a ClusterIssuer:

```bash
mkdir seq
cd seq
nano certmanager.yaml
```

Then insert the following:

```yaml
apiVersion: cert-manager.io/v1alpha3
kind: ClusterIssuer
metadata:
  name: letsencrypt-seq
  namespace: cert-manager
spec:
  acme:
    # The ACME server URL
    server: https://acme-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: your.email@yourdomain.com
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-seq
    # Enable the HTTP-01 challenge provider
    solvers:
    - http01:
        ingress:
          class: nginx
```

Don't forget to update your email address and then ++ctrl+"s"++ and ++ctrl+"x"++ to save and exit.

Once that's done, we'll create another file called `config.yaml` and save that. Populate it as follows, replacing with your domain in the three necessary places:

```yaml
control# config.yaml
ingress:
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: "letsencrypt-seq"
  tls:
    - hosts:
        - seq.yourdomain.com
      secretName: letsencrypt-seq

baseURI: https://seq.yourdomain.com

ui:
  ingress:
    enabled: true
    path: /
    hosts:
      - seq.yourdomain.com

persistence:
  storageClass: longhorn

resources:
  limits:
    memory: 4Gi

baseURI: https://seq.yourdomain.com
```

Once saved, we'll install the instance using Helm.

```bash
helm install -f config.yaml seq datalust/seq
```

After about a minute, you should be able to access Seq. I highly recommend turning on user authentication immediately.