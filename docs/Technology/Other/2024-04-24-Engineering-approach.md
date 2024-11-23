# Engineering approach

When I work on a project, small or large, this is the approach I like to take to designing and building a solution. This is written from the perspective of a new solution but could be adapted for existing solutions too.

## High Level Design

The first place I start is a high level design. I don’t go into great detail or try to design a final system – I simply include the basic components I’ll need to get to something deliverable. This usually includes software components such as APIs (e.g. customers, orders, stock, etc…) and infrastructure (e.g. Kubernetes cluster, identity provider, secret storage, etc…). Sometimes specific technologies will be listed if these are known (e.g. if it’s an Azure house, then Azure Kubernetes Service (AKS), Entra ID and Azure Key Vault). This may be done as one diagram or two.

## Breaking Up Work

I then take this high level design and think about the features and stories it will produce. Again, at this stage, nothing too detailed but thinking more about delivery of “something” even if it’s of no use.

As an example, let’s say we know we will be using AKS to host our application and have decided we will do this setup rather than look at container apps as a first stage. We’ve also decided we’ll be using a GitOps workflow (e.g. Flux) for deployments and, to start with, we want a simple API which has some kind of API key based access management. The initial requirements to get this API hosted and accessible might be:

1. Set up repos for infra and API
2. Create infrastructure*
3. Create simple API
4. Configure Flux
5. Configure pipelines

<small>* This first iteration may not be as secure or hardened as would be preferred but it’s nothing more than a first step with no data at risk.</small>

### Set up repos for infra and API

If Infrastructure-as-Code (IaC) is being used to manage your repos, simple add the two new repo names and you’re done. If still doing manually, add the appropriate repos after agreeing a naming convention. This is what I class as a sprint 0 story as it needs doing to unblock work on one or more other stories without creating (too many…) dependencies in a sprint. If running with a Kanban approach, this would be the first story to tackle.

### Create infrastructure

Again, if an existing IaC setup is being used, this may be a very quick process. If not, infrastructure could be created manually or using something like Terraform. If time and skills permit, I’d opt for Terraform to ensure long-term consistency and it also makes it easier to iterate through changes as work progresses.

Remember that in this initial phase, only a single, small environment is required.

### Create simple API

This is referring to nothing more than an API with a suitable set of stub endpoints. Nothing needs to be real in terms of real CRUD activities, etc… If helper libraries exist, it’s recommended these be used to simplify setup and ensure consistency. For example, this helper library could set up things like logging, basic health checks or config file loading.

As a strong advocate of Test Driven Development (TDD), I recommend starting with a TDD approach from the beginning, even for this “hello world” API.

Services should be build independently as possible and as loosely coupled as possible so it’s recommended any config, even that which will come from things like Azure Key Vault, reference local files and let Kubernetes manage this dependency. For databases such as SQL Server or Azure Storage, local Docker versions can be used, not that any of these should be needed for this first cut of the app.

### Configure Flux

Flux is a GitOps tool and whilst isn’t strictly necessary for the first cut, without resorting to more complex pipelines and manual deployments, is a reasonable first step.

If you’re using a common and shared Helm chart for your services, that should significantly speed up the release of services as it will handle a lot of the boiler plate configuration and ensure consistency.

This initial Flux setup may want to include nothing but the ability to deploy an application using a Helm chart or could include broader requirements such as ingress. As things such as port forwarding can be used at these early stages, this may be required.

### Pipelines

Hopefully a set of pipelines already exist in the form of templates that can be called with minimal parameters but, if not, this first pipeline should only do the basics of building and pushing an image to a container registry. If using Azure, Azure Container Registry (ACR) would be recommended for easier authentication.

It’s also highly recommended that a pull request pipeline be created at this time. Even when using trunk-based development, the process of going through a PR pipeline can ensure all tests are passing and security scans are successfully completed.

### Potential Stories

This requirements above should lead to specific work items. These should be as small as possible (i.e. completable in no more than one day) whilst still delivering “something of value”. For example, say setting up the Terraform for new infrastructure will take 2-3 days based on previous experience, a single story for all this work would be undesirable. A better breakdown would be:

1. Provision global resource group with ACR instance – this could include setting up the repo and storage account to hold the state
2. Provision suitable Entra groups and grant appropriate access to ACR
3. Provision AKS cluster and needed VNET
4. Provision a service principal for use with pipelines
5. Create pull request pipeline which runs a terraform plan and is called from the infrastructure repo
6. Create a deploy pipeline which runs a terraform apply and is called on any merging to main
7. Setup a repo for the Flux configuration and bootstrap the cluster using Terraform

Notice that this list encompasses parts of several of the initial high level tasks identified. At this very early stage, some dependencies on previous stories is to be expected.

It’s likely that existing resources (e.g. pipelines, code templates or libraries, etc…) exist which will make some of these tasks quite small.

