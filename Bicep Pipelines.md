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