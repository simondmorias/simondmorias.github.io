---
author: Simon D'Morias
date: '2019-01-18 11:43:25'
draft: false
title: PowerShell for Azure Databricks
type: post
url: /blog/2019/1/18/powershell-for-azure-databricks
tags:
- PowerShell
- Databricks
- Azure
---
            
![powershell_colour.png](/images/powershell_colour.png)

Last year we released a a PowerShell module called azure.databricks.cicd.tools on [GitHub](https://github.com/DataThirstLtd/azure.databricks.cicd.tools) and [PowerShell Gallery](https://www.powershellgallery.com/packages/azure.databricks.cicd.tools/). What we never did is publish anything about what it can do.

The original purpose was to help with CI/CD scenarios, so that you could create idempotent releases in Azure DevOps, Jenkins etc. But now it has almost full parity with the options available in the REST API. 

Databricks do offer a supported CLI (which requires Python installed), and a REST API - which is quite complex to use - but is what this PowerShell module uses. 

The module is also based on PowerShell Core, so works on MacOS and Linux as well as old PowerShell on Windows.

## Advantage of these PowerShell Tools

The CLI and REST API have quite complex requests and not all options are clear - for example if you want to create a Python 3 cluster you create a cluster and set an environment variable which has to be passed in a JSON array. The New-DatabricksCluster has a -PythonVersion flag to handle this for you. 

Another example is that the REST API accepts JSON arrays for many settings, now a problem with PowerShell if you work natively with the REST API is the ConvertTo-Json function removes arrays if you only have one item in it. Again this module deals with this for you.

This is just two examples of many scenarios which as Engineers we really should have to worry about.

## Features

Today the module supports the following functionality:  

* DBFS - Upload/Download files (including large files which is a pain with the API)  
* Notebooks - Import/Export  
* Jobs - Create/Delete/Execute  
* Clusters - Create/Delete/Start/Stop  
* Libraries - Create/Delete  
* Secrets - Create

## Examples

### Install

You can install the module from PowerShell Gallery using this Install-Module command, if you are running as Administrator you can remove the -Scope CurrentUser. You only need to install once - after which you can just use the Import-Module command.

```
Install-Module azure.databricks.cicd.tools -Scope CurrentUser
Import-Module azure.databricks.cicd.tools
```

### Connecting

All commands require you to pass the Azure region your instance is in (this is in the URL of your Databricks workspace - such as westeurope). You will also need an API Bearer token. You can create this in the workspace by clicking on the user icon in the top right corner and selecting User Settings > Generate New Token.

Hold these two values in variables so that you can reuse them in your scripts:
    
```
$BearerToken = “dapi4578234ujkhkjh345hjjkhkj“
$Region = "westeurope"
```

### Create a Cluster

We can now create a new cluster. There are quite a few options, this is just some of them:
    
```
New-DatabricksCluster  -BearerToken $BearerToken `
            -Region $Region `
            -ClusterName "TestCluster" `
            -SparkVersion "4.0.x-scala2.11" `
            -NodeType "Standard_D3_v2" `
            -MinNumberOfWorkers 2 `
            -MaxNumberOfWorkers 10 `
            -AutoTerminationMinutes 90 `
            -PythonVersion 3
```


Now this command is idempotent. So if you run it twice it will search for a cluster with this name and update it to have these settings. Therefore you can source control your cluster setup and make it part of your deployment pipeline. If anyway messes with settings through the UI (tut tut) your next deployment will correct the changes.

## Documentation

All commands are documented on the [GitHub Wiki](https://github.com/DataThirstLtd/azure.databricks.cicd.tools/wiki).

## Why no AWS?

This has intentionally been excluded. We expect the API to drift between the two platforms in the coming months - and this has already started with the SCIM API for Security. Which BTW doesn’t work great yet - which is why it is not implemented.

Therefore as we believe the majority of customers interested in PowerShell will be in Azure we elected not to support AWS now as it will be hard to maintain going forward.