## Methodology

Before you start “doing stuff”, the work management approach needs to be considered. An agile approach would be an obvious in today’s world with Kanban and Scrum being the two more common approaches taken.

Scrum usually consists of work blocks (known as sprints) of 1 to 4 weeks with 2 weeks being the most common. Each sprint has a goal and collection of work to achieve that goal with some that “must” be completed, some that “should” be completed, some that “could” be completed if all goes to plan and some that “won’t” be completed unless work is completed much quicker than expected. This is called MoSCoW prioritising and is a typical method used. You’ll often see 1 to 4 used instead of MoSCoW but they often translate to the same basic understandings.

Kanban is more like a continuous list of work with the queue regularly reviewed, resulting in work being added, reprioritised and potentially removed.

Either approach supports trunk-based development with regular releases i.e. there’s no need to wait until the end of a sprint to do a release.

### Story Writing

Once the approach has been agreed, the team (engineers, product owner, etc…) should agree on what stories are needed, write them, refine them and agree on a priority. Once enough work has been prepared for the first sprint or first week or two for Kanban, work can commence.

As time goes on, it’s good to have one or two sprints ahead (1-2 months of work) planned out. Things can always be reprioritised but this helps begin to see a bigger picture and set time expectations.

## The Components
Once the first batch of stories have been written, work can commence. For the various areas of engineering, I endeavour to adhere to the following:

### Infrastructure-as-Code (IaC)

All infrastructure should be done as IaC to ensure consistency and repeatability. My tool of choice is Terraform and whilst this tool isn’t perfect, it’s the most popular tool available and, as such, has a large amount of support available and works with many providers including Azure, AWS, GitHub, SonarCloud, etc…

If something is to be tested by manual changes, it is recommended that these changes are made in a separate area (e.g. subscription) to the infrastructure managed by IaC.

### Code

All code should be written using a Test Driven Development (TDD) approach with tests testing requirements (i.e. inputs and outputs) rather than the inner workings of a method. Ultimately a series of unit, integration and end-to-end tests should be developed and executed as part of release pipelines.

Any code committed to `main` must be releasable to production. This doesn’t necessarily mean it can be used on production – a feature gate could be keeping it hidden – but the code should be safe to release to production.

The use of tools for static code analysis and Snyk can help scan code and containers for code or security issues ahead of any code being released to a server.

Logging, observability and monitoring are essential to keeping a system healthy and diagnosing problems when they arise. Suitable tooling should be in place such as Prometheus and Grafana or Datadog should be in place as early as possible in the development stage. The use of OpenTelemetry is strongly encouraged to enable easier migration to different tooling.

The observance of things like SOLID, DRY, KISS, etc… are always encouraged.

### APIs and Microservices

When building microservices and APIs, I align to the following rules:

1. Communication through interfaces – any communication, whether REST, gRPC or class, will communicate through interfaces and, for message-based/event driven applications, . agreed schema
2. Any service must only communicate to another service or the data belonging to another service through the above interfaces – no service A looking at service B’s data . directly
3. Any service must be independently deployable
4. Any service should gracefully handle any unavailability of an external service
5. Any API must be built to be externalisable (i.e. can be exposed to the public internet)
6. Within a major version of an API, any changes should be non-breaking – when breaking changes are needed, a new major version should be created and the previous version(s) maintained

### Separation of Concerns

It is strongly recommended that, where possible, things such as authentication, configuration, credentials, etc… are managed independently of the code. In other words, an application running locally should use local configuration files to store non-sensitive configuration (e.g. a connection to a local SQL Server instance is OK but nothing that is hosted). If sensitive values must be stored, use things like User Secrets as these aren’t part of the repo and therefore not stored in source control.

For loading sensitive values, where possible use managed identities (or equivalents) and, where these aren’t possible, try to use (service) accounts that are machine created (likely with IaC) and their credentials stored in things like Key Vault and never actually accessed/known by humans. These credentials along with other configuration values can then be made available to applications (i.e. pods) in Kubernetes using tools like [External Secrets Operator](https://external-secrets.io/) and [Azure App Configuration Provider](https://learn.microsoft.com/en-us/azure/azure-app-configuration/quickstart-azure-kubernetes-service#use-app-configuration-kubernetes-provider) and mounting them as files into the pod. Equally, storage can be mounted in a similar manner.

## Putting It All Together

Work should be selected from the top of the queue with a goal to complete active stories (or bugs once they come in) before picking up new ones. In other words, pull requests should be given priority and then, before picking up a new story, see if you can potentially assist on an already active story.

### When is it done?

It’s important to have a clear definition of done so that all parties agree what this means. As a minimum, it should mean a story has successfully completed a pull request to main but arguably it should mean, as a minimum, QA (either automated or manual) has been completed. I would suggest something is only “done” once it has been successfully deployed to production and confirmed to be working.