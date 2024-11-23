---
title: Kubernetes starts responding with “You must be logged in to the server” error – how to fix it!
description: Fixing the “You must be logged in to the server” error in Kubernetes.
date: 2021-03-03
categories:
  - Kubernetes
  - Quick Tips
---
# Kubernetes starts responding with “You must be logged in to the server” error – how to fix it!

This is quite a simple one to fix and happens after the certificates have been updated. All you are required to do is go on the master nodes and run the following command:

```bash
sudo cp /etc/kubernetes/admin.conf ./.kube/config
```

You can copy this config file to other machines if you do things remotely.