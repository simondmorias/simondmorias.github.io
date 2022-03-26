---
author: Simon D'Morias
date: '2019-09-20 18:26:13'
draft: false
title: Part 4 - Developing a PySpark Application
type: post
url: /blog/2019/9/20/part-4-developing-a-pyspark-application
tags:
- PySpark
- Azure DevOps
---

This is the 4th part of a [series](/blog/2019/9/20/series-developing-a-pyspark-application) of posts to show how you can develop PySpark applications for Databricks with Databricks-Connect and Azure DevOps.   All source code can be found [here](https://github.com/DataThirstLtd/Databricks-Connect-PySpark).

## Create a CI Build

Now that we have everything running locally we want to create a CI process to build our Wheel, publish that as an artefact and of course to test our code.

In the root of the project is a file called azure-pipelines.yml. This Yaml file describes the build process that takes place.

We will step through the file here to understand what is happening.

```
pool:
  name: Hosted Ubuntu 1604
variables:
  python.version: '3.5'
```

First we target a Ubuntu agent, this will work on Windows but will be horribly slow as installing the dependencies on Windows is very slow. As Databricks uses Python 3.5 we are setting a variable to force the use of this version specifically.
    
```
- pwsh: |
   $s = (Get-Content configs/.databricks-connect).replace('<Token>',"$(DatabricksToken)").replace('<ClusterId>',"$(ClusterId)").replace('<OrgId>',"$(DatabricksOrgId)").replace('<location>',"$(Location)")
   
   $s | Set-Content configs/.databricks-connect
   
   Copy-Item "configs/int.config.json" -Destination "configs/config.json"
  displayName: 'Configure Environment'
  ```


This is a PowerShell core to configure Databricks-Connect by inserting variables (that you must define in your build) into the config file. It also copies the Int environment config file to the default config.json as this is the configure we want to use in our tests. We can still deploy this build to other environments later.
    
```
- task: CopyFiles@2
  displayName: 'Copy Files to: /home/vsts/'
  inputs:
    SourceFolder: configs
    Contents: |
     .databricks-connect
    TargetFolder: /home/vsts/
    OverWrite: true
```



The above step copies the Databricks-Connect config file to the root of the agent users home directory as this is a system default location to look for the file.

```
- script: |
   python -m pip install --upgrade pip && pip install -r requirements.txt 
  displayName: 'Install dependencies'

- script: 'pip install pytest && pytest tests --doctest-modules --junitxml=junit/test-results.xml'
  displayName: pytest
  continueOnError: true
```



We now install the dependencies and execute our pytest unit tests. Note that we are outputting the test results in junit format which Azure Devops recognises and can publish.
    
```
- pwsh: |
   Write-Output "Setting Build Number 0.0.$(Build.BuildID)"
   $s = (Get-Content "./pipelines/__init__.py").replace("0.0.1","0.0.$(Build.BuildID)") 
   $s | Set-Content "./pipelines/__init__.py"
   
   $s = (Get-Content "./setup.py").replace("0.0.1","0.0.$(Build.BuildID)") 
   $s | Set-Content "./setup.py"
  displayName: 'Set Build Number in files'
```



Before we build the Wheel we need to set the build number in the root init file and in the setup.py files. This ensures we can track what version of code we are running later.
    
```
- script: |
   python setup.py bdist_wheel
  displayName: 'Build Wheel'

- task: PowerShell@2
  displayName: 'Run Build.ps1'
  inputs:
    targetType: filePath
    filePath: ./Build.ps1
    pwsh: true
```


Lastly we  execute the same Build.ps1 script to copy the files needed. These will be published as the output artefact and available to download at the end of the build.

The output of the build with test results looks like this:

















  

    
  
    



      

      
        
          
        
        

        
          
            
          


            


             
![build.png](/images/build.png)

            


          


        
          
        

        
      
        
      

    


  


  




This build is available publicly to see the results here: [https://dev.azure.com/datathirst/Public-Demos/_build?definitionId=37&_a=summary](https://dev.azure.com/datathirst/Public-Demos/_build?definitionId=37&_a=summary)

## Next

[Create the Release](/blog/2019/9/20/part-5-developing-a-pyspark-application)

[Back to the Series Intro](https://datathirst.squarespace.com/blog/2019/9/20/series-developing-a-pyspark-application)
