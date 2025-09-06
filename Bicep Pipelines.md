### Pipelines

*  Azure Pipelines is a feature of the Azure DevOps service.
*  A pipeline is the repeatable process that you use to test and deploy your code defined in a configuration file.
*  We define pipeline in a YAML file.
*  The Azure DevOps web interface also provides some tools that you can use to view and edit your pipeline YAML files.
*  Pipeline YAML file is stored with your Bicep code in your Git repository.
---

### Agents and pools

* Azure Pipelines uses a machine called an agent. 
* An agent is a computer that's configured to run deployment steps for a pipeline.
* Each agent already has the Bicep and Azure tooling you used in earlier modules, so it can do the same things you do from your own computer.
* Instead of a human executing commands, the Azure Pipelines service instructs the agent to run the steps that you've defined in a YAML file.


- Azure Pipelines provides multiple types of agents for different operating systems, like Ubuntu or Windows, and different sets of tools. 
- Microsoft runs these agents, so you don't have to maintain any compute infrastructure for the agents. 
- The agents sometimes are called **Microsoft-hosted agents** or **hosted agents** because they're hosted on your behalf. 
- When your pipeline runs, a hosted agent is automatically created. When your pipeline is finished running, the hosted agent is automatically deleted. 
- You can't access hosted agents directly, so it's important that your pipeline contains all the steps necessary to deploy your solution.

---

### Agent Pool

* An agent pool contains multiple agents of the same type.
* When you create your pipeline, you tell Azure Pipelines which agent pool to use to execute each set of steps. 
* When your pipeline runs, it waits for an agent from the pool to become available, and then it instructs the agent to run your deployment steps. 
* Any agent in the pool might be assigned to run your pipeline.

---

### Triggers

* To instruct Azure Pipelines when to run your pipeline, you create a trigger. 
* You can choose from multiple types of triggers.

---

### Steps

* A step represents a single operation that the pipeline performs. 
* A step is similar to an individual command that you run in Bash or PowerShell. 
* You define the sequence and all the details of each step in your pipeline YAML file.

**Azure Pipelines offers two types of steps:**

**1. Scripts:** Use a script step to run a single command or a sequence of commands in Bash, PowerShell, or the Windows command shell.
**2. Tasks:** A task is a convenient way to access many different capabilities without writing script statements. 
  * For example, a built-in task can run the Azure CLI and Azure PowerShell cmdlets to test your code or upload files to an FTP server. 
  * Anyone can write a task and share it with other users by publishing the task in the Visual Studio Marketplace. 
  * A large set of commercial and open-source tasks are available.

* Script statements offer more control over what's executed.
* Tasks don't have to write and manage scripts.

---

### Jobs

* A Job represents an ordered set of steps.
* You can set each job to run on a different agent pool. 
* Running jobs on different agent pools is useful when you build and deploy solutions that need to use different operating systems in different parts of the job pipeline.


- You also can use stages in Azure Pipelines to divide your pipeline into logical phases and add manual checks at various points in your pipeline's execution.

---

### Basic pipeline example

```YAML
trigger: none

pool:
  vmImage: ubuntu-latest

jobs:
- job:
  steps:
  - script: echo Hello, world!
    displayName: 'Run a one-line script'
  
  - script: |
      echo We'll add more steps soon.
      echo For example, we'll add our Bicep deployment step.
    displayName: 'Run a multi-line script'
```

**Let's look at each part of the file in detail:**

* `trigger` tells your pipeline when to execute. In this case, `trigger: none` tells Azure Pipelines that you want to manually trigger the pipeline.
* `pool` instructs the pipeline which agent pool to use when it runs the pipeline steps. In this example, the pipeline runs on an agent running the Ubuntu operating system, which comes from the pool of Microsoft-hosted agents.
* `jobs` groups together all the jobs in your pipeline.
* `job` tells your pipeline that you have a single job.
* `steps` lists the sequence of actions to run in the job. The example YAML includes two steps. Both steps run a simple script to echo some text. Each step has a `displayName` value, which is a human-readable name for the step. You'll see the display name when you look at the pipeline logs. To create a multi-line script step, use the pipe character (`|`) as shown in the example. After your step executes, you'll see the outputs in the pipeline log.


