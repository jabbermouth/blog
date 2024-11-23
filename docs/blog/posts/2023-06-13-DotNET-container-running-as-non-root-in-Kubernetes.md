---
title: .NET Container Running as Non-Root in Kubernetes
description: A quick guide on how to get a standard .NET container running as a non-root, non privileged user in Kubernetes.
date: 2023-06-13
categories:
  - Kubernetes
  - .NET
  - Docker
  - Quick Tips
---
# .NET Container Running as Non-Root in Kubernetes

This is a quick guide on how to get a standard .NET container running as a non-root, non privileged user in Kubernetes. It is not a complete security guide but rather just enough if you require your pods to not run under root.

## Update Dockerfile
The first step is to update the Dockerfile. Two changes are required here; one to change the port and one to specify the user to use.

### Exported Port

At the start of the Dockerfile, replace any `EXPORT` statements with the following:

```dockerfile
ENV ASPNETCORE_URLS http://+:8000
EXPOSE 8000
```

This will expose your application to the cluster on port 8000 rather than port 80.

### User

Next, just before the `ENTRYPOINT` instruction, add the following line:

```dockerfile
USER $APP_UID
```

Now build and push the container to your container registry of choice either manually or via a CI/CD pipeline.

## Kubernetes Manifests

### Deployment Manifest

Add the following snippet to the deployment manifest under the container entry that is to be locked down:

```yaml
securityContext:
  allowPrivilegeEscalation: false
  runAsNonRoot: true
  runAsUser: 1654
```

### Service Manifest

As the exported container port has now changed, update any service you may have defined so it looks similar to the following service manifest:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: your-service
  namespace: service-namespace
spec:
  selector:
     app: your-app
  ports:
    - name: http
      port: 80
      targetPort: 8000
  type: ClusterIP
```

The pod will still be accessible via its service on port 80 so things like ingress or gateway definitions or references from other apps do not need to be updated.