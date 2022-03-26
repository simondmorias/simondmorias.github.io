---
author: Simon D'Morias
date: '2018-09-26 11:27:16'
draft: false
title: Controlling the Databricks Resource Group Name
type: post
url: /blog/2018/9/26/controlling-the-databricks-resource-group-name
tags:
- PowerShell
- Databricks
- REST API
- Azure
---

When you create a Databricks workspace using the Azure portal you obviously specify the Resource Group to create it in. But in the background a second resource group is created, this is known as the managed resource group - it is created with an almost random name. 

This is a pain if you have naming conventions or standards to adhere to.

The managed resource group is used for networking of your clusters and for providing the DBFS storage account.

Whilst the portal does not allow you to control this name, the underlying [REST API ](https://docs.microsoft.com/en-us/rest/api/databricks/workspaces/createorupdate#sku)does. So with PowerShell magic we can create an instance and set both names.

Note that both resource groups used in the variables must exist before running this script.
    
 ```
$TenantId = "75e26db8-ef63-42c8-ac89-fbeea21dfe71"
$SubscriptionID = "b146ae31-d42f-4c88-889b-318f2cc23f98"
$ResourceGroupName = "dataThirstTestDatabricks-RG"
$ManagedResourceGroupName = "dataThirstTestDatabricks-MANAGED-RG"
$WorkspaceName = "datatThirstTestWorkspace"
$Location = "westeurope"

Connect-AzureRmAccount -TenantId  $TenantId -Subscription $SubscriptionID

# Get Access Token for authenticating the REST API
$currentAzureContext = Get-AzureRmContext
$azureRmProfile = [Microsoft.Azure.Commands.Common.Authentication.Abstractions.AzureRmProfileProvider]::Instance.Profile
$profileClient = New-Object Microsoft.Azure.Commands.ResourceManager.Common.RMProfileClient($azureRmProfile)
$token = $profileClient.AcquireAccessToken($currentAzureContext.Subscription.TenantId)
$BearerToken = "Bearer " + $token.AccessToken

$Properties = @{
    managedResourceGroupId = "/subscriptions/$SubscriptionID/resourceGroups/$ManagedResourceGroupName" 
}
$sku = @{
    name = 'premium'
}

$Request = @{
    'location' = $Location
    'sku' = $sku
    'name' = $WorkspaceName
    'Properties' = $Properties
}

$Body = ConvertTo-Json -InputObject $Request -Depth 5

$Endpoint = "https://management.azure.com/subscriptions/$SubscriptionID/resourceGroups/$ResourceGroupName/providers/Microsoft.Databricks/workspaces/$($WorkspaceName)?api-version=2018-04-01"

$params = @{
    ContentType = 'application/json'
    Headers = @{'Authorization'=$BearerToken}
    Method = 'PUT'
    Body = $Body
    URI = $Endpoint
}

Invoke-RestMethod @params
 ```

I hope that is useful to someone.
