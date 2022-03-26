---
author: Simon D'Morias
date: '2018-11-30 16:04:22'
draft: false
title: PySpark Applications for Databricks
type: post
url: /blog/2018/11/30/pyspark-applications-for-databricks
tags:
- Databricks
- PySpark
- DevOps
---

**UPDATE April 2019**: If you are interested in creating a PySpark application for Databricks you should consider using Databricks-Connect. More details [here](/blog?tag=Databricks-Connect).

Whilst notebooks are great, there comes a time and place when you just want to use Python and PySpark in it’s pure form. Databricks has the ability to execute Python jobs for when notebooks don’t feel very enterprise data pipeline ready - %run and widgets just look like schoolboy hacks. Also the lack of debugging in Databricks is painful at times. By having a PySpark application we can debug locally in our IDE of choice (I’m using VSCode). 

For some reason Python Jobs are not available in the Workspace UI today (but is available in the REST API and when executing via Azure Data Factory). The Workspace UI does have the ability to use Spark-submit jobs and Python, which oddly the Azure Data Factory tasks do not support. These inconsistencies make it hard to get started.

















  

    
  
    



      

      
        
          
        
        

        
          
            
          


            


             
![Screenshot 2018-11-30 at 14.27.58.png](/images/Screenshot+2018-11-30+at+14.27.58.png)

            


          


        
          
        

        
      
        
      

    


  


  




## Developing in PySpark Locally

