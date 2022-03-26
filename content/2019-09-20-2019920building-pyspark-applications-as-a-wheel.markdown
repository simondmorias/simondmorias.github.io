---
author: Simon D'Morias
date: '2019-09-20 18:24:01'
draft: false
title: Part 1 - Developing a PySpark Application
type: post
url: /blog/2019/9/20/building-pyspark-applications-as-a-wheel
tags:
- PySpark
- Azure DevOps
---

This is the 1st part of a [series](/blog/2019/9/20/series-developing-a-pyspark-application) of posts to show how you can develop PySpark applications for Databricks with Databricks-Connect and Azure DevOps.  All source code can be found [here](https://github.com/DataThirstLtd/Databricks-Connect-PySpark).

## Overview

The goal of this post is to be able to create a PySpark application in Visual Studio Code using Databricks-Connect. This post focuses on creating an application in your local Development environment. Other posts in the series will look at CI & Testing.

Python has a packaging method known as [Wheels](https://pythonwheels.com/). These are packages that can be installed using pip from either a public repository like Pypi or a private repository.  Using the sample application on [GitHub](https://github.com/DataThirstLtd/Databricks-Connect-PySpark) we will create a project that can be processed into a Wheel  which can be versions and pushed to a Databricks cluster.

## Project Structure

The project for the Python application which we will wrap into a Wheel consists of this structure:
    
```\
    —configs
    —pipelines
    ——jobs
    ——utils
    —tests
    main.py
    setup.py
    simpleExecute.py```

The configs directory stored json config files for each environment we will deploy into. The only file read is ever config.json - is this is the active config. To swap in the prod config we would rename prod.config.json to config.json.

The pipelines folder is the main application, note that in line with Python Wheels each folder has a __init____.py file inside it. The jobs subfolder contain the actual pipeline jobs we want to execute - these consist of an etl() method that will be called.

The utils folder holds common shared scripts that we can reuse. Configmanagement.py for example reads the config file. We then have a tests folder for our unit and integration tests that we will run with pytest later.

There are also some PowerShell scripts in the root - we will cover these later in the build and release process.

Take some time to explore the pipelines folder and the functions within them. If you are at all familiar with PySpark it should seems fairly normal. 

## Setup Environment

In the root folder follow the [Readme.md](https://github.com/DataThirstLtd/Databricks-Connect-PySpark) to setup a virtual environment.

## Running Locally

So the first thing to do is run scripts from our local computer but against the Databricks cluster. Databricks-Connect makes this possible.  Firstly it is important to note you cannot just open a script inside the pipelines folder and press F5. Due to the way Python resolves paths this doesn’t work out of the box. You can mess around with your PATH environment variable to get this working - but I suggest not, instead just call your scripts from another script outside of the pipelines folder.

The simpleExecute.py script is for exactly this purpose. Use this scripts for testing your pipelines. Generally I would not commit this script (using gitignore) but I have included in the repo for illustration purpose.

Open the simpleExecute.py script and execute it ensuring you are in your Virtual Environment. This will execute the amazon etl function. All going well after a few seconds you should see this output:

















  

    
  
    



      

      
        
          
        
        

        
          
            
          


            


             
![simpleExecution.png](/images/simpleExecution.png)

            


          


        
          
        

        
      
        
      

    


  


  




You can now run any pipeline or test from this script. You can add breakpoints and debug the pipelines as needed.

## Main.py

Whilst we are not using this script yet it’s a good idea to take a look at this file now. It’s not possible for a Databricks job or Azure Data Factory to execute a script directly inside a Wheel. Instead you execute another script that calls the Wheel. This is what main.py is for. Our job will execute this script passing in arguments. The first argument must be the name of the pipeline job we want to execute. Any subsequent arguments will be passed into the etl() method as parameters.

## Next

[Adding Testing](/blog/2019/9/20/part-2-developing-a-pyspark-application)

[Back to the Series Intro](/blog/2019/9/20/series-developing-a-pyspark-application)
