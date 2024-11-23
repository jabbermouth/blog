# Pushing Helm Charts to a Container Registry

This article walks though building and then pushing a Helm chart to a container registry locally, Azure Container Registry (ACR) and Docker Hub.

## Getting Local Container Registry Running

The easiest way to achieve this is using Docker. Once Docker is installed, running the following command will setup an auto-restarting on boot container.

```powershell
docker run --name registry --env=PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin --volume=/var/lib/registry -p 8880:5000 --restart=always -d registry:2
```

Once up and running, the list of charts can be accessed at [http://localhost:8880/v2/_catalog](http://localhost:8880/v2/_catalog) although this will be blank when initially accessing it.

## Building A Package

Building a package is the same as building for any Chart registry such as Chart Museum. The important thing is that the chart name is all lowercase. The recommended convention is lowercase, numbers and dashes, replacing other characters and spaces with hyphens e.g. `MyShop.Checkout` would become `myshop-checkout`. This value should be used in the `name` property of the `Chart.yaml` file, resulting in something similar to the following first few lines:

```yaml
apiVersion: v2
name: myshop-checkout
version: 0.0.1-dev
```

### Versioning

Most systems will only pull a new version of a chart if the version number has increased. A script to automate version numbering is advised but, for the purposes of this tutorial, the fixed value will be used.

### Building Package

Assuming the above example chart is in a subfolder called MyShop.Checkout, run the following command to build a chart package called `myshop-checkout-0.0.1-dev.tgz` in the current folder:

```powershell
helm package ./MyShop.Checkout
```

## Pushing Package to Local Registry

Pushing to the local registry is straightforward as authentication is not required by default. Run the following command to add the above chart to the local registry:

```powershell
helm push myshop-checkout-0.0.1-dev.tgz oci://localhost:8880/helm
```

This can then be checked by going to [http://localhost:8880/v2/_catalog](http://localhost:8880/v2/_catalog).

## Pushing To Azure Container Registry (ACR)

The following PowerShell script will push the above example chart to an ACR. Subscription names and target registry will need updating per your settings.

```powershell
$subscription = "Your Subscription Name"
$containerRegistry = "yourcontainerregistry"
$ociUrl = "$containerRegistry.azurecr.io/helm"

$currentAccount = (az account show | ConvertFrom-Json)
if (-not $currentAccount) {
    Write-Host "Attemping login..."

    az login
    az account set --subscription $subscription
} else {
    Write-Host "Logged in already as $($currentAccount.user.name)"
}

$helmUsername = "00000000-0000-0000-0000-000000000000"
$helmPassword = $(az acr login --name $containerRegistry --expose-token --output tsv --query accessToken)

Write-Output $helmPassword | helm registry login "$containerRegistry.azurecr.io" --username $helmUsername --password-stdin
}

helm push myshop-checkout-0.0.1-dev.tgz oci://$ociUrl
```

## Pushing To Docker Hub

Firstly, if one doesn’t exist already, create a PAT with read/write permissions. Also, for any repositories that are to be deployed, create these if using a PAT without admin permissions. For example, if the repository will be called `myshop-checkout`, create a repository called `myshop-checkout`.

```powershell
# login
$helmUsername = "docker-name"
$helmPassword = "your_docker_pat"

Write-Output $helmPassword | helm registry login "registry-1.docker.io" --username $helmUsername --password-stdin

$ociUrl = "registry-1.docker.io/$helmUsername"

helm push myshop-checkout-0.0.1-dev.tgz oci://$ociUrl
```