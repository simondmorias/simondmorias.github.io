---
author: Simon D'Morias
date: '2018-09-13 11:51:36'
draft: false
title: Databricks CI/CD Tools
type: post
url: /blog/2018/9/13/databricks-cicd-tools
tags:
- Databricks
- DevOps
- PowerShell
---

A while back now I started to create some PowerShell modules for assisting with DevOps CI and CD scenarios.

These can be found on GitHub [here](https://github.com/DataThirstLtd/azure.databricks.cicd.tools).

## Why?

Firstly, the one thing I don’t like about Databricks is the CI/CD support. I think it is very lacking in support for data engineers and too focused on data science. Don’t get me wrong, Databricks is great - but it is also relatively young as a product and still ironing out the user experience part. Historically it has been all about making the Spark engine better, then focused on the Notebooks principle for exploring data. I’m sure full developer IDE type support will come in the future.

## Source Control

The Databricks UI has support for adding Notebooks to source control, currently just GitHub and BitBucket - odd that VSTS/Azure DevOps is missing. The problem with this solution is that it is per notebook. I would rather sync a folder/repo - this ensures consistency is maintained between dependant items. It would also allow me to have several environments which I could control releases to.

So I started these tools.

The first command is the Export-DatabricksFolder module - this exports a folder of Notebooks from Databricks to a local folder on your desktop. This can be a folder inside a git repo. The export is a plain text version of the Notebook source.

The idea is you continue to work in Notebooks, when you are happy with your changes you run this command and then commit the scripts to git.

Once you are happy to release the code into your Test/Staging.UAT/Production environment you can use the Import-DatabricksFolder command. This does the reverse and deploys your scripts from source control to your Databricks workspace.

## About

These PowerShell commandlets are a wrapper to the [Databricks REST API](https://docs.azuredatabricks.net/api/latest/index.html) which is fairly simple - but also slightly limited. You can’t export file patterns for example.

There are plenty more commands available from the API which I have not exposed (yet). The intention is to add more over time.

## VSTS/Azure DevOps

To make the tools more accessible I have published the tools on the [VSTS Marketplace](https://marketplace.visualstudio.com/items?itemName=DataThirstLtd.databricksDeployScriptsTasks). This allows you to deploy scripts as part of your deployment pipelines. The source code to this is also available on [GitHub](https://github.com/DataThirstLtd/databricks.vsts.tools).
