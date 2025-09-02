For Single comment use `//`
for multi line comment use `/*   */`

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
```

* `param` tells Bicep that you're defining a parameter.
* `appServiceAppName` is the name of the parameter. If you're deploying the Bicep file manually.
* `string` is the type of the parameter.
* ` = 'toy-product-launch-1'` default value.

