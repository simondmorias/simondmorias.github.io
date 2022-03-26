---
author: Simon D'Morias
date: '2019-02-21 16:59:57'
draft: false
title: Databricks Key Vault backed Secret Scopes
type: post
url: /blog/2019/2/21/databricks-key-vault-backed-secret-scopes
tags:
- Databricks
- PowerShell
- Azure
---

A few weeks ago now Databricks added the ability to have Azure Key Vault backed [Secret Scopes](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html). These are still in preview.

Today the PowerShell tools for Databricks we maintain have been updated to support these! Technically the API is not documented, and as the service is in preview stop working in the future. But we will update the module if Databricks change the API.

Example:
    
```
Import-Module azure.databricks.cicd.Tools
$BearerToken = "dapi1234567890"  
$Region = "westeurope"
$ResID = "/subscriptions/{mysubscriptionid}/resourceGroups/{myResourceGroup}/providers/Microsoft.KeyVault/vaults/{myKeyVaultName}}"

Add-DatabricksSecretScope -BearerToken $BearerToken -Region $Region -ScopeName "KVBackedScope" -KeyVaultResourceId $ResID
```

The $ResID shown above is a horrible URI for your Key Vault. You can find it on the Key Vault Properties blade and copy it from there.

Now when you use dbutils to get secrets from this scope you can reference a secret name in the key vault and it will be returned to you. Nice and simple!

More information on the tools: [https://github.com/DataThirstLtd/azure.databricks.cicd.tools](https://github.com/DataThirstLtd/azure.databricks.cicd.tools)
