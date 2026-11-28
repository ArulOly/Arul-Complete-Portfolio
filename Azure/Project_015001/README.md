# Azure: Inventory — Baseline Setup for School Lab (Project 015001)

## Project overview

Shared classroom equipment provides the setting for this inventory exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The school lab needs a repeatable inventory baseline that another student can verify.

## Project objectives

- Record the school lab inventory baseline.
- Declare an ARM storage account with inventory tags and HTTPS-only access.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Azure Resource Manager JSON and Azure Blob Storage.

## System architecture

`Inventory parameters → infrastructure template → local validation → declared asset baseline`

## Features

- Record the school lab inventory baseline.
- Declare an ARM storage account with inventory tags and HTTPS-only access.

## Prerequisites

Use the Azure Resource Manager 2023-05-01 Storage schema to review the template. Use a text editor and a local practice directory.

## Complete Azure source code

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageName": {
      "type": "string",
      "minLength": 3,
      "maxLength": 24
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2023-05-01",
      "name": "[parameters('storageName')]",
      "location": "[parameters('location')]",
      "kind": "StorageV2",
      "sku": {
        "name": "Standard_LRS"
      },
      "tags": {
        "Asset": "LAB-17",
        "Environment": "practice"
      },
      "properties": {
        "allowBlobPublicAccess": false,
        "supportsHttpsTrafficOnly": true,
        "minimumTlsVersion": "TLS1_2"
      }
    }
  ]
}
```

## Implementation explanation

Declare an ARM storage account with inventory tags and HTTPS-only access.

## Sample input data

Use this example input.

```json
{
  "storageName": "labinventory17",
  "location": "eastus"
}
```

## Execution instructions

Save the source as `inventory.json`. In an authorized Azure environment, the Azure CLI can check the planned template using `az deployment group validate --resource-group YOUR_PRACTICE_GROUP --template-file inventory.json --parameters storageName=YOUR_UNIQUE_NAME`. This account-level check was not run for the portfolio.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "resource_type": "Microsoft.Storage/storageAccounts",
  "asset_tag": "LAB-17",
  "allowBlobPublicAccess": false,
  "supportsHttpsTrafficOnly": true,
  "minimumTlsVersion": "TLS1_2"
}
```

## Testing and validation

The declared storage-account resource passed Microsoft's ARM resource schema with the illustrated parameter values. A local check verified the fields shown below.

```json
{
  "resource_type": "Microsoft.Storage/storageAccounts",
  "asset_tag": "LAB-17",
  "allowBlobPublicAccess": false,
  "supportsHttpsTrafficOnly": true,
  "minimumTlsVersion": "TLS1_2"
}
```

## Error handling

Check syntax, resource types, and input names before deployment. Always use an authorized practice resource group or project for provider-side validation.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

This example covers the declared configuration. No cloud resources were provisioned or checked in a live account.

## Learning outcomes

Practice inventory using Azure and check a repeatable baseline setup workflow in the school lab scenario.

## Conclusion

The project records an observable inventory result for training case `LAB-17`.

[Back to the Azure project index](../README.md)
