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

* With `parameters`, you can provide information to a Bicep template at deployment time. 
* You can make a Bicep template flexible and reusable by declaring parameters within your template.
* `Decorators` provide a way to attach constraints and metadata to a parameter, which helps anyone using your templates to understand what information they need to provide.
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

---

### Understand parameter types

### Objects

* You can use object parameters to combine structured data together in one place. An object can have multiple properties of different types.

```Bicep
param appServicePlanSku object = {
  name: 'F1'
  tier: 'Free'
  capacity: 1
}
```

- When you reference the parameter in the template, you can select the individual properties of the object by using a dot followed by the name of the property, like in this example:

```Bicep
resource appServicePlan 'Microsoft.Web/serverfarms@2024-04-01' = {
  name: appServicePlanName
  location: location
  sku: {
    name: appServicePlanSku.name // Here
    tier: appServicePlanSku.tier // Here
    capacity: appServicePlanSku.capacity // Here
  }
}
```

* You can attach custom tag metadata to the resources that you deploy, which you can use to identify important information about a resource.

```Bicep
param resourceTags object = {
  EnvironmentName: 'Test'
  CostCenter: '1000100'
  Team: 'Human Resources'
}}
```
* Whenever you define a resource in your Bicep file, you can reuse it wherever you define the tags property:

```Bicep
resource appServicePlan 'Microsoft.Web/serverfarms@2024-04-01' = {
  name: appServicePlanName
  location: location
  tags: resourceTags // Here
  sku: {
    name: 'S1'
  }
}

resource appServiceApp 'Microsoft.Web/sites@' = {
  name: appServiceAppName
  location: location
  tags: resourceTags // Here
  kind: 'app'
  properties: {
    serverFarmId: appServicePlan.id
  }
}
```
---

### Arrays

* you might use an array of string values to declare a list of email addresses for an Azure Monitor action group.
* For example. Azure Cosmos DB lets you create database accounts that span `multiple regions`, and it automatically handles the data replication for you. 
* When you deploy a new database account, you need to specify the list of Azure regions that you want the account to be deployed into.

```Bicep
param cosmosDBAccountLocations array = [
  {
    locationName: 'australiaeast'
  }
  {
    locationName: 'southcentralus'
  }
  {
    locationName: 'westeurope'
  }
]
```
* When you declare your Azure Cosmos DB resource, you can now reference the array parameter:

```Bicep
resource account 'Microsoft.DocumentDB/databaseAccounts@2024-11-15' = {
  name: accountName
  location: location
  properties: {
    locations: cosmosDBAccountLocations
  }
}
```
---

### Specify a list of allowed values

* To enforce this rule, you can use the `@allowed` parameter decorator.
* A parameter decorator is a way of giving Bicep information about what a parameter's value needs to be. 
* Here's parameter named appServicePlanSkuName can be restricted so that only a few specific values can be assigned:

```Bicep
@allowed([
  'P1v3'
  'P2v3'
  'P3v3'
])
param appServicePlanSkuName string
```
---

### Restrict parameter length and values

* It's a good practice to specify the minimum and maximum character length for parameters that control naming, to avoid errors later during deployment.

```Bicep
@minLength(5)
@maxLength(24)
param storageAccountName string
```
* You can apply multiple decorators to a parameter by putting each decorator on its own line.

* When you work with numeric parameters, you might need values to be in a particular range.
* For example, your company might decide that whenever anybody deploys an App Service plan, they should always deploy at least one instance, but no more than 10 instances of the plan. 

```Bicep
@minValue(1)
@maxValue(10)
param appServicePlanInstanceCount int
```
---
### Add descriptions to parameters

* When someone use your templates, they'll need to understand what each parameter does so they can provide the right values.

```Bicep
@description('The locations into which this Cosmos DB account should be configured. This parameter needs to be a list of objects, each of which has a locationName property.')
param cosmosDBAccountLocations array
```

---
---

### Create parameters files

* Parameters files make it easy to specify parameter values together as a set.
* parameters files are created by using a Bicep parameters file with the .bicepparam file extension or a JSON parameters file that contains the parameter values.
* You can supply a parameters file when you deploy your Bicep template.


```Bicep
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "appServicePlanInstanceCount": {
      "value": 3
    },
    "appServicePlanSku": {
      "value": {
        "name": "P1v3",
        "tier": "PremiumV3"
      }
    },
    "cosmosDBAccountLocations": {
      "value": [
        {
          "locationName": "australiaeast"
        },
        {
          "locationName": "southcentralus"
        },
        {
          "locationName": "westeurope"
        }
      ]
    }
  }
}
```

* The `parameters` section lists each parameter and the value you want to use. The parameter value must be specified as an object. The object has a property called `value` that defines the actual parameter value to use.

- Generally, you'll create a parameters file for each environment. 
- It's a good practice to include the environment name in the name of the parameters file. 
- For example, you might have a parameters file named `main.parameters.dev.json` for your development environment and one named `main.parameters.production.json` for your production environment.

---

### Use parameters files at deployment time

```Bicep
New-AzResourceGroupDeployment `
  -Name main `
  -TemplateFile main.bicep `
  -TemplateParameterFile main.parameters.json
