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
* 

- The following code deploys a storage account only when the `deployStorageAccount` parameter is set to `true`:

```Bicep
param deployStorageAccount bool

resource storageAccount 'Microsoft.Storage/storageAccounts@2023-05-01' = if (deployStorageAccount) {
  name: 'teddybearstorage'
  location: resourceGroup().location
  kind: 'StorageV2'
  // ...
}
```

### Use expressions as conditions

* In the following example, the code deploys a SQL auditing resource only when the `environmentName` parameter value is equal to `Production`:

```Bicep
@allowed([
  'Development'
  'Production'
])
param environmentName string

resource auditingSettings 'Microsoft.Sql/servers/auditingSettings@2024-05-01-preview' = if (environmentName == 'Production') {
  parent: server
  name: 'default'
  properties: {
  }
}
```
* It's usually a good idea to create a variable for the expression that you're using as a condition. That way, your Bicep file is easier to understand and read. Here's an example:

```Bicep
@allowed([
  'Development'
  'Production'
])
param environmentName string

var auditingEnabled = environmentName == 'Production'

resource auditingSettings 'Microsoft.Sql/servers/auditingSettings@2024-05-01-preview' = if (auditingEnabled) {
  parent: server
  name: 'default'
  properties: {
  }
}
```

---
### Depend on conditionally deployed resources

* When you deploy resources conditionally, you sometimes need to be aware of how Bicep evaluates the dependencies between them.

```Bicep
@allowed([
  'Development'
  'Production'
])
param environmentName string
param location string = resourceGroup().location
param auditStorageAccountName string = 'bearaudit${uniqueString(resourceGroup().id)}'

var auditingEnabled = environmentName == 'Production'
var storageAccountSkuName = 'Standard_LRS'

resource auditStorageAccount 'Microsoft.Storage/storageAccounts@2023-05-01' = if (auditingEnabled) {
  name: auditStorageAccountName
  location: location
  sku: {
    name: storageAccountSkuName
  }
  kind: 'StorageV2'
}

resource auditingSettings 'Microsoft.Sql/servers/auditingSettings@2024-05-01-preview' = if (auditingEnabled) {
  parent: server
  name: 'default'
  properties: {
  }
}
```
* Notice that the storage account has a condition too. This means that it won't be deployed for non-production environments either. The SQL auditing settings resource can now refer to the storage account details:

```Bicep
resource auditingSettings 'Microsoft.Sql/servers/auditingSettings@2024-05-01-preview' = if (auditingEnabled) {
  parent: server
  name: 'default'
  properties: {
    state: 'Enabled'
    storageEndpoint: environmentName == 'Production' ? auditStorageAccount.properties.primaryEndpoints.blob : ''
    storageAccountAccessKey: environmentName == 'Production' ? listKeys(auditStorageAccount.id, auditStorageAccount.apiVersion).keys[0].value : ''
  }
}
```

* Notice that this Bicep code uses the question mark (?) operator within the storageEndpoint and storageAccountAccessKey properties. 
* When the Bicep code is deployed to a production environment, the expressions are evaluated to the details from the storage account. 
* When the code is deployed to a non-production environment, the expressions evaluate to an empty string ('').

* You might wonder why this code is necessary, because auditingSettings and auditStorageAccount both have the same condition, and so you'll never need to deploy a SQL auditing settings resource without a storage account. 
* Although this is true, Azure Resource Manager evaluates the property expressions before the conditionals on the resources. 
* That means that if the Bicep code doesn't have this expression, the deployment will fail with a ResourceNotFound error.

**You can't define two resources with the same name in the same Bicep file and then conditionally deploy only one of them. The deployment will fail, because Resource Manager views this as a conflict.**

---
### Deploy multiple resources by using loops

### Use copy loops

* When you define a resource or a module in a Bicep file, you can use the `for` keyword to create a loop. 
* Place the `for` keyword in the resource declaration, then specify how you want Bicep to identify each item in the loop.


```Bicep
param storageAccountNames array = [
  'saauditus'
  'saauditeurope'
  'saauditapac'
]

resource storageAccountResources 'Microsoft.Storage/storageAccounts@2023-05-01' = [for storageAccountName in storageAccountNames: {
  name: storageAccountName
  location: resourceGroup().location
  kind: 'StorageV2'
  sku: {
    name: 'Standard_LRS'
  }
}]
```

* Bicep requires you put an opening bracket (`[`) character before the for keyword, and a closing bracket (`]`) character after the resource definition.

---

### Loop based on a count

* Bicep provides the `range()` function, which creates an array of numbers. 
* For example, if you need to create four storage accounts called `sa1` through `sa4`, you could use a resource definition like this:

```Bicep
resource storageAccountResources 'Microsoft.Storage/storageAccounts@2023-05-01' = [for i in range(1,4): {
  name: 'sa${i}'
  location: resourceGroup().location
  kind: 'StorageV2'
  sku: {
    name: 'Standard_LRS'
  }
}]
```
* When you use the `range()` function, you specify its start value and the number of values you want to create. 
* For example, if you want to create storage accounts with the names `sa0`, `sa1`, and `sa2`, you'd use the function `range(0,3)`.

