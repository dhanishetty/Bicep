For Single comment use `//`
for multi line comment use `/*   */`

---

1. Infrastructure as code (IaC) automates infrastructure provisioning using a descriptive coding language, ensuring consistent and repeatable deployments.
2. IaC can be compared to an instruction manual that details the desired configuration of resources.
3. Benefits of adopting IaC include:
    * Increased confidence in deployments through consistency and security.
    * Ability to manage multiple environments effectively.
    * Improved understanding of cloud resources with version control and documentation.
4. IaC helps avoid configuration drift through idempotent operations, allowing for consistent redeployment.
5. It supports provisioning new environments, differentiating between production and nonproduction setups, and aids in disaster recovery.
6. Two approaches to IaC are **imperative** (step-by-step commands) and **declarative** (just declaring the desired state. specifying end configuration without detailing the process).
7. Examples of imperative code include scripting languages like Bash or Azure PowerShell, while declarative code can be written using templates like JSON or Bicep.
8. The module focuses on using Bicep templates for declarative IaC.

---
### deploying Bicep using Azure CLI

az deployment group create --template-file main.bicep --resource-group storage-resource-group

This command is **deploying your Bicep template into a specific Azure resource group**.

* `az deployment group create` → Creates (or updates) a deployment at **resource group scope**.
* `--template-file main.bicep` → Points to your Bicep template that defines the Azure resources.
* `--resource-group storage-resource-group` → Tells Azure **which resource group** to deploy those resources into (here: `storage-resource-group`).

👉 In short:
It takes the infrastructure defined in **`main.bicep`** and deploys it into the **`storage-resource-group`** resource group in Azure.

---
---

### Convert Bicep file to Json file

az bicep build --file main.bicep

---
---

### Resource providers

A `Resource provider` is a logical grouping of resource types that usually relate to one or a few Azure services.

Examples of resource providers include:

  - `Microsoft.Compute`, which is used for virtual machines.
  - `Microsoft.Network`, which is used for networking resources like virtual networks, network security groups, and route tables.
  - `Microsoft.Cache`, which is used for Azure Cache for Redis.
  - `Microsoft.Sql`, which is used for Azure SQL.
  - `Microsoft.Web`, which is used for Azure App Service and Azure Functions.
  - `Microsoft.DocumentDB`, which is used for Azure Cosmos DB.

---
---

### Resource types

- You combine the resource provider and type name to make a fully qualified resource type name. 
- The fully qualified type name consists of the resource provider name, a slash (`/`), and the resource type. 
    - For example, a storage account's fully qualified type name is `Microsoft.Storage/storageAccounts`. In this instance, its resource provider name is `Microsoft.Storage` and the resource type is `storageAccounts`.

---
---
### Resource IDs

- Every Azure resource has a unique resource ID.
- A resource ID for a storage account looks like this:
        /subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/ToyDevelopment/providers/Microsoft.Storage/storageAccounts/secrettoys
- For example The resource ID for a blob container includes the name of the storage account that contains the container, and the container's name.
  
---

### Parameters and variables

* A `parameter` lets you bring in values from outside the Bicep file.
* A `variable` is defined and set within the Bicep file.
* It's a good idea to use parameters for things that will change between each deployment, like:
  - Resource names that need to be unique.
  - Locations into which to deploy the resources.
  - Settings that affect the pricing of resources, like their SKUs, pricing tiers, and instance counts.
  - Credentials and information needed to access other systems that aren't defined in the Bicep file.

### Add a parameter with default value

```Bicep
param appServiceAppName string // Without default value
param appServiceAppName string  = 'toy-product-launch-1' // With default value

param location string = resourceGroup().location //This deploys all resources into the same location in which the resource group was created.
```

* `param` tells Bicep that you're defining a parameter.
* `appServiceAppName` is the name of the parameter. If you're deploying the Bicep file manually.
* `string` is the type of the parameter.
* ` = 'toy-product-launch-1'` default value.