---
### Deploy Bicep files by using a pipeline

### Service connections

* Deployment by pipeline requires authentication. 
* Pipelines authenticate to Azure by using a service principal. 
* A service principal's credentials consist of an application ID and a secret, which usually is a key or a certificate. 
* In Azure Pipelines, you use a service connection to securely store these credentials so that your pipeline can use them. A service connection also includes some other information to help your pipeline identify the Azure environment that you want to deploy to.
* we'll use Azure DevOps to automatically create a service principal when it creates a service connection.
* When you create a service connection, you **name** the connection.
* Steps refer to the service connection by using this name, so your pipeline YAML code doesn't need to contain secret information.
* When your pipeline starts, the agent that's running your deployment steps has access to the service connection, including its credentials.
* A pipeline step uses the credentials to sign in to Azure, just like you sign in yourself. Then, the actions that are defined in the step use the service principal's identity.


- You must ensure that your service principal has the permissions it needs to execute your deployment steps. 
- For example, you might need to assign the service principal the Contributor role for the resource group it deploys your resources to.

### For multiple Connections

* Service connections are created in your Azure DevOps project. 
* Multiple pipelines can share a single service connection. 
* However, it's usually a good idea to set up a service connection and the corresponding service principal for each pipeline and each environment you deploy to. 
* This practice helps increase the security of your pipelines, and it reduces the likelihood of accidentally deploying or configuring resources in a different environment than the one you expect.

- You also can set up your service connection so that it can be used only in specific pipelines. 
- For example, when you create a service connection that deploys to your website's production environment, it's a good idea to ensure that only your website's pipeline can use this service connection. 
- Restricting a service connection to specific pipelines stops someone else from accidentally using the same service connection for a different project and potentially causing your production website to go down.

---

### Deploy a Bicep file by using the Azure Resource Group Deployment task

* When you need to deploy a Bicep file from a pipeline, you can use the `Azure Resource Group Deployment` task. Here's an example of how to configure a step to use the task:

```YAML
- task: AzureResourceManagerTemplateDeployment@3
  inputs:
    connectedServiceName: 'MyServiceConnection'
    location: 'westus3'
    resourceGroupName: Example
    csmFile: deploy/main.bicep
    overrideParameters: >
        -parameterName parameterValue
```

The first line specifies `AzureResourceManagerTemplateDeployment@3`. It tells Azure Pipelines that the task you want to use for this step is named `AzureResourceManagerTemplateDeployment`, and you want to use version 3 of the task.

When you use the Azure Resource Group Deployment task, you specify inputs to tell the task what to do. Here are some inputs you might specify when you use the task:

* `connectedServiceName` is the name of the service connection to use.
* `location` needs to be specified even though its value might not be used. The Azure Resource Group Deployment task can also create a resource group for you, and if it does, it needs to know the Azure region in which to create the resource group. In this module, you'll specify the `location` input value, but its value isn't used.
* `resourceGroupName` specifies the name of the resource group that the Bicep file should be deployed to.
* `overrideParameters` contains any parameter values you want to pass into your Bicep file at deployment time.

When the task starts, it uses the service connection to sign in to Azure. By the time the task runs the deployment you specified, the task has authenticated. You don't need to run `az login`.

### Run Azure CLI and Azure PowerShell commands

* Two of the most useful built-in tasks in Azure Pipelines are the Azure CLI and Azure PowerShell tasks. 
* You can use these tasks to execute one or more Azure CLI or PowerShell commands.

### Variables

* Use **Variables** to store secrets or values that are different for different deployements and use them with YAML files in pipelines.

### Craete a Variable

* The Azure Pipelines web interface has an editor you can use to create `variables` for your pipeline: 
* users can override a variable value when they run your pipeline manually.

### Use a variable in your pipeline

After you create a variable, you'll use a specific syntax to refer to the variable in your pipeline's YAML file:

```YAML
- task: AzureResourceManagerTemplateDeployment@3
  inputs:
    connectedServiceName: $(ServiceConnectionName) #Variable
    location: $(DeploymentDefaultLocation) #Variable
    resourceGroupName: $(ResourceGroupName) #Variable
    csmFile: deploy/main.bicep
    overrideParameters: >
      -environmentType $(EnvironmentType) # variable
```

---

### System variables

Azure Pipelines also uses system `variables`. System variables contain predefined information you might want to use in your pipeline. Here are some of the system variables you can use in your pipeline:
  * `Build.BuildNumber` is the unique identifier for your pipeline run. Despite its name, the `Build.BuildNumber` value often is a string, and not a number. You might use this variable to name your Azure deployment so you can track the deployment back to the specific pipeline run that triggered it.
  * `Agent.BuildDirectory` is the path on your agent machine's file system where your pipeline run's files are stored. This information can be useful when you want to reference files on the build agent.

---

### Create variables in your pipeline's YAML file

* Use this option when you have values that aren't secret.

```YAML
trigger: none

pool:
  vmImage: ubuntu-latest

variables:
  ServiceConnectionName: 'MyServiceConnection' # Variable
  EnvironmentType: 'Test' # Variable
  ResourceGroupName: 'MyResourceGroup' # Variable
  DeploymentDefaultLocation: 'westus3' # Variable

jobs:
- job:
  steps:
  - task: AzureResourceManagerTemplateDeployment@3
    inputs:
      connectedServiceName: $(ServiceConnectionName)
      location: $(DeploymentDefaultLocation)
      resourceGroupName: $(ResourceGroupName)
      csmFile: deploy/main.bicep
      overrideParameters: >
        -environmentType $(EnvironmentType)
```

---
### Use triggers to control when your pipeline runs

### What is a pipeline trigger?

A **pipeline trigger** is a condition that, when met, automatically runs your pipeline based on rules you create.
    * You can set triggers to run your pipeline at scheduled intervals. 
    * You can also set triggers to run your pipeline every time a file in your repository changes. 
 
### Branch triggers

A common type of trigger is a `branch trigger`, also called a `continuous integration trigger` or `CI trigger`.
  * When you use a branch trigger, every time you make a change to a specific branch, the pipeline runs.
  * It's common to use this type of trigger against your default or `main` branch, with this code:

```YAML
trigger:
- main
```

### Multiple branches change Trigger

You can set up triggers to run your pipeline on a specific branch or on sets of branches.
  * you can trigger for a specific release of your project like release/v1, release/v2.
  * You want to run your pipeline anytime your code changes on a branch that begins with the name release/. You can use the `include` property with a `*` wildcard:

```YAML
trigger:
  branches:
    include: # for specific branches
    - main # for specific branches
    - release/* # for specific branches
```

### Exclude branches

By using the exclude property, you ensure that the pipeline isn't automatically triggered for changes to selected branches:

```YAML
trigger:
  branches:
    include:
    - '*'
    exclude:
    - feature/*
```
---
### Path filters

* you might have a deploy folder in your repository that contains your `Bicep code` and a separate `docs` folder that contains your documentation files. 
* You want to trigger your pipeline when anyone makes a change to any of the `Bicep` files in the deploy folder.
* But you don't want to trigger the pipeline if someone changes only a `docs` folder. 
* To set up a trigger to respond to changes in a specific folder in your repository, you can use a path filter:

```YAML
trigger:
  branches:
    include:
    - main
  paths: # here
    exclude: # here
    - docs
    include: # here
    - deploy
```

* Now If someone commits a change that updates only a documentation file, the pipeline doesn't run. 
* But if someone changes a Bicep file, or even if they change a Bicep file in addition to a documentation file, the trigger runs the pipeline.
---

### Schedule your pipeline to run automatically

**You can run your pipeline on a set schedule and not in response to a file change.**

* Use the `schedules` keyword instead of `trigger`, and set the frequency by using a cron expression:

```YAML
schedules:
- cron: "0 0 * * *" # here
  displayName: Daily environment restore
  branches:
    include:
    - main
```

