1. Infrastructure as code (IaC) automates infrastructure provisioning using a descriptive coding language, ensuring consistent and repeatable deployments.
2. IaC can be compared to an instruction manual that details the desired configuration of resources.
3. Benefits of adopting IaC include:
    * Increased confidence in deployments through consistency and security.
    * Ability to manage multiple environments effectively.
    * Improved understanding of cloud resources with version control and documentation.
4. IaC helps avoid configuration drift through idempotent operations, allowing for consistent redeployment.
5. It supports provisioning new environments, differentiating between production and nonproduction setups, and aids in disaster recovery.
6. Two approaches to IaC are imperative (step-by-step commands) and declarative (specifying end configuration without detailing the process).
7. Examples of imperative code include scripting languages like Bash or Azure PowerShell, while declarative code can be written using templates like JSON or Bicep.
8. The module focuses on using Bicep templates for declarative IaC.

---
### deploying Bicep using Azure CLI

az deployment group create --template-file main.bicep --resource-group storage-resource-group

---
---

### Convert Bicep file to Json file

az bicep build --file main.bicep

---

