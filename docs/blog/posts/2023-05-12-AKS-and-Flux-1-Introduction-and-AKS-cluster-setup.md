---
title: Azure Kubernetes Service (AKS) and Flux – i – Introduction and AKS cluster setup
description: Contents currently unavailable.
date: 2023-05-12
categories:
  - Azure
  - Kubernetes
  - Flux
  - Tutorials
---
# Azure Kubernetes Service (AKS) and Flux – i – Introduction and AKS cluster setup

## Background

I wanted an [Azure Kubernetes Service (AKS)](https://azure.microsoft.com/en-us/products/kubernetes-service) cluster to run some tests against but also, ultimately, host some sites. I wanted an easy way to manage the contents of the cluster so decided to go with a GitOps workflow using a mix of [Helm](https://helm.sh/) and [Flux](https://fluxcd.io/). For the purposes of this walkthrough, make sure these are pre-installed along with Kubectl. You may also find a tool called [K9s](https://k9scli.io/) useful. If you’re using Windows, using [Chocolatey](https://chocolatey.org/install#individual) should make installing these packages easier.

The reason behind this choice was a combination of research done both in and out of my day job.

## Demo Applications

For the purposes of this article, any applications installed that aren’t generally available ones (e.g. NGINX Ingress Controller or Cert-Manager) will be my demo container and its [associated Helm chart](https://hub.docker.com/repository/docker/jabbermouth/helm-demo/general). This demo package offers multiple versions (all actually being the same image except for the version number) to test things like version ranges and, once up and running, the application can make use of other Azure services such as Azure Service Bus, Azure Key Vault and Azure App Configuration, all using managed identity.

## Goal

The goal is simple – get an AKS cluster up and running, set up reserved hosting (this will save about 64% on hosting costs a month) and get a basic demo up and running.

Be aware that following these instructions will likely cost you money unless you have free Azure credit.

## Walkthrough

This section is split into several subsections covering the various steps to set up an AKS cluster and some other Azure services and control access to them using a managed identity.

For the purposes of this demo, it will describe creating a cluster using the Azure Portal. Also, it’s not intended as a complete guide to using Azure so some steps will not be covered in detail.

This guide assumes you have an Azure Subscription. As I’m in the UK, all references to region will be UK South.

### Azure Cluster

Go to the Azure Portal then “Kubernetes services” and select the [“Create a Kubernetes cluster” option](https://portal.azure.com/#create/Microsoft.AKS).

Choose the subscription and resource group you wish to use. For the “Cluster preset configuration” choose “Cost-optimised ($)” (this will give you a Standard_B4ms node with 4 vCPUs and 16GB of memory). You can change this but a minimum of 8GB is recommended for this demo.

Next, enter a suitable cluster name and choose the region you want. For the Kubernetes version, choose the latest available version. For the scale method, set this to “Manual” and set the number of nodes to 1.

In a normal cluster, a minimum of two nodes is recommended for resilience reasons however, for the purposes of this demo and keeping costs down, a single node will suffice.

Next navigate to node pools and choose the “userpool” node pool and change the scale method to “Manual” and the number of nodes to 0.

Access can be left on the default settings for now. On network, change the “Network configuration” to “Azure CNI” and then choose a virtual network or create a new one. For the “DNS name prefix”, use the same name as the cluster, using lowercase only, numbers and hyphens for spaces, dots, etc…

On the “Advaned” config page, for the “Infrastructure resource group”, enter the same name used for the “DNS name prefix”

Once you get to the “Review + create” tab, check everything looks OK and click “Create”. After a short while, the cluster will be provisioned and you’ll receive a notification of this.

If you wish to create a more realistic cluster but still keep costs down, I’d recommend two Standard_B2s nodes for agentpool and two Standard_B4s nodes for user pool. Remember the Kubernetes is designed to scale horizonally so, in many situations, it’s better to add nodes rather than more CPU and memory to each node. For an “early days” cluster, a series of Standard_B8ms up to Standard_B16ms would be a reasonable choice.

### Reserved Pricing

Only do this if you wish to keep a machine running with the same machine SKU (i.e. spec) chosen above. It doesn’t have to be the same machine you keep but the reservation will be linked to the machine type.

Navigate to the [“Purchase reservations” section](https://portal.azure.com/#create/Microsoft.Reservations) of Azure and choose the “Virtual machine” option. Under recommended, you should see the virtual machine you set up above. Select it along with the term you want. Three years offers the best discounts