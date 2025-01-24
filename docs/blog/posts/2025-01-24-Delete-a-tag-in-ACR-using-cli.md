---
title: Delete a tag in ACR using CLI
description: How to delete a tag from an Azure Container Registry repository using the Azure CLI
date: 2025-01-05
categories:
  - Azure Container Registry
  - Azure
  - Quick Tips
---
# Delete a tag in ACR repository using CLI

If you have a tag in an ACR repository that you no longer need, you can delete it using the Azure CLI.  It's a simple process and just requires remembering a few property names.

If you've not logged into Azure, do that first:
```powershell
az login
```

Then, list the repositories in the registry to find the tag you want to delete:
```powershell
az acr repository list --name acrname
```

Next list the tags for the repository you're interested in:
```powershell
az acr repository show-tags --name acrname --repository your.repository
```

Once you've found the repository and tag you want to delete, you can delete it:
```powershell
az acr repository delete --name acrname --image your.repository:1.2.3-alpha.456789
```

Obviously you'll need to replace `acrname`, `your.repository` and `your.repository:1.2.3-alpha.456789` with your own values.