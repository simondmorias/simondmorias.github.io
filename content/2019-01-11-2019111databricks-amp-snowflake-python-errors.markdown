---
author: Simon D'Morias
date: '2019-01-11 09:07:17'
draft: false
title: Databricks & Snowflake Python Errors
type: post
url: /blog/2019/1/11/databricks-amp-snowflake-python-errors
tags:
- Databricks
- Snowflake
- Python
- PySpark
---



      

      
        
          
        
        

        
          
            
          


            

![Snowflake](/images/snowflake-1024x538_color_trans.png)

            


          


        
          
        

        
      
        
      

    


  


  




I’ve recently been playing with writing data to Snowflake from Databricks. Reading and Writing is pretty simple as per the [instructions from Databricks](https://docs.databricks.com/spark/latest/data-sources/snowflake.html). 

But if you want to execute SnowSQL commands using the [snowflake-python-connector](https://docs.snowflake.net/manuals/user-guide/python-connector.html) and Python 3 you will be greeted with this error when you try to import the module (despite attaching the library without error):
    
```cffi library '_openssl' has no function, constant or global variable named 'Cryptography_HAS_SET_ECDH_AUTO'```

Your error may vary slightly - but you will be missing an OpenSSL function. This is because the version of OpenSSL installed on Databricks is too old for the Snowflake connector. 

What’s interesting is that the Databricks link at the start of the post shows reading/writing data using Scala and Python. But for executing SnowSQL it only shows Scala examples. So I suspect Databricks are aware of this issue but have decided not to fix it yet. This does suggest that there maybe some incompatibilities with doing this - please let me know if you find this breaks anything. Don’t panic if something does break - just restart your cluster and the changes outlined below will be undone.

To fix this problem firstly remove the library and restart your cluster. Then install the library using PIP:
    
```%sh sudo apt-get install python3-pip -y```

Check you don’t get any errors and then run:
    
```%sh pip3 install --upgrade snowflake-connector-python```

You should now be able to run:
    
```import snowflake.connector```

Without any errors.

You will need to do this on every cluster restart. So you should consider an [init script](https://docs.databricks.com/user-guide/clusters/init-scripts.html) for the cluster.