**A cron expression is a specially formatted sequence of characters that sets how often an event will happen. In this example, `0 0 * * *` means run every day at midnight UTC.**
* You can also set which branch of your repository to use in the scheduled event. When the pipeline starts, it uses the most recent version of the code from the branch you set in the schedule.

---

### Use multiple triggers

* You can combine triggers and schedules, like in this example:
* The pipeline runs every day at midnight UTC and also whenever a change is pushed to the main branch.

```YAML
trigger:
- main

schedules:
- cron: "0 0 * * *"
  displayName: Deploy test environment
  branches:
    include:
    - main
```
---

### Concurrency control

* In some situations, having multiple concurrent runs of your deployment pipelines, it can be challenging to ensure that your pipeline runs aren't overwriting your Azure resources or configuration in ways that you don't expect.
* To avoid these problems, you can use the `batch` keyword with a trigger, like in this example:

```YAML
trigger:
  batch: true # Here
  branches:
    include:
    - main
```
When your trigger fires, Azure Pipelines ensures that it waits for any active pipeline run to complete. Then, it starts a new run with all of the changes that have accumulated since the last run.

---
---

### Authenticate your Azure deployment pipeline by using service principals

* Service principals provide a way to authenticate pipelines, applications, and software.

### Types of security principals

**Microsoft Entra ID is the service that manages identities for Azure. Microsoft Entra ID has multiple types of identities, which are also called security principals:**
  * A `user` represents a human who usually signs in interactively by using a browser. Users often have additional security checks to perform when they sign in, such as multifactor authentication (MFA) and Conditional Access based on their location or network.
  * A `group` represents a collection of users. Groups don't authenticate directly, but they provide a convenient way to assign permissions to a set of users together.
  * A `service principal` represents an automated process or system that usually doesn't have a human directly running it.
  * A `managed identity` is a special type of service principal that's designed for situations where a human isn't involved in the authentication process.
---
### Service principals

* It can sign in to Microsoft Entra ID, but there's no human to sign in and interact with the authentication process. 
* In Microsoft Entra ID, an `application ID` and a `credential` identifies a service principal. 
* The application ID is a globally unique ID (GUID). 
* For pipelines, the credential is usually a strong password called a `key`. Alternatively, you can use a `certificate` as a credential.

---

### Managed identities

* In contrast to the other types of service principals, a managed identity doesn't require that you know or maintain its credentials. 
* **A managed identity is associated with an Azure resource.** 
* Azure manages the credentials automatically. 
* They're a great way for Azure resources to authenticate themselves for situations like automating your Azure management, connecting to databases, and reading secret data from Azure Key Vault.


* When you work with pipelines, you usually can't use managed identities. 
* This is because managed identities require that you own and manage the Azure resources that run your deployments. 
* When you work with Azure Pipelines, you usually rely on shared infrastructure provided by Microsoft.

---
* There are some situations where pipelines can use managed identities. 
* In Azure Pipelines, you can create a self-hosted agent to run your pipeline's scripts and code by using on your own Azure-based virtual machine. 
* Because you own the virtual machine, you can assign it a managed identity and use it from your pipeline.

* However, most of the time your pipelines run by using a hosted agent, which is a server that Microsoft manages. Hosted agents aren't currently compatible with managed identities.
* In other parts of your solution, if you have a choice between using a managed identity or using a normal service principal, it's best to go with a managed identity. They're easier to work with and are usually more secure.
---

### Why can't you just use your user account?

* User accounts always need someone actively managing them.
---

### How do service principals work?

* Service principals are a feature of Microsoft Entra ID. 
* Microsoft Entra ID is a global identity service. 
* Many companies use Microsoft Entra ID, and each company is called a `tenant`.
  
  >>> Application
* Microsoft Entra ID has a concept of an `application`, which represents a system, piece of software, process, or some other nonhuman agent. 
* You can think of a `deployment pipeline` as an application.
* When you create an `application` and tell Microsoft Entra ID about it, you create an `object` called an `application registration`. 
* An application registration **represents** the application in Microsoft Entra ID.
  
  >>> Service Principals
