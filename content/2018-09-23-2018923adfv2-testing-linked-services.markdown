---
author: Simon D'Morias
date: '2018-09-23 12:13:38'
draft: false
title: ADFv2 - Testing Linked Services
type: post
url: /blog/2018/9/23/adfv2-testing-linked-services
tags:
- PowerShell
- ADF
- REST API
- Azure
---

## Azure Data Factory v2 Linked Services Testing

If you are deploying lots of Linked Services to an environment it would be nice to run a test that proves they connect successfully. This can validate many things, including:  * Key Vault Secrets have also been deployed  * Permissions applied to your secrets  * Integration Runtime is up and working  * Firewall ports opened  * User Permissions deployed

## PowerShell

PowerShell and the Azure REST API to the rescue. Firstly you will need a function I [previously blogged about called getBearer](/blog/2018/9/23/powershell-and-azure-rest-api-authentication) to get a Bearer token.

I’m going to assume that you want to run this script in a non interactive mode as part of a Release Pipeline through some like Azure DevOps. This first function will ensure we are connected to Azure:
    
```
function connect($SubscriptionID)
{
  if($null -eq (Get-AzureRMContext).Account)
  {
    Connect-AzureRmAccount
    Select-AzureRmSubscription -Subscription $SubscriptionID
  }
}
```
    




If you are running locally in PowerShell this ask you to login (ONCE!), if you are running as Azure PowerShell it will do nothing because you are already connected.

Next we need to get your Linked Service as JSON. The REST API has a nice method to do this:

```
function getLinkedService([string]$LinkedService)
{
  $ADFEndpoint = "https://management.azure.com/subscriptions/$SubscriptionId/resourceGroups/$ResourceGroup/providers/Microsoft.DataFactory/factories/$DataFactoryName/linkedservices/$($LinkedService)?api-version=2018-06-01"

  $params = @{
      ContentType = 'application/json'
      Headers = @{'accept'='application/json';'Authorization'=$BearerToken}
      Method = 'GET'
      URI = $ADFEndpoint
  }

  $a = Invoke-RestMethod @params
  Return ConvertTo-Json -InputObject @{"linkedService" = $a} -Depth 10
}
```



Now the interesting bit, this API call I had to find using the web UI and [Fiddler](https://www.telerik.com/fiddler). So yes this is an undocumented API call. Note that the API version shows it as being in preview. Hopefully it will remain unchanged:
    
```
function testLinkedServiceConnection($Body)
{

  $AzureEndpoint = "https://management.azure.com/subscriptions/$SubscriptionID/resourcegroups/$ResourceGroup/providers/Microsoft.DataFactory/factories/$DataFactoryName/testConnectivity?api-version=2017-09-01-preview"

  $params = @{
      ContentType = 'application/json'
      Headers = @{'accept'='application/json';'Authorization'=$BearerToken}
      Body = $Body
      Method = 'Post'
      URI = $AzureEndpoint
  }

  Return (Invoke-RestMethod @params).succeeded
}
```



Put it altogether:
    
```
$ClientID       = "d8e4efe0-f335-4da8-9adb-38942dca3783" 
$ClientSecret   = "mysecret" 
$DataFactoryName = "dataThirstADFv2Test1"
$ResourceGroup  = "dataThirstTestBed8"
$SubscriptionID = "d4c56897-10e7-4c62-a139-326bcf267f68"
connect $SubscriptionID
$BearerToken = getBearer "bb0280d2-a9cf-46e8-9485-19d65b1b2c84" $ClientID $ClientSecret
$LinkedServiceBody = getLinkedService "LS_DataThirst_TestBed8_ADLS"
testLinkedServiceConnection $LinkedServiceBody
```

The final output should be a True or False based on whether it connected or not.

The full script can be found [here on GitHub](https://gist.github.com/simondmorias/e4d9e7265a20af8d7d17b92d483e3889).