---

### Access the iteration index

* With Bicep, you can iterate through arrays and retrieve the index of the current element in the array. 
* For example, let's say you want to create a logical server in each location that's specified by an array, and you want the server names to be `sqlserver-1`, `sqlserver-2`, and so on. You could achieve this by using the following Bicep code:

```Bicep
param locations array = [
  'westeurope'
  'eastus2'
  'eastasia'
]

resource sqlServers 'Microsoft.Sql/servers@2024-05-01-preview' = [for (location, i) in locations: {
  name: 'sqlserver-${i+1}'
  location: location
  properties: {
    administratorLogin: administratorLogin
    administratorLoginPassword: administratorLoginPassword
  }
}]
```
---
### Filter items with loops

* In some situations, you might want to deploy resources by using copy loops combined with conditions. You can do this by combining the `if` and `for` keywords.
* In the following example, the code uses an array parameter to define a set of logical servers. A condition is used with the copy loop to deploy the servers only when the `environmentName` property of the loop object equals `Production`:

```Bicep
param sqlServerDetails array = [
  {
    name: 'sqlserver-we'
    location: 'westeurope'
    environmentName: 'Production'
  }
  {
    name: 'sqlserver-eus2'
    location: 'eastus2'
    environmentName: 'Development'
  }
  {
    name: 'sqlserver-eas'
    location: 'eastasia'
    environmentName: 'Production'
  }
]

resource sqlServers 'Microsoft.Sql/servers@2024-05-01-preview' = [for sqlServer in sqlServerDetails: if (sqlServer.environmentName == 'Production') {
  name: sqlServer.name
  location: sqlServer.location
  properties: {
    administratorLogin: administratorLogin
    administratorLoginPassword: administratorLoginPassword
  }
  tags: {
    environment: sqlServer.environmentName
  }
}]
```
* If you deployed the preceding example, you'd see two logical servers, `sqlserver-we` and `sqlserver-eas`, but not `sqlserver-eus2`, because that object's environmentName property doesn't match Production.

---
### Control loop execution

* In some cases, you might need to deploy resources in loops sequentially instead of in parallel.
* For example, if you have lots of Azure App Service apps in your production environment, you might want to deploy changes to only a small number at a time to prevent the updates from restarting all of them at the same time.
* You can control the way your copy loops run in Bicep by using the `@batchSize` decorator. Put the decorator on the resource or module declaration with the `for` keyword.

**All the resources in the following loop will be deployed at the same time, in parallel:**

```Bicep
resource appServiceApp 'Microsoft.Web/sites@2024-04-01' = [for i in range(1,3): {
  name: 'app${i}'
  // ...
}]
```
* Now let's apply the `@batchSize` decorator with a value of `2`. When you deploy the file, Bicep will deploy in batches of two:
* You can also tell Bicep to run the loop sequentially by setting the `@batchSize` to `1`:

**Bicep waits for each complete batch to finish before it moves on to the next. In the preceding example, if `app2` finishes its deployment before app1, Bicep waits until `app1` finishes before it starts to deploy `app3`.**

---

### Use loops with resource properties

* You can use loops to help set resource properties. 
* For example, when you deploy a virtual network, you need to specify its subnets. 
* A subnet has to have two pieces of important information: 
    - a name and an address prefix. 
* You can use a parameter with an array of objects so that you can specify different subnets for each environment:

```Bicep
param subnetNames array = [
  'api'
  'worker'
]

resource virtualNetwork 'Microsoft.Network/virtualNetworks@2024-05-01' = {
  name: 'teddybear'
  location: resourceGroup().location
  properties: {
    addressSpace: {
      addressPrefixes: [
        '10.0.0.0/16'
      ]
    }
    subnets: [for (subnetName, i) in subnetNames: {
      name: subnetName
      properties: {
        addressPrefix: '10.0.${i}.0/24'
      }
    }]
  }
}
```
---
### Nested loops

* For your toy company, you need to deploy virtual networks in every country/region where the toy will be launched. 
* Every virtual network needs a different address space and two subnets. 
* Let's start by deploying the virtual networks in a loop:
* This loop deploys the virtual networks for each location, and it sets the addressPrefix for the virtual network by using the loop index to ensure each virtual network gets a different address prefix.
* You can use a nested loop to deploy the subnets within each virtual network:
* The nested loop uses the range() function to create two subnets.

```Bicep
param locations array = [
  'westeurope'
  'eastus2'
  'eastasia'
]

var subnetCount = 2

resource virtualNetworks 'Microsoft.Network/virtualNetworks@2024-05-01' = [for (location, i) in locations : {
  name: 'vnet-${location}'
  location: location
  properties: {
    addressSpace:{
      addressPrefixes:[
        '10.${i}.0.0/16'
      ]
    }
    subnets: [for j in range(1, subnetCount): {
      name: 'subnet-${j}'
      properties: {
        addressPrefix: '10.${i}.${j}.0/24'
      }
    }]
  }
}]
```
**When you deploy the Bicep file, you get the following virtual networks and subnets:**

Here’s the table based on your provided data:

| Virtual network name | Location    | Address prefix   | Subnets                      |
|-----------------------|------------|------------------|-----------------------------|
| vnet-westeurope      | westeurope | 10.0.0.0/16      | 10.0.1.0/24, 10.0.2.0/24    |
| vnet-eastus2         | eastus2    | 10.1.0.0/16      | 10.1.1.0/24, 10.1.2.0/24    |
| vnet-eastasia        | eastasia   | 10.2.0.0/16      | 10.2.1.0/24, 10.2.2.0/24    |


---

### Variable loops

* By using variable loops, you can create an array, which you can then use through your Bicep file. As you do with other loops, you use the `for` keyword to create a variable loop:
* The following example creates an array that contains the values `item1`, `item2`, `item3`, `item4`, and `item5`.

```Bicep
var items = [for i in range(1, 5): 'item${i}']
```

* You'd ordinarily use variable loops to create more complex objects that you could then use within a resource declaration. 
* Here's how to use variable loops to create a `subnets` property:


- This example illustrates an effective use for variable loops: 
- turning a parameter that has simple, easy-to-understand values into a more complex object that corresponds to the Azure resource's required definition. 
- You can use variable loops to enable parameters to specify only the key information that will change for each item in the list. 
- You can then use Bicep expressions or default values to set other required properties for the resource.

```Bicep
param addressPrefix string = '10.10.0.0/16'
param subnets array = [
  {
    name: 'frontend'
    ipAddressRange: '10.10.0.0/24'
  }
  {
    name: 'backend'
    ipAddressRange: '10.10.1.0/24'
  }
]

var subnetsProperty = [for subnet in subnets: {
  name: subnet.name
  properties: {
    addressPrefix: subnet.ipAddressRange
  }
}]

resource virtualNetwork 'Microsoft.Network/virtualNetworks@2024-05-01' = {
  name: 'teddybear'
  location: resourceGroup().location
  properties:{
    addressSpace:{
      addressPrefixes:[
        addressPrefix
      ]
    }
    subnets: subnetsProperty
  }
}
```


---
### Output loops

* You can use Bicep outputs to provide information from your deployments back to the user or tool that started the deployment. 


```Bicep
var items = [
  'item1'
  'item2'
  'item3'
  'item4'
  'item5'
]

output outputItems array = [for i in range(0, length(items)): items[i]]
```
Second Example

```Bicep
param locations array = [
  'westeurope'
  'eastus2'
  'eastasia'
]

resource storageAccounts 'Microsoft.Storage/storageAccounts@2023-05-01' = [for location in locations: {
  name: 'toy${uniqueString(resourceGroup().id, location)}'
  location: location
  kind: 'StorageV2'
  sku: {
    name: 'Standard_LRS'
  }
}]

output storageEndpoints array = [for i in range(0, length(locations)): {
  name: storageAccounts[i].name
  location: storageAccounts[i].location
  blobEndpoint: storageAccounts[i].properties.primaryEndpoints.blob
  fileEndpoint: storageAccounts[i].properties.primaryEndpoints.file
}]
```
---
---
### Encapsulation

* Modules help you keep related resource definitions together. 
* For example, when you define an Azure Functions app, you typically deploy the app, a hosting plan for the app, and a storage account for the app's metadata. 
* These three components are defined separately, but they represent a logical grouping of resources, so it might make sense to define them as a module.

---

### Deployments

* In Azure, a `deployment` is a special resource that represents a deployment operation. 
* Deployments are Azure resources that have the resource type `Microsoft.Resources/deployments`. 
* When you submit a Bicep deployment, you create or update a deployment resource. 
* Similarly, when you create resources in the Azure portal, the portal creates a deployment resource on your behalf.
* However, not all changes to Azure resources create or use deployments. 
* For example, when you use the Azure portal to modify an existing resource, it generally doesn't create a deployment to make the change. 
* When you use third-party tools like Terraform to deploy or configure your resources, they might not create deployments.
* When you deploy a Bicep file by using the Azure CLI or Azure PowerShell, you can optionally specify the name of the deployment. 
* If you don't specify a name, the Azure CLI or Azure PowerShell automatically creates a deployment name for you from the file name of the template. 
* For example, if you deploy a file named main.bicep, the default deployment name is `main`.
* When you use modules, Bicep creates a separate deployment for every module. 
* The name property that you specify for the module becomes the name of the deployment. 
* When you deploy a Bicep file that contains a module, multiple deployment resources are created: one for the parent template and one for each module.
* You can list and view the details of deployment resources to monitor the status of your Bicep deployments or to view history of deployments. 
* However, when you reuse the same name for a deployment, Azure overwrites the last deployment with the same name. 
* If you need to maintain the deployment history, ensure that you use a unique name for every deployment. 
* You might include the date and time of the deployment in the name to help make it unique.

---

### Generated JSON ARM templates

* When you deploy a Bicep file, Bicep converts it to a JSON ARM template. This conversion is also called `transpilation`.
* The modules that the template uses are embedded into the JSON file. 
* Regardless of how many modules you include in your template, only a single JSON file is created.

---

### Add parameters and outputs to modules

