---
author: Simon D'Morias
date: '2019-07-18 10:38:14'
draft: false
title: Databricks PowerShell Tools Update 1.1.21
type: post
url: /blog/2019/7/18/databricks-powershell-tools-update-1121
tags:
- Databricks
- PowerShell
---

A new release of [azure.databricks.cicd.tools](https://www.powershellgallery.com/packages/azure.databricks.cicd.tools/1.1.21) has gone out today.

Changes include:  

* Support for Cluster Log Path for:
* New-DatabricksCluster
* Add-DatabricksJarJob
* Add-DatabricksNotebookJob
* Add-DatabricksPythonJob  
* Support for Instance Pool ID on the above as well
* Support from creating new instance pools will come soon  
* New Command Restart-DatabricksCluster  
* New Command Remove-DatabricksLibrary  
* Add -RunImmediate to Add-DatabricksNotebookJob  
* Fixed case sensitive issue preventing all commands importing on Linux

All the docs have been updated in the [wiki ](https://github.com/DataThirstLtd/azure.databricks.cicd.tools/wiki)as well.

Please report any [issues](https://github.com/DataThirstLtd/azure.databricks.cicd.tools/issues).
