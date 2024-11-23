---
title: Delete a Kubernetes resource that is stuck "Terminating"
description: Delete a Kubernetes resource that is stuck "Terminating".
date: 2024-05-06
categories:
  - Kubernetes
  - Quick Tips
---
# Delete a Kubernetes resource that is stuck "Terminating"

## General Resource

Run the following command to force a resource to delete. It works by removing all finalizers. Only use this if a resource has become “stuck”.

```bash
kubectl patch resourcetype resource-name -n resource-namespace -p '{"metadata":{"finalizers":[]}}' --type=merge
```

Replace `resourcetype` with the type of resource (e.g. pod, helmrelease, etc…), replace `resource-name` with the name of your resource and `resource-namespace` with the name of the resource which the resource belongs to.

If the resource has become orphaned (i.e. the namespace a resource belongs to has been deleted), recreate the namespace and then run the above command for each resource.

## Namespace

If a namespace is stuck and the above method doesn’t work, run the following command, replacing `my-namespace` with the name of your namespace to delete.

```bash
kubectl get ns my-namespace -o json | jq '.spec.finalizers = []' | kubectl replace --raw "/api/v1/namespaces/my-namespace/finalize" -f -
```

Solution found on Stack Overflow.