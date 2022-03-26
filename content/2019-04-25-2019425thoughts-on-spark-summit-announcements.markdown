---
author: Simon D'Morias
date: '2019-04-25 09:08:45'
draft: false
title: Thoughts on Spark Summit Announcements
type: post
url: /blog/2019/4/25/thoughts-on-spark-summit-announcements
tags:
- Spark
- .Net
- Databricks
- k8s
---

Yesterday saw the Spark Summit keynote take place in San Francisco. There were quite a few announcements, some expected, some not so much. In order of importance to how I see things today this is my summary and view of them.

# Kubernetes

No real fan fare was made about this - and judging by the comments on Twitter people do not seem too excited by it. But for me this is massive - Spark 3.0 will support K8s deployments. The reason this is so important is cluster startup time.

















  

    
  
    



      

      
        
          
        
        

        
          
            
          


            


             
![spark3.jpg](/images/spark3.jpg)

            


          


        
          
        

        
      
        
      

    


  


  




Currently in Azure it takes 7-10 minutes to start a cluster depending on its size and vm type you go for. During this startup time you are charged for the VM and Databricks DBU. Now because the Engineering workloads cluster type has to be started on demand this is quite painful. If you just use Analytics clusters this won’t affect you much - but you shouldn’t be for data pipeline work.

That cost adds up, it tedious for developers, and slows iterations. But it is also encouraging people to use analytics clusters for data pipeline workloads, worst still with the default FAIR scheduler enabled. They then execute several jobs in parallel using Data Factory. This is terrible for performance, particularly memory management - which is why engineering clusters use FIFO (First in First Out) scheduling. But you can’t send multiple jobs to the same engineering cluster.

Which is where k8s can help. If cluster start time can be reduced to seconds using containers then the problem goes away. You can run all of those parallel jobs on separate smaller clusters. The overhead should be minimal.

I suspect it will be a while until we see the option of creating k8s clusters in Databricks - but when we do it will be awesome!

# .Net Spark

No massive surprise here given the interest in Azure Databricks. Microsoft developers need a .[Net solution](https://devblogs.microsoft.com/dotnet/introducing-net-for-apache-spark/) for big data. The demise of Azure Data Lake Analytics (ADLA) left a big hole. With the existence of .Net Core this makes perfect sense and opens Spark up to whole new set of customers and developers. Notably the team behind the old ADLA appear to be involved in this project - that’s great news and maybe even leaves the option of a migration path to Databricks?! (That’s a huge maybe - some people would laugh a lot at me for suggesting this!).

I’m interested to see how this will work out. Firstly the anti Microsoft crowd will fail to take it seriously, thats not a major issue but I do fear it will be treated as second class by the Spark community (Python still is by many). 

More interestingly will be how the use of libraries work.  Some things in Spark do not work well even in Python - a prime example is handling Spatial data. The libraries for Python are near non-existent and terrible, you get forced into using Scala. Even then they are limited and for really complex work you have to drop down to Java to access levels below the RDD. 

How will this work from a .net world? I will assume that like Python you will not be able to access these lower levels. Granted you shouldn’t have to. But I’ve seen that today you do need to for highly performant calculations which are not simple. 

I think this is a great option to have, and I will be taking a play in the coming weeks.

# Misc

## Delta Lake

[I didn’t see this coming](https://delta.io/). Though I’m not particularly excited about it. Delta is great product, as far as I can tell right now the Databricks offering has not changed - they have simply open sourced it so that the whole Spark community can use it.

## Adaptive Execution & Join Hints

If you look at the slide above from the keynote you will see Spark 3.0 looks to include Adaptive Execution, something SQL Server added in 2017 - I don’t know the details but this sounds interesting as bad execution DAG’s are inevitable - and debugging is a complex task. So anything that can help developers in this space has huge potential.

Join hints are also mentioned.

The most details on this can be found in the [Jira](https://issues.apache.org/jira/browse/SPARK-9850) ticket.

## Koalas

Pandas for PySpark. Again another way to make life easier for dev’s switching to Spark from pure Python. I suspect the data science community will enjoy this the most. Personally I always found conversion from Pandas to Spark quite easy - the syntax is very similar for the bulk/key components. This just makes life even easier!

# Databricks-Connect

Sadly no mention of Databricks-Connect being released. Though the keynote was focused on Spark rather than Databricks anyway so that might be why. I’m really hopeful this will be released soon.
