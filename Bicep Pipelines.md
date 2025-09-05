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