* `Service principals` and `applications` are tightly linked. 
* Whenever an application registration is added to a `Microsoft Entra tenant`, a `service principal object` is created in that Microsoft Entra tenant. 
* When you create a service principal, most of the tools that you use also create an application registration at the same time. So you might not notice that there are two different objects.

---

### Create a service principal and key

### Understand how service principals are authenticated

* When a service principal needs to communicate with Azure, it signs in to Microsoft Entra ID. 
* After Microsoft Entra ID verifies the service principal's identity, it issues a `token` that the client application stores and uses when it makes any requests to Azure.
* Service principals use two main credentials: 
  - Keys
  - Certificates

---

### Keys

* Keys are similar to `passwords`. 
* However, keys are much longer and more complex. 
* In fact, for most situations, Microsoft Entra ID generates keys itself to ensure that:
  - The keys are cryptographically random. That is, they're extremely hard to guess.
  - Humans don't accidentally use weak passwords as keys.
* you only need to handle the key briefly when first configuring the `service principal` and your `pipeline`.
* A single service principal can have `multiple keys` at the same time, but users can't have multiple passwords. 
* Like passwords, keys have an `expiration date`.
---
### Certificates

* Certificates are another way to authenticate service principals. 
* They're very secure but can be hard to manage.
* certificates are harder for attackers to steal. 
* It's also harder to intercept and modify requests that use certificates. 
* However, certificates require more infrastructure and have some ongoing maintenance overhead.

---

### Work with keys for service principals

* When you create a `service principal`, you generally ask Azure to create a `key` at the same time.
* Azure shows you the `key` only when you create the service principal.
  
### Manage service principals for Azure Pipelines

* Pipeline tools include secure ways to specify your service principal's application ID and key.
* Never store credentials of any kind in source control. Instead, use `service connections` when you work with Azure Pipelines.
---
### Create a service principal and key

* You can use the Azure PowerShell cmdlets to create and manage service principals.
* The `New-AzADServicePrincipal` cmdlet creates an application registration in Microsoft Entra ID, adds a service principal to your Microsoft Entra tenant, and creates a key for the application registration.
* This cmdlet accepts several arguments and can optionally assign roles to the service principal.

```PowerShell
$servicePrincipal = New-AzADServicePrincipal -DisplayName MyPipeline
```
When you run this command, Azure PowerShell populates the `servicePrincipal` variable with information about the service principal, including the key:

```PowerShell
$servicePrincipalKey = $servicePrincipal.PasswordCredentials.SecretText
```
You can't get this key again, so be sure to use it immediately or save it somewhere secure.

---

### Identify a service principal

Service principals have several identifiers and names that you use to identify and work with them. The identifiers that you use the most are:
   * **Application ID**: The application registration has a unique identifier, often called an `application ID` or sometimes a `client ID`. You typically use it as the username when the service principal signs in to Azure.
  * **Object ID**: The application registration and the service principal have their own separate object IDs, which are unique identifiers assigned by Microsoft Entra ID. Occasionally, you'll need to use these object IDs when you manage a service principal.
  * **Display name**: This is a human-readable name that describes the service principal.

A display name isn't unique. Multiple service principals might share the same display name. It's a good practice to use the application ID instead.

---
### Handle expired keys

* Service principals don't expire, but their keys do.
* By default, the expiration time is set to one year.
* You need to renew or rotate keys regularly.

  >>> Keys Resetting
* To reset a key for a service principal, first use the `Remove-AzADServicePrincipalCredential` cmdlet to remove the existing credential. 
* Then use the `New-AzADServicePrincipalCredential` cmdlet to add a new credential. 
* These cmdlets both use the service principal's object ID to identify it. 
* Before you use the cmdlets, you need to obtain this ID from the application ID:

