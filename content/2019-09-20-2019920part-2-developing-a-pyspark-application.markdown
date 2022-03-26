---
author: Simon D'Morias
date: '2019-09-20 18:24:49'
draft: false
title: Part 2 - Developing a PySpark Application
type: post
url: /blog/2019/9/20/part-2-developing-a-pyspark-application
tags:
- PySpark
- Azure DevOps
- Building
- Testing
---

This is the 2nd part of a [series](/blog/2019/9/20/series-developing-a-pyspark-application) of posts to show how you can develop PySpark applications for Databricks with Databricks-Connect and Azure DevOps.   All source code can be found [here](https://github.com/DataThirstLtd/Databricks-Connect-PySpark).

## Adding Tests

Now that we have a working local application we should add some tests. As with any data based project testing can be very difficult. Strictly speaking all tests should setup the data they need and be fully independent.  This is actually a good case to use normal PySpark locally rather than Databricks-Connect as that forces you to use small local files which you could add to a repo. Databricks-Connect executes on the cluster and therefore you need to store your test data somewhere accessible to the cluster, sandboxing requires extra effort. I’ll hopefully do another post on this in the future.

In this case I’m going to keep things very simple.

The first test for config management just tests a method returns true as it always should when running through Databricks-Connect.

The second test is for the amazon pipeline and contains this code:

```
    def mock_extract():
        data = [("A1","B1"), ("A2","B2")]
        return spark.createDataFrame(data, ["col1", "col2"])

    def test_transform_Amazon_newColumn(self):
        df = Test_amazon.mock_extract()
        df = amazon.transform_Amazon(df)
        assert "meta_timestamp" in df.columns
```


Ideally your testing can work like this for your pipelines. When we look at E, T & L - it’s generally the T that is complex part, E & L are quite basic operations, and whilst they deserve testing as well, I focus the bulk of effort on the T as that is where we are likely to see bugs.

In PySpark the common method of transforming data is to chain together transformation functions that accept a dataframe as an input and return the dataframe transformed. Rather than just “listing” all the transformations in a single function as this is hard to test.

So in the above example we can create a dummy dataframe with some values and pass that into our function and inspect the output.

The test simply validate that the column has been added as expected. 

The context.py file in this folder is a helper to add the pipelines folder to PATH on our system. Python does not allow navigating backwards hence this requirement.

## Executing Tests

The test can be executed from Anaconda prompt and executing "pytest” from the root folder:

















  

    
  
    



      

      
        
          
        
        

        
          
            
          


            


             
![pytest.png](/images/pytest.png)

            


          


        
          
        

        
      
        
      

    


  


  




  
You can also pass a single filename if you would like, such as:
    
```pytest .\tests\amazon_test.py```

That’s it we now have a testing method for our pipelines.

## Next

[Package in a Wheel](/blog/2019/9/20/part-3-developing-a-pyspark-application)

[Back to the Series Intro](/blog/2019/9/20/series-developing-a-pyspark-application)
