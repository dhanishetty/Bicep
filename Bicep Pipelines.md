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

