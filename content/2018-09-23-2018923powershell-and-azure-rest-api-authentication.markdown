---
author: Simon D'Morias
date: '2018-09-23 11:31:28'
draft: false
title: PowerShell and Azure REST API Authentication
type: post
url: /blog/2018/9/23/powershell-and-azure-rest-api-authentication
tags:
- PowerShell
- Azure
- REST API
---

Sometimes you find that the Azure PowerShell commandlets do not offer all of the functionality of the REST API/Portal. In these cases you can fall back to the REST API which can be called from PowerShell of course. 

The first thing you always need to do is authenticate. These scripts will authenticate using a service principle so that they can be used in non-interactive mode. Details of how to create a service principle can be found here: [https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-service-principal-portal](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-service-principal-portal). 

Once you have this you need the ApplicationID (shown on the link above - note that this is interchangeably known as the ClientID or sometimes even as ObjectID). And also the Key (which is basically a password) - again details of how to get this are shown on the link above.

Now the interesting bit. We will create a function to perform the authentication taking in our [TenantID](https://www.whatismytenantid.com/), ClientID and Secret:
    
  ```
function getBearer([string]$TenantID, [string]$ClientID, [string]$ClientSecret)
{
  $TokenEndpoint = {https://login.windows.net/{0}/oauth2/token} -f $TenantID 
  $ARMResource = "https://management.core.windows.net/";

  $Body = @{
          'resource'= $ARMResource
          'client_id' = $ClientID
          'grant_type' = 'client_credentials'
          'client_secret' = $ClientSecret
  }

  $params = @{
      ContentType = 'application/x-www-form-urlencoded'
      Headers = @{'accept'='application/json'}
      Body = $Body
      Method = 'Post'
      URI = $TokenEndpoint
  }

  $token = Invoke-RestMethod @params

  Return "Bearer " + ($token.access_token).ToString()
}
  ```




So firstly you authenticate for a Tenant - which is your Azure Active Directory (AAD) - all of your subscriptions belong to this tenant. In the URL we pass the Tenant ID, then as part of the  Body we pass a set of parameters including the our ClientID and Secret. 

Then output of the function is a string for the bearer token in the format that the REST API expects the token to be passed back in.

Example call:
    
```
$ClientID       = "20f1ca3f-7ff2-4fc3-90eb-4be3da03cc34" 
$ClientSecret   = "mysecret" 
$token = getBearer "e7ba8d12-2cb2-48d5-8186-e3dc7bf7b86d" $ClientID $ClientSecret
Write-Output $token
```



You should get a long string back that looks something like: “Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Imk2bEd…”

(BTW the values in the example above are dummy values!)  

