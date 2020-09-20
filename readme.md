# Azure Functions with Private Endpoints

This repo is just a slimmed-down version of the repo found here:
[https://github.com/mcollier/azure-functions-private-storage](https://github.com/mcollier/azure-functions-private-storage) . Refer to this linked repo for further explanation about how the private endpoints work with Functions.

## Architecture Overview

This sample will demonstrate an Azure Function which retrieves files from an Azure Storage blob container.  The function will communicate with the Azure Storage account referenced by AzureWebJobsStorage, via private endpoints.  

### Azure Function

The Azure Function app provisioned in this sample uses an [Azure Functions Elastic Premium plan](https://docs.microsoft.com/azure/azure-functions/functions-premium-plan#features). Alternatively, PremiumV2 App Service Plan can be used. If using an App Service Environment, private endpoints aren't needed.

### Azure Storage accounts

There are two Azure Storage accounts used in this sample:
- storage account referenced by the [AzureWebJobsStorage](https://docs.microsoft.com/azure/azure-functions/functions-app-settings#azurewebjobsstorage) setting. This storage account uses firewall virtual network restrictions and uses private endpoints.
- storage account referenced by the [WEBSITE_CONTENTAZUREFILECONNECTIONSTRING](https://docs.microsoft.com/azure/azure-functions/functions-app-settings#website_contentazurefileconnectionstring) setting. This storage account will contain the application code and does not virtual network restrictions.

### Virtual Network

Azure resources in this sample either integrate with or are placed within a virtual network. The use of private endpoints keeps network traffic contained with the virtual network.

The sample uses four subnets:

- Subnet for Azure Function virtual network integration.  This subnet is delegated to the function.
- Subnet for private endpoints.  Private IP addresses are allocated from this subnet.

### Private Endpoints

[Azure Private Endpoints](https://docs.microsoft.com/azure/private-link/private-endpoint-overview) are used to connect to specific Azure resources using a private IP address.  This ensures that network traffic remains within the designated virtual network, and access is available only for specific resources.  This sample configures private endpoints for the following Azure resources:

- [Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-private-endpoints)
  - Azure File storage
  - Azure Blob storage
  - Azure Queue storage
  - Azure Table storage
  
### Private DNS Zones

Using a private endpoint to connect to Azure resources means connecting to a private IP address instead of the public endpoint.  Existing Azure services are configured to use existing DNS to connect to the public endpoint.  The DNS configuration will need to be overridden to connect to the private endpoint.

A private DNS zone will be created for each Azure resource configured with a private endpoint.  A DNS A record is created for each private IP address associated with the private endpoint. 

The following DNS zones are created in this sample:

- privatelink.queue.core.windows.net
- privatelink.blob.core.windows.net
- privatelink.table.core.windows.net
- privatelink.file.core.windows.net

### Application Insights

[Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/app-insights-overview) is used to [monitor the Azure Function](https://docs.microsoft.com/azure/azure-functions/functions-monitoring).