We are going to create a project that structurally looks like this the image on the right. The full project is available on [GitHub](https://github.com/DataThirstLtd/databricks.pyspark.application.demo).

This article will leave spark-submit for another day and focus on Python jobs. I will also assume you have PySpark working locally. We will focus on developing a PySpark application that you can execute locally and be debugged, and also deploy to a Databricks cluster with no changes. 

## The Differences

When developing PySpark you need to be aware of the differences between notebooks and the differences between developing locally to a cluster. The key differences are summarised here:


<table class="tg" >
  <tr >
    Feature
    Notebooks
    Python Job
    Local PySpark
  </tr>
  <tr >
    
<td class="tg-8kr4" >spark.SparkSession
</td>
    
<td class="tg-8kr4" >Already available
</td>
    
<td class="tg-8kr4" >Must find manually
</td>
    
<td class="tg-8kr4" >Must create manually
</td>
  </tr>
  <tr >
    
<td class="tg-8kr4" >dbutils
</td>
    
<td class="tg-8kr4" >Already available
</td>
    
<td class="tg-8kr4" >Already available
</td>
    
<td class="tg-8kr4" >Never available
</td>
  </tr>
  <tr >
    
<td class="tg-8kr4" >Import Notebooks
</td>
    
<td class="tg-8kr4" >via %run
</td>
    
<td class="tg-8kr4" >Not possible
</td>
    
<td class="tg-8kr4" >Not possible
</td>
  </tr>
  <tr >
    
<td class="tg-8kr4" >Import other modules
</td>
    
<td class="tg-8kr4" >Using libraries only
</td>
    
<td class="tg-8kr4" >If in PYTHONPATH
</td>
    
<td class="tg-8kr4" >via PYTHONPATH or   
relative paths
</td>
  </tr>
  <tr >
    
<td class="tg-8kr4" >Widgets
</td>
    
<td class="tg-8kr4" >Yes
</td>
    
<td class="tg-8kr4" >No
</td>
    
<td class="tg-8kr4" >No
</td>
  </tr>
</table>

Using PySpark we must getOrCreate the spark.SparkSession on each script. That is pretty simple:
    
```
from pyspark.sql import SparkSession
spark = SparkSession.builder.getOrCreate()
```
    




The only thing to remember here is that the session must be called spark (lowercase).

Dbutils is handy in notebooks. And it is available in Python jobs, but it is not available to download and use as a local module. This means we want to avoid using it in our jobs. I have managed to do this with the exception of getting secrets which is a must have on the Databricks cluster unless you want to pass around plaintext passwords (which you don’t right?). 

An important point to remember is to never run import dbutils in your Python script. This command succeeds but clobbers all the commands so nothing works. It is imported by default.

One other key difference is that you are probably not running Hadoop locally, although you can if you really want to. This means that you cannot mount/connect to Data Lake or Blob storage accounts. As this is local development - I recommend having a sample of files locally and essentially mocking your cloud storage. As a bonus put these in your repo - it makes writing tests so much simpler - in the sample project I have created a subfolder called DataLake.

## Importing other Modules

Like my sample project the chances are that you will have several .py files containing the code you want to execute. It wouldn’t be good practice to create a PySpark application in a single file. You will split out helpers etc into other files. These need to be imported into the executing script.

Before we get too far into that it import to understand PYTHON_PATH is an environment variable which contains a list of locations to look for module when you run the import command. It is safer to always set this variable rather than rely on relative paths that may not work on clusters (and won’t on a Databricks cluster).

There is a simple fix for this, which allows you to add full paths from a relative path:
    
```
dirname = os.path.dirname(__file__)
sys.path.insert(0, (os.path.join(dirname, 'Utils')))
```

This adds a subfolder called Utils to my PYTHON_PATH variable so that when I execute import helpersfunctions (which is helperfunctions.py in the Utils folder) it succeeds.

## Parameters

When you execute your application you will probably want to pass in some parameters such as a file paths, dates to process etc. To do this we will use a library called argparse. This sample code reads in two arguments called job and slot.
    
```
parser = argparse.ArgumentParser()
parser.add_argument("job", type=str, nargs='?', default="Job1.MyMethod")
parser.add_argument("slot", type=str, nargs='?', default="2018/11/19")
args = parser.parse_args()
job_module, job_method = args.job.rsplit('.',1)
slot = args.slot
```


The codes exists in the main.py file which will be the script our ADF pipeline or Python job will execute. The job parameter will tell it which module and method to execute, the slot is just a sample parameter.

By using the job parameter this script can be reused to execute any module and method (handy if you want loop in ADF with creating hundreds of ADF pipeline tasks).

It will be execute using this code:

```
from importlib import import_module
mod = import_module(job_module)
met = getattr(mod, job_method)
met(slot)
```

## Configuration

Lastly we need some configuration to handle the differences between local and the Databricks cluster environment. I’m setting spark.conf in script, but you can also set this from ADF or in the Job:
    
 ```
 if "local" in spark.sparkContext.master:
    dirname = os.path.dirname(__file__)
    sys.path.insert(0, (os.path.join(dirname, 'Utils')))
    sys.path.insert(0, (os.path.join(dirname, 'Jobs')))
    spark.conf.set("ADLS",os.path.join(dirname, 'DataLake'))
else:
    spark.sparkContext.addPyFile("dbfs:/MyApplication/Code/scripts.zip")
    spark.conf.set("ADLS",'adl://myazuredatalake.azuredatalakestore.net/')
    spark.conf.set("dfs.adls.oauth2.access.token.provider.type", "ClientCredential")
    spark.conf.set("dfs.adls.oauth2.client.id", dbutils.secrets.get(scope = "SparkADLS - Secrets", key = "clientid"))
    spark.conf.set("dfs.adls.oauth2.credential", dbutils.secrets.get(scope = "SparkADLS - Secrets", key = "credential"))
    spark.conf.set("dfs.adls.oauth2.refresh.url", "https://login.microsoftonline.com/[tenantid]/oauth2/token")
 ```

The ADLS is the path to my Data Lake which is a local path when the Spark context is local and the Azure Data Lake when I’m on the cluster. I’m also using dbutils when in Databricks to get the secret connection details for the Lake.

## Execute Locally

You should now understand what is happening in the [sample repo](https://github.com/DataThirstLtd/databricks.pyspark.application.demo). So much you can execute main.py and understand what is happening. At this point I would suggest doing that if you have not already. Now you can also add breakpoints and debug your code (yay!).

## Build & Deploy

When Databricks executes jobs it copies the file you specify to execute to a temporary folder which is a dynamic folder name. Unlike Spark-submit you cannot specify multiple files to copy. The easiest way to handle this is to zip up all of your dependant module files into a flat archive (no folders) and add the zip to the cluster from DBFS. 

Eagle eyed readers may have noticed the line spark.sparkContext.addPyFile("dbfs:/MyApplication/Code/scripts.zip") in the last code snippet. This zip file contains our dependant modules. the addPyFile command allows all servers in the cluster to see the file.

All code must be deployed to DBFS in advance of your job running. I have created two scripts to handle this, build.ps1 and deploy.ps1 (yes PowerShell - Mac users don’t freak out I use a Mac too - get PowerShell Core - all is cool). 

Build.ps1:
    
```
Set-Location $PSScriptRoot
Remove-Item ./bin -Recurse -Force -ErrorAction:SilentlyContinue
New-Item ./bin -ItemType Directory -Force | Out-Null
Copy-Item "Jobs/*.py" ./bin
Copy-Item "Utils/*.py" ./bin
$source = Resolve-Path ./bin/*.py
$ZipFilePath = "./bin/scripts"
Compress-Archive -LiteralPath $source -DestinationPath $ZipFilePath 
Remove-Item ./bin/*.py -Force
Copy-Item "./main.py" ./bin
```

The build.ps1 script creates a bin directory (add to .gitignore) that contains the main script we will execute and a zip file of the dependant scripts.

The next script is deploy.ps1 which will upload the two files to DBFS where you can execute it from:
    
 ```
Set-Location $PSScriptRoot
./build.ps1

if (!(Get-Module -ListAvailable -Name azure.databricks.cicd.Tools)) {
    Install-Module azure.databricks.cicd.Tools -Force -Scope CurrentUser
}
Import-Module -Name azure.databricks.cicd.Tools

$BearerToken = Get-Content -Path ./MyBearerToken.txt -Raw # Create this file with your bearer tokem and add to gitignore
$Region = "westeurope"
$localBinfolder = Join-Path $PSScriptRoot "/bin/"
$TargetDBFSFolderCode = "/MyApplication/Code"

# Clean Target Folder
Remove-DatabricksDBFSItem -BearerToken $BearerToken -Region $Region -Path $TargetDBFSFolderCode

# Upload files to DBFS
Add-DatabricksDBFSFile -BearerToken $BearerToken -Region $Region -LocalRootFolder $localBinfolder -FilePattern "main.py"  -TargetLocation $TargetDBFSFolderCode -Verbose
Add-DatabricksDBFSFile -BearerToken $BearerToken -Region $Region -LocalRootFolder $localBinfolder -FilePattern "*.zip"  -TargetLocation $TargetDBFSFolderCode -Verbose
 ```

Note you will need to create the file MyBearerToken.txt in the same folder with your bearer token in. Also set the region to your region.

## Execute as a Python Job

Using PowerShell you can create a Python job (the Databricks does not let you create Python jobs, but you can view them):
    
 ```
Set-Location $PSScriptRoot
Import-Module azure.databricks.cicd.Tools 
$BearerToken = Get-Content "MyBearerToken.txt"  # Create this file in the Tests folder with just your bearer token in
$Region = "westeurope"

$JobName = "MyApplication-Test-PythonJob"
$SparkVersion = "4.1.x-scala2.11"
$NodeType = "Standard_D3_v2"
$MinNumberOfWorkers = 1
$MaxNumberOfWorkers = 1
$Timeout = 1000
$MaxRetries = 1
$ScheduleCronExpression = "0 15 22 ? * *"
$Timezone = "UTC"
$PythonPath = "dbfs:/MyApplication/Code/Main.py"
$PythonParameters = "Job1.MyMethod", "2018/11/30"

 Add-DatabricksPythonJob -BearerToken $BearerToken -Region $Region -JobName $JobName `
    -SparkVersion $SparkVersion -NodeType $NodeType `
    -MinNumberOfWorkers $MinNumberOfWorkers -MaxNumberOfWorkers $MaxNumberOfWorkers `
    -Timeout $Timeout -MaxRetries $MaxRetries `
    -ScheduleCronExpression $ScheduleCronExpression `
    -Timezone $Timezone -PythonPath $PythonPath `
    -PythonParameters $PythonParameters `
    -PythonVersion 3 
```

Before you execute this on Databricks you will need to create an Azure Data Lake and set the credentials and Secrets.

Once executed you should see the job in Databricks and be able to execute it with Success!

You can also execute from Azure Data Factory using the Databricks Python task. Just point to your script and pass parameters as normal:

 















  

    
  
    



      

      
        
          
        
        

        
          
            
          


            


             
![Screenshot 2018-11-30 at 15.44.31.png](/images/Screenshot+2018-11-30+at+15.44.31.png)

            


          


        
          
        

        
      
        
      

    


  


  


 

Congratulations for making it this far in a very long post, I hope you find it useful. 

The full GitRepo is [here](https://github.com/DataThirstLtd/databricks.pyspark.application.demo).