```PowerShell
$applicationId = APPLICATION_ID
$servicePrincipalObjectId = (Get-AzADServicePrincipal -ApplicationId $applicationId).Id

Remove-AzADServicePrincipalCredential -ObjectId $servicePrincipalObjectId

$newCredential = New-AzADServicePrincipalCredential -ObjectId $servicePrincipalObjectId
$newKey = $newCredential.SecretText
```

* A single service principal can have multiple keys. 
* You can safely update your application to use a new key while the old key is still valid, and then delete the old key when it's no longer in use. 
* This technique avoids downtime from key expiration.

---

### Manage the lifecycle of your service principal

When you build a service principal for a pipeline, what will happen if the pipeline is eventually deleted or is no longer used?

Service principals aren't removed automatically, so you need to audit and remove old ones. 
attackers might gain access to their keys. It's best not to have credentials that aren't actively used.

It's a good practice to document your service principals in a place that you and your team can easily access. You should include the following information for each service principal:

* Essential identifying information, including its name and application ID.
* The purpose of the service principal.
* Who created it, who's responsible for managing it and its keys, and who might have answers if there's a problem.
* The permissions that it needs, and a clear justification for why it needs them.
* What its expected lifetime is.

You should regularly audit your service principals to ensure that they're still in use and that the permissions they've been assigned are still correct.

---
---

### Grant a service principal access to Azure

### Service principal authorization

* It's the responsibility of the Azure role-based access control (RBAC) system, sometimes called identity and access management (IAM). 
* By using Azure RBAC, you can grant a service principal access to a specific resource group, subscription, or management group.
* Microsoft Entra ID also has its own role system, which is sometimes called directory roles. You use these roles to grant permissions for service principals to manage Microsoft Entra ID. 
---

### Select the right role assignment for your pipeline

**A role assignment has three key parts:** 
  * who the role is assigned to (the assignee), 
  * what they can do (the role), and 
  * what resource or resources the role assignment applies to (the scope).

### Assignee
* You use the service principal's `application ID` to identify the correct service principal for that assignee.

### Role

* **Reader**: which allows the assignee to read information about resources but not modify or delete them.
* **Contributor**: which allows the assignee to create resources and to read and modify existing resources. However, contributors can't grant other principals access to resources.
* **Owner**: which allows full control over resources, including granting other principals access.

You can also create your own custom role definition to specify the exact list of permissions that you want to assign.

### Scope

### Mixing and matching role assignments

* You can create multiple role assignments that provide different permissions at different scopes.

### Working with multiple environments

* You should create separate service principals for each environment, and grant each service principal the minimum set of permissions that it needs for its deployments.

### Create a role assignment for a service principal

* To create a role assignment for a service principal, use the `New-AzRoleAssignment` cmdlet. You need to specify the assignee, role, and scope:

```PowerShell
New-AzRoleAssignment `
  -ApplicationId 00001111-aaaa-2222-bbbb-3333cccc4444 `
  -RoleDefinitionName Contributor `
  -Scope '/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/ToyWebsite' `
  -Description "The deployment pipeline for the company's website needs to be able to create resources within the resource group."
```

Let's look at each argument:

  * `-ApplicationId` specifies the service principal's application registration ID.
  * `-RoleDefinitionName` specifies the name of a built-in role. If you use a custom role definition, specify the full role definition ID by using the `-RoleDefinitionId` argument instead.
  * `-Scope` specifies the scope. This is usually a resource ID for a single resource, a resource group, or a subscription.
  * `-Description` is a human-readable description of the role assignment.
---
### Create a service principal and role assignment in one operation

```PowerShell
$servicePrincipal = New-AzADServicePrincipal `
  -DisplayName MyPipeline `
  -Role Contributor ` # here
  -Scope '/subscriptions/f0750bbe-ea75-4ae5-b24d-a92ca601da2c/resourceGroups/ToyWebsite' # here
