# Install Metrics Server into Kubernetes running in Docker Desktop using Helm

If using the Helm chart to install Metrics Server in a Docker Desktop instance of Kubernetes, it may not start due to insecure certificates. To resolve this, run the following command when doing the install (it can also be applied to an existing installation):

```bash
helm upgrade --install --set args[0]='--kubelet-insecure-tls' metrics-server metrics-server/metrics-server
```

If the repo hasn’t been added already, run the following first:

```bash
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/
```