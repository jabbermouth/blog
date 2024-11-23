---
title: Running Octopus Deploy Container With Kubernetes and Helm
description: When trying to run Kubernetes or Helm deployments from a local Octopus Deploy container, an error will be encountered because these tools aren’t available by default. One solution to this problem is create a custom container that includes them.
date: 2022-12-07
categories:
  - Octopus Deploy
  - Kubernetes
  - Helm
  - Tutorials
---
# Running Octopus Deploy Container With Kubernetes and Helm

When trying to run Kubernetes or Helm deployments from a local Octopus Deploy container, an error will be encountered because these tools aren’t available by default.

One solution to this problem is create a custom container that includes them. Below is an example of this.

```Dockerfile
FROM octopusdeploy/octopusdeploy:latest

RUN apt update
RUN apt install -y ca-certificates curl apt-transport-https
RUN curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
RUN curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | tee /usr/share/keyrings/helm.gpg > /dev/null
RUN echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | tee /etc/apt/sources.list.d/kubernetes.list
RUN echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | tee /etc/apt/sources.list.d/helm-stable-debian.list
RUN apt update
RUN apt install -y kubectl helm
```

To build the container, run the following command:

```powershell
docker build --pull -t octopusdeploy-withkubectlandhelm .
```

Once created, [follow the standard instructions](https://octopus.com/docs/installation/octopus-server-linux-container/docker-compose-linux) on Octopus’ site, replacing the image name in the Docker Compose file with the custom container name. This:

```yaml
image: octopusdeploy/octopusdeploy:${OCTOPUS_SERVER_TAG}
```

Becomes:

```yaml
image: octopusdeploy-withkubectlandhelm
```