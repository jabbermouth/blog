---
title: Changing branch used by Flux deployment
description: If the need arises to change the branch used by a Flux installation, this can be done without bootstrapping the cluster again. Not that this is recommended only on similar setups (e.g. you’re trying out a new change on a dev cluster and want to point to your dev branch which is based on the default branch).
date: 2023-09-26
categories:
  - Kubernetes
  - Flux
  - Quick Tips
---
# Changing branch used by Flux deployment

If the need arises to change the branch used by a Flux installation, this can be done without bootstrapping the cluster again. Not that this is recommended only on similar setups (e.g. you’re trying out a new change on a dev cluster and want to point to your dev branch which is based on the default branch).

## Components

This article assumes the following components are in play:

- Kubernetes cluster called `cluster01`
- Flux mono repo with the cluster definition in the locations `clusters/development` (this folder contains the `flux-system` folder)
- A default branch called `main`
- A working branch called `12345-something-to-test` that is based on `main` with a new change in it

## Process

1. Switch to your working branch
2. Run the following command at a shell prompt (Bash, PowerShell, etc…) when the current context is you the target cluster: `flux suspend source git flux-system`
3. Update `clusters/development/flux-system/gotk-sync.yaml` to set the value of branch to `12345-something-to-test` and commit and push it
4. Within your Kubernetes cluster, update the resource of type `GitRepository` named `flux-system` in the `flux-system` namespace so the branch field is also `12345-something-to-test`
5. Run the following command at a shell prompt (Bash, PowerShell, etc…) when the current context is you the target cluster: `flux resume source git flux-system`

Once testing is complete, repeat the above process but setting the branch back to `main`.