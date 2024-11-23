# Azure Kubernetes Service (AKS) and Flux – ii – Flux with Empty Repository

## Flux

Flux requires a hosted Git repository to function. This can be in Azure DevOps, GitHub, Bitbucket, etc…

The instructions below show how to setup an empty repository so can fully configure your cluster from scratch. There are also instructions for setting up a basic cluster featuring [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/), [Cert-Manager](https://cert-manager.io/), [Seq](https://datalust.co/seq) and a demo application by copying an existing Git repo. Be aware that, out of the box, Seq is open access.

### Repository Setup
This article will set up things using an empty repository. A future article (will be linked when available) will guide you through doing this with a pre-populated repo. This example uses Azure DevOps for a Git repository with a personal access token (PAT).

You’ll need to create a PAT so do this now. You’ll need “Full” code permissions for the PAT and, for ease, set expiry to be as long as possible. Longer term, other solutions will need to be used and not a PAT linked to a personal account. The default branch is called `main`.

#### Empty Repository

In Azure DevOps, create a new repository with the README enabled. For this demo, we’ll assume it’s called “Flux.Demo” Clone this repo into Visual Studio Code or another tool that’s good for editing YAML files.

You can maintain multiple clusters in one repo (AKA a monorepo) so the advice is to think about this when setting up your repository. The pattern I’ll be using is region\environment\cluster e.g. uksouth\production\01 but any logical pattern makes sense.

In VS Code, create a “clusters” folder and within it a “uksouth” folder and within that a “development” folder and within that an “01” folder. In the root, create a folder called “infrastructure” and within that create two folders: “flux-support” and “common”.

In the “flux-support” directory, create a file called “cluster-variables-configmap.yaml” and populate with the following:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-variables
  namespace: flux-system
data:
  cluster_region: '${cluster_region}'
  cluster_env: '${cluster_env}'
  cluster_number: '${cluster_number}'
  cluster_managedidentity_id: '${cluster_managedidentity_id}'
```

We’ll use this config map in other definitions when we want to know what region, environment, etc… we’re in. You can use tags on your infrastructure for some information but this will allow any value to be defined and, importantly, accessed within your Flux definitions, not just your running workloads.

In the “flux-support” folder, also create the a file called “kustomization.yaml”:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - cluster-variables-configmap.yaml
```

In the “common” folder, we’ll add four files which will create an NGINX Ingress Controller along with setting up Cert-Manager and allow us to deploy the demo app.

#### cert-manager.yaml
  
```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: cert-manager
  labels:
    toolkit.fluxcd.io/tenant: sre-team
---
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: HelmRepository
metadata:
  name: cert-manager
  namespace: cert-manager
spec:
  interval: 24h
  url: https://charts.jetstack.io
---
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: cert-manager
  namespace: cert-manager
spec:
  interval: 30m
  chart:
    spec:
      chart: cert-manager
      version: "1.x"
      sourceRef:
        kind: HelmRepository
        name: cert-manager
        namespace: cert-manager
      interval: 12h
  values:
    installCRDs: true
```

#### helmrepository-jabbermouth.yaml

```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: HelmRepository
metadata:
  name: jabbermouth
  namespace: flux-system
spec:
  interval: 5m0s
  url: oci://registry-1.docker.io/jabbermouth
  type: 'oci'
```

#### ingress-nginx.yaml

```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: ingress-nginx
  labels:
    toolkit.fluxcd.io/tenant: sre-team
---
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: HelmRepository
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
spec:
  interval: 24h
  url: https://kubernetes.github.io/ingress-nginx
---
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
spec:
  interval: 30m
  chart:
    spec:
      chart: ingress-nginx
      version: '*'
      sourceRef:
        kind: HelmRepository
        name: ingress-nginx
        namespace: ingress-nginx
      interval: 12h
  values:
    controller:
      service:
        annotations:
          service.beta.kubernetes.io/azure-load-balancer-health-probe-request-path: /healthz
```

#### kustomization.yaml

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - helmrepository-jabbermouth.yaml
  - cert-manager.yaml
  - ingress-nginx.yaml
```

Going back to the clusters\uksouth\development\01 folder, create a folder called “flux-system” and in it create two empty files called “gotk-components.yaml” and “gotk-sync.yaml”.

Back in the clusters\uksouth\development\01 folder, create a new file called “flux-support.yaml” and populate it with the following content:

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: flux-fluxsupport
  namespace: flux-system
spec:
  interval: 10m0s
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: ./infrastructure/flux-support
  prune: true
  wait: true
  timeout: 5m0s
  postBuild:
    substitute:
      cluster_region: 'uksouth'
      cluster_env: 'development'
      cluster_number: 'c01'
      cluster_managedidentity_id: ''
```

We’ll populate the cluster_managedidentity_id value later. Also note that cluster_number is prefixing the number with a “c” due to a quirk with how numbers are handled.

Next, create a file called “common.yaml” and populate it with the following content:

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: flux-common
  namespace: flux-system
spec:
  interval: 10m0s
  dependsOn:
    - name: flux-fluxsupport
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: ./infrastructure/common
  prune: true
  wait: true
  timeout: 5m0s
```

Note the dependsOn value which says this can not be run until the flux-support kustomization has completed.

### First Deployment – Bootstrapping the Cluster

You show now commit your changes to Git and push them to the remote server.

The next step is to make Flux aware of the repo. The following PowerShell script will do this. Note you will need to update the three “your-” values (your-git-pat, your-organisation and your-project) with your specific values.

```powershell
$REPO_TOKEN="your-git-pat"
$REPO_URL="https://dev.azure.com/your-organisation/your-project/_git/Flux.Demo"
$REGION = "uksouth",
$ENVIRONMENT = "development",
$CLUSTER = "01",
$BRANCH = "main"

flux bootstrap git `
  --token-auth=true `
  --password=$REPO_TOKEN `
  --url=$REPO_URL `
  --branch=$BRANCH `
  --path="clusters/$REGION/$ENVIRONMENT/$CLUSTER"
```

To view the state of your Flux deployment, you can run the following command:

```powershell
flux get kustomizations
```

To view this in watch mode, append `--watch` to the end of the command. Once all the rows show as ready and have an “Applied revision: main@sha1:…” then your cluster should be ready.

Next, run the following command to get the external IP of your ingress controller:

```powershell
kubectl get svc -n ingress-nginx
```

Note the external IP for later use. You can also do a simple test by going to http://<external-ip>/ and you should see a “404 Not Found” error page from Nginx.

#### Demo Application

In the cluster definition folder (clusters\uksouth\development\01), create a file called “applications.yaml” and populate it with the following contents:

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: flux-applications
  namespace: flux-system
spec:
  interval: 10m0s
  dependsOn:
    - name: flux-common
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: ./applications/demo
  prune: true
  wait: true
  timeout: 5m0s
  postBuild:
    substituteFrom:
      - kind: ConfigMap
        name: cluster-variables
```

Now, in the root, create a folder called “applications” and, in that folder, create a folder called “demo”. We’ll create two files in that folder:

### demo.yaml

```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: applications
---
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: demoapp
  namespace: applications
spec:
  interval: 5m
  chart:
    spec:
      chart: helm-demo
      version: '^1.0.0'
      sourceRef:
        kind: HelmRepository
        name: jabbermouth
        namespace: flux-system
      interval: 1m
  values:
    ingress:
      domain: demo.yourdomain.com
      path: /simple
      tls:
        enabled: true
        letsEncryptEmail: 'youremail@yourdomain.com'
    config:
      environment:
        overridden:
          createAs: 'inline'
          value: 'Common to all environments'
        onlyFromEnvVar:
          value: 'Demo application'

You will need to update the ingress section with your domain and email address.

Next, create the kustomization file in the applications\demo folder:

#### kustomization.yaml

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - demo.yaml
```

Before pushing your changes, create a DNS record that matches the domain entered above, using the external IP address you retrieved in the previous section.

Once the DNS record is configured, commit and push your changes to repo and monitor Flux as before:

flux get kustomizations
To force a reconciliation, you can run the following command:

```powershell
flux reconcile kustomization flux-system --with-source
```

If all worked as expected, going to the equivalent of [https://demo.yourdomain.com/simple](https://demo.yourdomain.com/simple) should show a simple demo site with a valid “SSL” certificate.