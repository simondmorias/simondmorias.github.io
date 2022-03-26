---
author: Simon D'Morias
date: '2020-01-24 17:01:12'
draft: false
title: PySpark in Visual Studio Online (Container)
type: post
url: /blog/2020/1/24/pyspark-in-visual-studio-online
tags:
- PySpark
- Containers
- VSCode
- VSOnline
- Visual Studio Online
---

A new product was recently released called [Visual Studio Online](https://visualstudio.microsoft.com/services/visual-studio-online/) (VSOnline). Not to be confused with the old Visual Studio Online (VSO) which was renamed VSTS and then renamed to the now Azure DevOps (that’s a rant for another day).

Update May 2020: Microsoft have renamed it already - its now CodeSpaces! I quite like that name.

VSOnline allows you to create a development environment in a container hosted in Azure for almost [pennies](https://azure.microsoft.com/en-gb/pricing/details/visual-studio-online/). And to be quite blunt it is seriously impressive. In seconds I can have a development environment which is always consistent. I can open in VS Code and even in a browser (which is frankly bonkers). 

Why is this so important?

Well firstly I can have a Linux dev environment on my Windows machine, without a VM - yes WSL2 is coming, but I still hate turning on any virtualisation tech on my laptop - it just causes problems with connected standby and sleeping in general. (I have no scientific proof of this - it just feels that way!)

## New Machine/New Starters

When you get a new PC it is a PIA to setup all the tools you need and to setup a dev environment locally. For new starters on your team it is even worse - they do not have experience of doing it before it can be horribly slow to get up and running. 

This is where containers come in. The container is your environment - all you need is a machine (any operating system) with VS Code or just a browser (I probably wouldn’t use a browser for full time dev - but to check something, or to access it from anywhere, it’s great).

## Screwing up your Machine

There is nothing worse than finding a bug that only happens on your machine. For everyone else it is fine - just YOU. These environment problems can takes hours to solve and lets be honest we have all reinstalled a OS (yes alright - Windows) at some point in our lives to work around something. Containers allow you to refresh an environment back to a configuration with next to no effort.

## Custom Containers

If you just create a new default environment through the  web portal or inside the VSCode (by click on the Green icon in the bottom left corner) you get a default container which has a number of dev tools installed. But in reality you really want to create a custom container with the tools you need. The default for example will not run pyspark by default.

## PySpark Container

So with a bit of messing around I have create a custom container that you can clone here: https://github.com/simondmorias/testing-vsoDocker

You can visit the repo to take a look at what it does. The main files of interest are inside the .devcontainer folder. The Dockerfile uses a base Java image (because that’s hard to install) and then installs Mini Conda and PowerShell Core. It then creates a Conda environment called PySpark using the environment.yml file.

Lastly there is a file called devcontainer.json. This is the file that VSOnline uses. It should be fairly self explanatory. It just points to the Dockerfile and says which VSCode extensions you want installed.

To create the container follow these steps:  

* Click the green icon in the bottom left corner of VSCode  
* Select “Create New Environment”  
* Select “Custom Settings”  
* Paste the GitHub repo URL (https://github.com/simondmorias/testing-vsoDocker)  
* Name the environment  
* Select Standard or Premium (Standard is plenty)  
* Wait about two minutes - sit back and watch the live logging on screen as it builds.  
* When it’s done you will see a “Connect” option in the bottom right of your screen

(Don’t be alarmed if some of the logging comes out red - it looks like an error but it does complete just fine.)

To test it open test.py and press F5 - you should see:

             
![VSOnline-with-pyspark.png](/images/VSOnline-with-pyspark.png)

            
If you are using Databricks-Connect then update the environment.yml file to remove pyspark and add:
    
```
name: pyspark
    channels:
      - conda-forge
    dependencies:
      - python=3.7.3
      - pytest=5.0.1
      - requests=2.22.0
      - pylint
      - setuptools=40.2.0
      - pip:
        - databricks-connect==6.1.0
```

You will need to configure Databricks-Connect after you start the container by running:  databricks-connect configure from the terminal.

## Next Steps

If you want to develop use VSOnline then all you need to do is add the .devcontainer folder to your repo and you should be good to go! SO SIMPLE!!!

UPDATE: [Prebuilt container](https://datathirst.net/blog/2020/6/7/databricks-connect-in-a-container)
