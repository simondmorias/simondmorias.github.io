---
author: Simon D'Morias
date: '2019-09-20 18:23:17'
draft: false
title: Series - Developing a PySpark Application
type: post
url: /blog/2019/9/20/series-developing-a-pyspark-application
tags:
- Databricks-Connect
- Databricks
- PySpark
- Azure
- Azure DevOps
---

This is a series of blog post to demonstrate how PySpark applications can be developed specifically with Databricks in mind. Though the general principal applied here can be used with any Apache Spark setup (not just Databricks).

Python has many complexities with regards to paths, packaging and deploying versions. These blogs posts are what we have learnt working with clients to build robust, reliable development processes.

The goal is to have a local development environment but using Databricks-Connect to execute against, and then a solid CI and testing framework to support the development going forward using Azure DevOps pipelines. And then to provide a [sample project](https://github.com/DataThirstLtd/Databricks-Connect-PySpark) demonstrating this.

This series can also be used to build more general libraries that can be shared amongst your users to ensure you have common tools across the teams.

We will be using:  

* Conda  
* Databricks-Connect  
* Databricks  
* Visual Studio Code  
* Azure DevOps

Parts:  

* [Part 1 - Developing in Visual Studio Code](/blog/2019/9/20/building-pyspark-applications-as-a-wheel)  
* [Part 2 - Adding Tests](/blog/2019/9/20/part-2-developing-a-pyspark-application)  
* [Part 3 - Packaging into a Wheel](/blog/2019/9/20/part-3-developing-a-pyspark-application)  
* [Part 4 - CI Build in Azure DevOps](/blog/2019/9/20/part-4-developing-a-pyspark-application)  
* [Part 5 - Configuration & Releasing](/blog/2019/9/20/part-5-developing-a-pyspark-application)