```

---
### Override parameter values

* Here's an example Bicep file that defines three parameters, each with default values:

```Bicep
param location string = resourceGroup().location
param appServicePlanInstanceCount int = 1
param appServicePlanSku object = {
  name: 'F1'
  tier: 'Free'
}
```

* Let's look at a parameters file that overrides the value of two of the parameters but doesn't specify a value for the `location` parameter:

```Bicep
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "appServicePlanInstanceCount": {
      "value": 3
    },
    "appServicePlanSku": {
      "value": {
        "name": "P1v3",
        "tier": "PremiumV3"
      }
    }
  }
}
```

* When you create the deployment, you override one of the parameter values. You specify the parameter name as if it's an argument to the cmdlet:

```Bicep
New-AzResourceGroupDeployment `
  -Name main `
  -TemplateFile main.bicep `
  -TemplateParameterFile main.parameters.json `
  -appServicePlanInstanceCount 5
```
* `location` = 	Value is the resource group's location. =	The Bicep file specifies this parameter as a default value, and it's not overridden.
* `appServicePlanSku` = Value of an object with a `name` property set to `P1v3` and a `tier` of `PremiumV3`.	The default value in the Bicep file is overridden by the parameters file.
* `appServicePlanInstanceCount`	= value is `5`	= The value specified at deployment time overrides the default value and the value in the parameters file.

---

### Secure your parameters

* Sometimes you need to pass sensitive values into your deployments, like passwords and API keys. But you need to ensure these values are protected.

**The best approach is to avoid using credentials entirely. `Managed identities` for Azure resources can enable the components of your solution to securely communicate with one another without any credentials. `Managed identities` aren't available for every resource, but it's a good idea to use them wherever you can. Where you can't, you can use the approaches described here.**

### Define secure parameters

* The `@secure` decorator can be applied to string and object parameters that contain secret values. 
* When you define a parameter as `@secure`, Azure won't make the parameter values available in the deployment logs.
* If you create the deployment interactively by using the Azure CLI or Azure PowerShell and you need to enter the values during the deployment, the terminal won't display the text on your screen.

```Bicep
@secure()
param sqlServerAdministratorLogin string

@secure()
param sqlServerAdministratorPassword string
```
### Avoid using parameters files for secrets

* As Parameters files are often saved to a centralized version control system, like Git. 

### Integrate with Azure Key Vault

* Integrate your Bicep templates with Key Vault by using a parameters file with a reference to a `Key Vault secret`.
* You can use this feature by referring to the key vault and secret in your parameters file.
* The value is never exposed because you only reference its `identifier`, which by itself isn't anything secret. 
* When you deploy the template, Azure Resource Manager will contact the key vault and retrieve the data.

**You can refer to secrets in key vaults that are located in a different resource group or subscription from the one you're deploying to.**

* Here's a parameters file that uses Key Vault references to look up the SQL logical server administrator login and password to use:

```Bicep
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "sqlServerAdministratorLogin": {
      "reference": {
        "keyVault": {
          "id": "/subscriptions/f0750bbe-ea75-4ae5-b24d-a92ca601da2c/resourceGroups/PlatformResources/providers/Microsoft.KeyVault/vaults/toysecrets"
        },
        "secretName": "sqlAdminLogin"
      }
    },
    "sqlServerAdministratorPassword": {
      "reference": {
        "keyVault": {
          "id": "/subscriptions/f0750bbe-ea75-4ae5-b24d-a92ca601da2c/resourceGroups/PlatformResources/providers/Microsoft.KeyVault/vaults/toysecrets"
        },
        "secretName": "sqlAdminLoginPassword"
      }
    }
  }
}
```
* Notice that instead of specifying a `value` for each of the parameters, this file has a `reference` object, which contains details of the key vault and secret.

**Your key vault must be configured to allow Resource Manager to access the data in the key vault during template deployments. Also, the user who deploys the template must have permission to access the key vault.**

---

### Use Key Vault with modules

* Modules may have parameters that accept secret values, and you can use Bicep's Key Vault integration to provide these values securely.

```Bicep
resource keyVault 'Microsoft.KeyVault/vaults@2023-07-01' existing = {
  name: keyVaultName
}

module applicationModule 'application.bicep' = {
  name: 'application-module'
  params: {
    apiKey: keyVault.getSecret('ApiKey')
  }
}
```

* Notice that in this Bicep file, the Key Vault resource is referenced by using the `existing `keyword. 
* The keyword tells Bicep that the Key Vault already exists, and this code is a reference to that vault. 
* Bicep won't redeploy it. 
* Also, notice that the module's code uses the `getSecret()` function in the value for the module's `apiKey` parameter. 
* This is a special Bicep function that can only be used with secure module parameters.

---
---
### Deploy resources conditionally

* You can use conditions in your Bicep code to deploy resources only when specific constraints are in place.

### Use basic conditions

* When you deploy a resource in Bicep, you can provide the `if` keyword followed by a condition. 
* The condition should resolve to a Boolean (true or false) value. 
* If the value is true, the resource is deployed. If the value is false, the resource is not deployed.