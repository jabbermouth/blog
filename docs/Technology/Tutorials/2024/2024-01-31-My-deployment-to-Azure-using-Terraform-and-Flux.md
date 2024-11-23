# My deployment to Azure using Terraform and Flux

To deploy my needed infrastructure and applications to Azure, I use a combination of Terraform and Flux, all running from GitHub Actions workflows. This is only a high level overview and some details have been excluded or skimmed over for brevity.

## Terraform

For me, one of the biggest limitations of Terraform is how bad it is at DRY (i.e. Don’t Repeat Yourself). I wanted to maintain a pure Terraform solution whilst trying to minimise the repetition but also allow the easy spinning up of new clusters as needed. I also knew I needed a “global” environment as well as various deployment environments but, for now, development and production will suffice.

### Modules

Each Terraform modules is in it’s own repo with a gitinfo.txt file specifying the version of the module. On merge to main, a pipeline runs which tags the commit with Major, Major.Minor and Major.Minor.Patch tags so that the modules can be specifically pinned or, if desired, a broader level of pinning can be used.

#### Folder Structure

Each module contains a `src` folder which contains an `examples` folder for Terraform examples of the modules being used and a `module` folder which contains the actual module. This will be then referenced by the scope repos.

### Scopes

Three scope repos are in use – Environment, Global and GitHub – which make use of the above modules and standard Terraform resources.

Global and GitHub are single use scopes and are only utilised once and configure all global resources (e.g. DNS zones) and all GitHub repositories. The latter holds configuration for all repositories in GitHub including it’s own. The initial application of this, to get round the chicken and egg problem, will be covered in another article.

The environment scope is used to configure all environments and within each environment scope, there are regions and within them are farms and within them are clusters. This allows for common resources to be placed at the appropriate level.

#### Folder Structure

Each scope contains a `src` folder which contains three sub folders:

- `collection` – This contains the resources to be applied to the scope
- `config` – This contains one or more `tfvars` files which, especially in the case of environments, contain the different configuration to be applied to each environment
- scope – This is the starting point of a scope and it what is referenced by the plan and apply pipelines – it should only contain a single reference to the module in `../collection`

### Data Between Scopes

To pass data between scopes, the pipelines create secrets within GitHub which can then be referenced by other pipelines and passed to the target scope. The main example of this is the global scope needs to pass data, e.g. DNS details, to the environments.

### Variables in Environment Scope

To aid with data passing between layers within the environment, a few variables have been set up which get passed through the layers and can have data added as they pass down the chain. These variables are `defaults`, `created_infrastructure` and `credentials`. As an example, when an environment level key vault is created, it’s ID and name are added to `created_infrastructure` so that access can be set up for the AKS cluster using a managed identity.

## Flux

### Bootstrapping

When a cluster is provisioned, it is automatically given a cluster folder in the Flux repo and has a `cluster_variables` secret created to store values that either Flux or Helm charts being applied by Flux may need including things like the region or global key vault’s name.

### Folder Structure
The top level folder is the clusters folder and it contains a `_template` folder along with a folder per environment. The individual environment folders then contain a folder for the region, the farm, and finally the cluster (e.g. `development/uksouth/f01/c01`). The c01 folder is based on the `_template` folder.

The remaining folders are `applications`, `clients`, `infrastructure`, `redirects` and `services` with each of these being referenced from the c01 folder.

The `infrastructure` folder contains setup manifests and Helm releases for things like Istio, External DNS or External Secrets Operator.

The `redirects` folder is split up by environment and defines any redirects which should be added for that environment. These are managed using a redirects Helm chart and a series of values passed to its HelmRelease object.

The `services` folder allows you to define services which should be deployed at the various levels of global, environmental, regional, farm or cluster level. There is a definitions folder containing the base definition for each service.

The `applications` folder defines applications which should be deployed to specific clusters and, as with services, there is a definitions folder which contains the default configuration. These are generally non-targeted applications such as a commercial landing page.

The final folder is `clients` which contains a definition for any client applications. It’s quite likely this folder may only contain a single definition if only a single SaaS application (note this is single application, not single microservice) exists. There are then the usual nested environment, region, farm and cluster folders with each cluster defining the clients that are deployed to that specific instance.