```

### Grant access using Bicep

* Role assignments are Azure resources. This means that you can create a role assignment by using Bicep. 

```Bicep
resource roleAssignment 'Microsoft.Authorization/roleAssignments@2023-04-01-preview' = {
  name: guid(principalId, roleDefinitionId, resourceGroup().id)
  properties: {
    principalType: 'ServicePrincipal'
    roleDefinitionId: subscriptionResourceId('Microsoft.Authorization/roleDefinitions', roleDefinitionId)
    principalId: principalId
    description: 'The deployment pipeline for the company\'s website needs to be able to create resources within the resource group.'
  }
}
```
Let's look at each argument:

* `name` is a unique identifier for the role assignment. This must be in the form of a globally unique identifier (GUID). It's a good practice to use the `guid()` function in Bicep to create a GUID, and to use the principal ID, role definition ID, and scope as the seed arguments for the function to ensure you create a name that's unique for each role assignment.
* `principalType` should be set to `ServicePrincipal`.
* `roleDefinitionId` is the fully qualified resource ID for the role definition you're assigning. Mostly you'll work with built-in roles, and you'll find the role definition ID in the Azure built-in roles documentation. For example, the Contributor role has the role definition ID `b24988ac-6180-42a0-ab88-20f7382dd24c`. When you specify it in your Bicep file, you express this using a fully qualified resource ID, such as `/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c`.
* `principalId` is the service principal's object ID. Make sure you don't use the application ID or the application registration's object ID.
* `description` is a human-readable description of the role assignment.

---
---

### Test your Bicep code by using Azure Pipelines

### Understand pipeline stages

* `Stages` help you to divide your pipeline into multiple logical blocks. Each stage can contain one or more `jobs`. Jobs contain an ordered list of `steps` that should be completed, like running command-line scripts.

>>> Continous Integration (CI) and Continous Deployement (CD)

* When you use an automated pipeline, building and testing your code are often called continuous integration (`CI`).
* Deploying code in an automated pipeline is often called continuous deployment (`CD`).


* During CI stages, you check the validity of the changes that have been made to your code.
* Stages run in a sequence. 
* You can control how and when each stage runs. 
* For example, you can configure your CD stages to run only after your CI stages run successfully.

---

### Shifting left

* By using stages, you can verify the quality of your code before you deploy it. 
* Using these stages is sometimes called `shifting left`.

- Consider a timeline of the activities that you perform when you write code. 
- The timeline starts with the 
  * plan 
  * design 
  * build
  * test
  * deploy
  * support 
- If you write them down in line then the closer to the left side of the timeline, the easier, faster, and cheaper it is to fix
- When you work with tools like Azure DevOps, pull requests typically represent changes that someone on your team wants to make to the code on your main branch.

---

### Define a pipeline stage

* When you have multiple stages in a pipeline, you need to define each one. 
* Stages run in a sequence that you specify.
* Imagine that you've built a Bicep file that you need to deploy twice: 
  - once to infrastructure in the United States and 
  - once to infrastructure in Europe.
* Before you deploy the file, you validate your Bicep code. 

Here's how the stages are defined in a pipeline YAML file:

```YAML
stages:

- stage: Test
  jobs:
  - job: Test

- stage: DeployUS
  jobs: 
  - job: DeployUS

- stage: DeployEurope
  jobs: 
  - job: DeployEurope
```

### Control the sequence of stages

* The stages run in the order that you define. 
* A stage runs only if the previous stage succeeds. 
* You can specify the dependencies between stages by using the `dependsOn` keyword:

```YAML
stages:

- stage: Test
  jobs:
  - job: Test

- stage: DeployUS
  dependsOn: Test # Here
  jobs: 
  - job: DeployUS

- stage: DeployEurope
  dependsOn: Test # Here
  jobs: 
  - job: DeployEurope

```

**In reality, stages and jobs run in parallel only if you have enough agents to run multiple jobs at the same time. When you use Microsoft-hosted agents, you might need to purchase additional parallel jobs to achieve this.**

---

### Conditions

* You might want to run a stage when a previous stage fails. 
* For example, here's a different pipeline. If the deployment fails, a stage called Rollback runs immediately afterward:
* You can use the condition keyword to specify a condition that should be met before a stage runs:

```YAML
stages:

- stage: Test
  jobs:
  - job: Test

- stage: Deploy
  dependsOn: Test
  jobs: 
  - job: Deploy

