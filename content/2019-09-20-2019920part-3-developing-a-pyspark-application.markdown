---
author: Simon D'Morias
date: '2019-09-20 18:26:05'
draft: false
title: Part 3 - Developing a PySpark Application
type: post
url: /blog/2019/9/20/part-3-developing-a-pyspark-application
tags:
- PySpark
- Azure DevOps
- Building
---

This is the 3rd part of a [series](/blog/2019/9/20/series-developing-a-pyspark-application) of posts to show how you can develop PySpark applications for Databricks with Databricks-Connect and Azure DevOps.   All source code can be found [here](https://github.com/DataThirstLtd/Databricks-Connect-PySpark).

## Packaging into a Wheel

Before we can create a CI process we should ensure that we can build and package the application locally. By reusing the same scripts locally and on CI server minimises the chances of something breaking.

Python has a package called setup that will create a [Wheel](https://pythonwheels.com/)for you.

The only requirement is the creation of a setup.py file in the root folder of the project. I have create with this:
    
```
from setuptools import setup

setup(name='pipelines',
      version='0.0.1',
      description='Sample PySpark Application for use with Databricks Connect',
      url='http://github.com/dataThirstLtd',
      author='Data Thirst Ltd',
      author_email='hello@datathirst.net',
      packages=['pipelines', 'pipelines.utils', 'pipelines.jobs'],
      zip_safe=False)
```


Most of these fields you can complete yourself as you see fit. But a few notes:  

* Leave the Version as 0.0.1 - we will set this in the CI process using a replace.  
* Packages should be equal to the folders you want to make available for importing.

Now that we have this file we can run the following command to create the package (from Anaconda prompt):
    
```python setup.py bdist_wheel```

Executing this will generate several folders in your project (dist, build and pipelines.egg-info). But if you look in the .vscode folder and the settings.json file you will see I have hidden these folders - as I find it easier to see what is going on without them in the way.

## Build.ps1

There is a PowerShell script to copy the required files into a bin folder ready to be published. This script has a parameter for the environment name to pick the correct config.json file in the config folder.

At the end it also deletes all the folder created by the Wheel as these are no longer required.

From your Anaconda prompt window (or VSCode Terminal) you can run .\Build.ps1 to execute the script.

## Next

[Create the CI Build](/blog/2019/9/20/part-4-developing-a-pyspark-application)

[Back to the Series Intro](/blog/2019/9/20/series-developing-a-pyspark-application)
