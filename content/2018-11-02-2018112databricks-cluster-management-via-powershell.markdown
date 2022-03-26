---
author: Simon D'Morias
date: '2018-11-02 15:27:59'
draft: false
title: Databricks Cluster Management via PowerShell
type: post
url: /blog/2018/11/2/databricks-cluster-management-via-powershell
tags:
- Databricks
- PowerShell
---

We have released a big update to the CI/CD Tools on GitHub today: [https://github.com/DataThirstLtd/azure.databricks.cicd.tools](https://github.com/DataThirstLtd/azure.databricks.cicd.tools)

These updates are for cluster management within Databricks. They allow for you to Create or Update Clusters. Stop/Start/Delete and Resize.

There are also some new helper functions to get a list of available Spark versions and types of VM’s available to you.

The full set of new commands is:  

* Get-DatabricksClusters - Returns a list of all clusters in your workspace  
* New-DatabricksCluster - Creates/Updates a cluster  
* Start-DatabricksCluster  
* Stop-DatabricksCluster  
* Update-DatabricksClusterResize - Modify the number of scale workers  
* Remove-DatabricksCluster - Deletes your cluster  
* Get-DatabricksNodeTypes - returns a list of valid nodes type (such as DS3v2 etc)  
* Get-DatabricksSparkVersions - returns a list of valid versions

These will hopefully be added to the [VSTS/Azure DevOps tasks](https://marketplace.visualstudio.com/items?itemName=DataThirstLtd.databricksDeployScriptsTasks) in near future. In the meantime you can use them by including PowerShell scripts in builds.

If you find any issues please raise an Issue on GitHub! Also contributors are welcomed if you have ideas for them.
