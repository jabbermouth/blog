---
title: Running an Azure DevOps Build Agent in Kubernetes
description: This article talks through the process of setting up an Azure DevOps build agent in Kubernetes using a Helm chart.
date: 2022-11-12
categories:
  - Azure DevOps
  - Kubernetes
  - Helm
  - Tutorials
---
# Running an Azure DevOps Build Agent in Kubernetes

## The basics…

Firstly, create a new namespace called `build-agents` to hold the new agents and also create a PAT in Azure DevOps, ideally associated with a service principal.

```bash
kubectl create ns build-agents
```

If desired, add details about your Docker account if you wish to use your account for unlimited pulls.

```bash
kubectl create secret docker-registry dockerhub-account --docker-username=dockerhub-account-name --docker-password=dockerhub-account-password --docker-email= --namespace build-agents
```

In the above script, `dockerhub-account` will be the name of the secret in Kubernetes, `dockerhub-account-name` should be changed to your Docker Hub account name and `dockerhub-account-password` should be changed to your Docker Hub password or token.

Next create a simple YAML file with the following values:

## buildagent-settings.yaml

```yaml
image:
  pullPolicy: Always
  pullSecrets:
    - dockerhub-account

personalAccessToken: "your-ads-pat"

agent:
  organisationUrl: "https://dev.azure.com/your-org-name/"
  pool: "Default"
  dockerNodes: true
```

In the above file, you should change `your-ads-pat` to a Personal Access Token (PAT) you’ve generated which has “Read & manage” agent permissions. You should also change `your-org-name` to the name of your Azure DevOps organisation. If you wish to deploy the agent to a pool other than the "Default" one, you should also update `pool`. The `image` section can be completely removed or either of the sub-values if desired. If you’re not running Docker for your container hosting (Kubernetes 1.24+ will no longer use Docker as its container engine, using Containerd instead), set dockerNodes to false.

Once you are done, it’s time to install the first agent:

```bash
helm repo add jabbermouth https://helm.jabbermouth.co.uk
helm install -n build-agents -f buildagent-settings.yaml linux-agent-1 jabbermouth/AzureDevOpsBuildAgent
```

To add an additional build agent, simply run the following:

```bash
helm install -n build-agents -f buildagent-settings.yaml linux-agent-2 jabbermouth/AzureDevOpsBuildAgent
```

The container can be found on [Docker Hub](https://hub.docker.com/repository/docker/jabbermouth/ads-linux-agent), the files used to build the container can be [found on GitHub](https://github.com/jabbermouth/ads-linux-agent) and the Helm chart code can also be [found on GitHub](https://github.com/jabbermouth/azure-devops-build-agent-helm).