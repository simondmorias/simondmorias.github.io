---
author: Simon D'Morias
date: '2019-09-20 18:26:32'
draft: false
title: Part 5 - Developing a PySpark Application
type: post
url: /blog/2019/9/20/part-5-developing-a-pyspark-application
tags:
- PySpark
- Azure DevOps
- Building
- Testing
---

This is the 5th and final part of a [series](/blog/2019/9/20/series-developing-a-pyspark-application) of posts to show how you can develop PySpark applications for Databricks with Databricks-Connect and Azure DevOps.   All source code can be found [here](https://github.com/DataThirstLtd/Databricks-Connect-PySpark).

## Configuration & Releasing

We are now ready to deploy. I’m working on the assumption we have two further environments to deploy into - UAT and Production.

## Deploy.ps1

This script in the root folder will do all the work we need to release our Wheel and setup some Databricks Jobs for us. You do not need to use Jobs - you can use Azure Data Factory instead if you prefer, the passing of parameters is identical to this method.

The deploy script makes use of our [azure.databricks.cicd.tools](https://github.com/DataThirstLtd/azure.databricks.cicd.tools) which I highly recommend you take a play with if you haven’t already.

You can execute this script from your local computer assuming you have built the Wheel and executed the Build.ps1 scripts first. You should note that if you do this the version on the Wheel will remain as 0.0.1 - if you deploy for a second time you MUST restart the cluster in order for it to pickup the new version.

## Azure DevOps Release Pipeline

We can now create a Release to pickup our build artefacts. Again this is public and can be viewed here. The overall pipeline looks like this:

 















  

    
  
    



      

      
        
          
        
        

        
          
            
          


            


             
![pipeline.png](/images/pipeline.png)

            


          


        
          
        

        
      
        
      

    


  


  


 

Here I have created the two environments, and set the artefacts to be the output of our CI build. I have also created two variables named DatabricksToken and ClusterId which can vary for each environment.

The task for each stage is identical and looks like this:

 















  

    
  
    



      

      
        
          
        
        

        
          
            
          


            


             
![Deploy.png](/images/Deploy.png)

            


          


        
          
        

        
      
        
      

    


  


  


 

I have selected the Deploy.ps1 script and set the parameters to pass in my variables.

I can now run the deployment and check the results.

## Validation

Firstly we can check the files have deployed to DBFS correctly using a notebook:

















  

    
  
    



      

      
        
          
        
        

        
          
            
          


            


             
![dbutils_validate.png](/images/dbutils_validate.png)

            


          


        
          
        

        
      
        
      

    


  


  




Notice that the BuildId has been inject into the filename for the Wheel. Our config file has been deployed which we can validate is for UAT by running this command:
    
```dbutils.fs.head('/DatabricksConnectDemo/Code/config.json')```

And lastly we should be able to see (and execute) our Jobs in the Jobs screen, again notice the BuildId in the Library reference of Job:

















  

    
  
    



      

      
        
          
        
        

        
          
            
          


            


             
![databricks_job.png](/images/databricks_job.png)

            


          


        
          
        

        
      
        
      

    


  


  




Also note how the arguments are passed - ADF works in the same way.

## Lastly

You can also use your library in notebooks. For example, this code will display the version of the current code:
    
```dbutils.library.install("dbfs:/DatabricksConnectDemo/Code/pipelines-0.0.766-py3-none-any.whl")
    dbutils.library.restartPython()
    import pipelines
    print(pipelines.__version__)```

And you can execute the pipelines directly:
    
```from pipelines.jobs import amazon
    amazon.etl()```

Both of those gives these outputs:

















  

    
  
    



      

      
        
          
        
        

        
          
            
          


            


             
![useWheelInNotebook.png](/images/useWheelInNotebook.png)

            


          


        
          
        

        
      
        
      

    


  


  




That’s it - I hope you found this useful.

[Back to Series](/blog/2019/9/20/series-developing-a-pyspark-application)
