# terraform-azurerm-static-website

A module to enable a static website for an azure storage account. This is a workaround until <https://github.com/terraform-providers/terraform-provider-azurerm/issues/1903> is resolved

## Limtations

1. You need a valid session in the Azure CLI (even when you Authenticating terrafrom not with the Azure CLI)
2. A destroy does not disable the website. You must first set `enabled` to false and apply. The you can remove the resource

## Example

```terraform
resource "azurerm_resource_group" "testrg" {
  name     = "resourceGroupName"
  location = "westus"
}

resource "azurerm_storage_account" "testsa" {
  name                = "storageaccountnamessh"
  resource_group_name = azurerm_resource_group.testrg.name
  location            = "westus"

  account_tier             = "Standard"
  account_kind             = "StorageV2"
  account_replication_type = "GRS"
}

module "staticweb" {
  source               = "./terraform-azurerm-static-website"
  storage_account_name = azurerm_storage_account.testsa.name
}

data "azurerm_storage_account" "test" {
  name                = azurerm_storage_account.testsa.name
  resource_group_name = azurerm_resource_group.testrg.name

  depends_on = ["module.staticweb"]
}

output "static-web-url" {
  value = data.azurerm_storage_account.test.primary_web_endpoint
}
```