---
### Adding a variable

```Bicep
var appServicePlanName = 'toy-product-launch-plan'
```
---
### Resource names

* We have two resources that need unique names: the storage account and the App Service app.
* Bicep has another function called uniqueString(). When you use this function, you need to provide a seed value.

```Bicep
param storageAccountName string = uniqueString(resourceGroup().id)

// resourceGroup().id = /subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/MyResourceGroup
```

### Combined strings

* A good resource name should also be descriptive, so that it's clear what the resource is for.
* Bicep has a feature called string interpolation that lets you combine strings.

```Bicep
param storageAccountName string = 'toylaunch${uniqueString(resourceGroup().id)}'
```

* `toylaunch` is a hard-coded string that helps anyone who looks at the deployed resource in Azure to understand the storage account's purpose.
* `${uniqueString(resourceGroup().id)}` is a way of telling Bicep to evaluate the output of the `uniqueString(resourceGroup().id)` function, then concatenate it into the string.

---

### Selecting SKUs (Stock Keeping Unit) (specific configuration or tier of a service or resource) for resources

* A list of allowed values for the environmentType parameter. Bicep won't let anyone deploy the Bicep file unless they provide one of these values.

```Bicep
@allowed([
  'nonprod'
  'prod'
])
param environmentType string
```

* Create variables that determine the SKUs to use for the storage account and App Service plan based on the environment:

```Bicep
var storageAccountSkuName = (environmentType == 'prod') ? 'Standard_GRS' : 'Standard_LRS'
var appServicePlanSkuName = (environmentType == 'prod') ? 'P2V3' : 'F1'
```

* For the `storageAccountSkuName` variable, if the `environmentType` parameter is set to `prod`, then use the `Standard_GRS` SKU. Otherwise, use the `Standard_LRS` SKU.
* For the `appServicePlanSkuName` variable, if the `environmentType` parameter is set to `prod`, then use the `P2V3` SKU and the `PremiumV3` tier. Otherwise, use the `F1` SKU.

---

### Group resources by using modules

* You'll also often need to emit outputs from the Bicep modules and files. 

### Outputs

example scenario's

* You create a Bicep file that deploys a virtual machine, and you need to get the public IP address so you can SSH into the machine.
* You create a Bicep file that accepts a set of parameters, like an environment name and an application name. The file uses an expression to name an Azure App Service app that it deploys. You need to output the app's name that the file has deployed so you can use it within a deployment pipeline to publish the application binaries.

```Bicep
output appServiceAppName string = appServiceAppName
```
The output definition includes a few key parts:

* `output`: Declares an output value for the deployment.
* `appServiceAppName (first one)`: The name of the output variable.
* `string`: The data type of the output.
* `= appServiceAppName (second one)`: The value being assigned to the output, which is usually a variable or property defined earlier in the Bicep file.

---

### Module

* Bicep modules allow you to organize and reuse your Bicep code by creating smaller units that can be composed into a Bicep file. 
* Any Bicep file can be used as a module by another template.
* Imagine you have a Bicep file that deploys application, database, and networking resources for solution A. 
* You might split this Bicep file into three modules, each of which is focused on its own set of resources.
* When you want the Bicep file to include a reference to a module file, use the `module` keyword. A module definition looks similar to a resource declaration, but instead of including a resource type and API version, you'll use the module's file name:

```Bicep
module myModule 'modules/mymodule.bicep' = {
  name: 'MyModule'
  params: {
    location: location
  }
}
```

It means:

- The `module` keyword tells Bicep you're about to use another Bicep file as a module.
- **module myModule 'modules/mymodule.bicep'** → Include another Bicep file called `mymodule.bicep` from the `modules` folder.
- **name: 'MyModule'** → Give this module deployment the name `MyModule`.
- **params: { location: location }** → Pass the `location` parameter from the main file into the module.

In short: *You’re reusing another Bicep template and passing it the location parameter.*  


