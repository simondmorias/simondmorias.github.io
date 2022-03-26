---
author: Simon D'Morias
date: '2020-06-07 12:03:18'
draft: false
title: Databricks-Connect in a Container
type: post
url: /blog/2020/6/7/databricks-connect-in-a-container
tags:
- Databricks-Connect
- Azure
- VSCode
---

             
![docker.png](/images/docker.png)

Today I have published a set of containers to Docker Hub to enable developers to create Databricks-Connect Dev environments really quickly and consistently.

* Docker Hub: [https://hub.docker.com/r/datathirstltd/dbconnect](https://hub.docker.com/r/datathirstltd/dbconnect)   
* Source Code: [https://github.com/DataThirstLtd/databricksConnectDocker](https://github.com/DataThirstLtd/databricksConnectDocker)

These are targeted at dev using PySpark in VSCode. Though I suspect it will work for Scala and Java development as well.

## Why?

Because setting up Databricks-Connect (particularly on Windows is a PIA). This allow:  

* A common setup between team members  
* Multiple side by side versions 
* Ability to reset your environment  
* Even run the whole thing from a [browser](https://docs.microsoft.com/en-gb/visualstudio/online/how-to/browser)!

## Getting Started  

* Open an empty folder in VS Code  
* Create a directory called .devcontainer  
* Create an empty file in the directory called devcontainer.json  
* Paste this code into the file:
    
```{
        "context": "..",
        "image": "datathirstltd/dbconnect:6.4.1",
    
        "settings": {
            "python.pythonPath": "/opt/conda/envs/dbconnect/bin/python",
            "python.venvPath": "/opt/conda/envs/dbconnect/lib/python3.7/site-packages/pyspark/jars"
        },
    
        //  Optional command - could add your own environment.yml file here (you must keep --name the same)
        // "postCreateCommand": "conda env update --file environment.yml --name dbconnect",
    
        // Rather than storing/committing your bearer token here we recommend using a local variable and passing thru "DATABRICKS_API_TOKEN": "${localEnv:DatabricksToken}",
        // You can manually set these as environment variables if you prefer
        "containerEnv": {
            "DATABRICKS_ADDRESS": "https://westeurope.azuredatabricks.net/",
            "DATABRICKS_API_TOKEN": "dapia12345678901234567890",
            "DATABRICKS_CLUSTER_ID": "0000-11111-hello123",
            "DATABRICKS_ORG_ID": "1234567890",
            "DATABRICKS_PORT": "8787"
        },
        "extensions": [
            "ms-python.python"
        ]
    }  
```

**IMPORTANT: Correct the image tag to the version of Databricks Runtime your cluster is running. Currently we only support Databricks 6+**  * Update the Databricks Variables for your environment  * Optionally add any additional extensions you want to the extensions block.

**IMPORTANT: Changing any setting in the devcontainer.json after the container has been build requires you to rebuild the container for it take effect**

To open using Docker locally:  

* Click on the Green icon in the bottom left of VSCode and select "Reopen in Container"

To open in a CodeSpace:  

* Commit your folder to a repo first  
* Open the Remote Explorer (left hand toolbar)  
* Ensure CodeSpaces is selected in the top drop down  
* Click + (Create new CodeSpace)  * Follow the prompts

The first pull can be a little slow as the image is quite big. But once it is cached rebuilding the container should take just a few seconds.