- stage: Rollback
  condition: failed('Deploy') # Here
  jobs: 
  - job: Rollback

```

* Every job runs on a new agent. That means that every job starts from a clean environment, so in every job you typically need to check out the source code as your first step.
---
### Bicep deployment stages

1. **Lint**: Use the Bicep linter to verify that the Bicep file is well formed and doesn't contain any obvious errors.
2. **Validate**: Use the Azure Resource Manager preflight validation process to check for problems that might occur during deployment.
3. **Preview**: Use the what-if command to validate a list of changes that will be applied against your Azure environment. Have a human manually review the what-if results and approve the pipeline before it proceeds.
4. **Deploy**: Submit your deployment to Resource Manager and wait for it to finish.
5. **Smoke Test**: Run basic post-deployment checks against some of the important resources that you deployed. These reviews are called `infrastructure smoke tests`.

---
---

### Lint and validate your Bicep code

* When you deploy a Bicep file, the Bicep tooling first runs some basic validation steps.
* Bicep runs a linter over your files. Linting is the process of checking your code against a set of recommendations. 
* The Bicep linter reviews your file and verifies that you followed best practices for maintainability, correctness, flexibility, and extensibility.

**A linter contains a predefined set of rules for each of these categories. Example linter rules include:**

* **Unused parameters**. The linter scans for any parameters that aren't used anywhere in the Bicep file. By eliminating unused parameters, you make it easier to deploy your template because you don't have to provide unnecessary values. You also reduce confusion when someone tries to work with your Bicep file.
* **String interpolation**. The linter checks if your file uses the `concat()` function instead of Bicep string interpolation. String interpolation makes your Bicep files more readable.
* **Default values for secure parameters**. The linter warns you if you set default values for parameters that are marked with the `@secure()` decorator. A default value for a secure parameter is a bad practice because it gives the secure parameter a human-readable value, and people might not change it before deployment.

* The Bicep linter runs automatically every time you build a Bicep file.
* You can configure Bicep to verify your file by manually building the Bicep file via the Bicep CLI:

```PowerShell
bicep build main.bicep
```
* When you run the `build` command, Bicep also transpiles your Bicep code to a `JSON ARM template`. You generally don't need the file that it outputs, so you can ignore it.
* Because you want the linter to check your Bicep templates each time anyone checks code into your repository, you can add a lint stage and job to your pipeline:
* You can express this addition in your pipeline YAML file like this:

```YAML
stages:

- stage: Lint
  jobs: 
  - job: Lint
    steps:
      - script: |
          az bicep build --file deploy/main.bicep  # Here
```

---

### Linter warnings and errors

* By default, the linter emits a warning when it discovers that a Bicep file has violated a rule and these aren't treated as errors, so they won't stop the pipeline run or stop subsequent stages from running.
* You can change this behavior by configuring Bicep to treat the linter rule violations as errors instead of warnings. 
* You do this configuration by adding a bicepconfig.json file to the folder that contains your Bicep file. 
* You can decide which linter issues should be treated as errors and which should remain as warnings.
---
### Preflight validation

* You also should check whether your Bicep template is likely to deploy to your Azure environment successfully. 
* This check is called `preflight validation`, and it runs checks that need information from Azure. 
* These kinds of checks include:
  - Are the names that you specified for your Bicep resources valid?
  - Are the names that you specified for your Bicep resources already taken?
  - Are the regions that you're deploying your resources to valid?
* Preflight validation requires communication with Azure, but it doesn't actually deploy any resources.

You can use the `AzureResourceManagerTemplateDeployment` task to submit a Bicep file for preflight validation. Configure the `deploymentMode` to `Validation`:

```YAML
- stage: Validate
  jobs:
  - job: Validate
    steps:
      - task: AzureResourceManagerTemplateDeployment@3
        inputs:
          connectedServiceName: 'MyServiceConnection'
          location: $(deploymentDefaultLocation)
          deploymentMode: Validation # Here
          resourceGroupName: $(ResourceGroupName)
          csmFile: deploy/main.bicep
```

