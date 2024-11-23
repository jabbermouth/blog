---
title: Forcibly terminating a Kubernetes pod
description: How to forcibly terminate a Kubernetes pod.
date: 2020-11-30
categories:
  - Kubernetes
  - Quick Tips
---
# Forcibly terminating a Kubernetes pod

Run the following command, replacing the italics with your pod's name. Use `-n namespace` if your pod is in another namespace.

```
kubectl delete pods your-pod-name --grace-period=0 --force
```