---
author: Simon D'Morias
date: '2018-10-12 15:06:48'
draft: false
title: Executing SQL Server Stored Procedures from Databricks (PySpark)
type: post
url: /blog/2018/10/12/executing-sql-server-stored-procedures-on-databricks-pyspark
tags:
- Databricks
- SQL Server
- PySpark
---



      

      
        
          
        
        

        
          
            
          


            


             
![sql.png](/images/sql.png)

            


          


        
          
        

        
      
        
      

    


  


  




Databricks provides some nice [connectors for reading and writing data to SQL Server](https://docs.databricks.com/spark/latest/data-sources/sql-databases.html). These are generally want you need as these act in a distributed fashion and support push down predicates etc etc.

But sometimes you want to execute a stored procedure or a simple statement. I must stress this is not recommended - more on that at the end of this blog. I’m going to assume that as you made it here you _really_ want to do this.

[Pyodbc](https://github.com/mkleehammer/pyodbc) is the defacto library for talking to SQL Server from Python. If you try to add the library to your Databricks cluster you will get a very unfriendly error about missing files. Dig a little deeper you will find that pyodbc depends on something called [unixodbc-dev](https://packages.debian.org/sid/unixodbc-dev). 

So next you try to add that library to Databricks - you will be presented with an error than a suitable version for your OS cannot be found. Painful.

Long story short - after some messing around I came up with a script to solve these problems for you. You can run this in a notebook:
    
```
%sh
curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add -
curl https://packages.microsoft.com/config/ubuntu/16.04/prod.list > /etc/apt/sources.list.d/mssql-release.list 
apt-get update
ACCEPT_EULA=Y apt-get install msodbcsql17
apt-get -y install unixodbc-dev
/databricks/python/bin/pip install pyodbc
``` 

**UPDATE 10 JAN 2019: **I ran into problems on Databricks Runtime 5.0 and higher, this script solved the issue:
    
```
%sh
curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add -
curl https://packages.microsoft.com/config/ubuntu/16.04/prod.list > /etc/apt/sources.list.d/mssql-release.list 
apt-get update
ACCEPT_EULA=Y apt-get install msodbcsql17
apt-get -y install unixodbc-dev
sudo apt-get install python3-pip -y
pip3 install --upgrade pyodbc
```



In short the above downloads the [ODBC Driver for SQL Server ](https://docs.microsoft.com/en-us/sql/connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server?view=sql-server-2017)(version 17 is the latest today). Then the two libraries mentioned above.

Now for another problem. When you restart your cluster or create a new one these settings will be lost and you will need to run this again. Save yourself the trouble and put this into an [init script](https://docs.databricks.com/user-guide/clusters/init-scripts.html).  This way you wont have to repeat this pain.

So now you are setup you should be able to use pyodbc to execute any SQL Server Stored Procedure or SQL Statement. Here is a snippet code of how to use the library:
    
 
```
import pyodbc

conn = pyodbc.connect( 'DRIVER={ODBC Driver 17 for SQL Server};'
                       'SERVER=mydatabe.database.azure.net;'
                       'DATABASE=AdventureWorks;UID=jonnyFast;'
                       'PWD=MyPassword')

# Example doing a simple execute
conn.execute('INSERT INTO Bob (Bob1, Bob2) VALUES (?, ?)', ('A', 'B'))

# Example getting records back from stored procedure (could also be a SELECT statement)
cursor = conn.cursor()
execsp = "EXEC GetConfig 'Dev'"
conn.autocommit = True
cursor.execute(execsp)

# Get all records and iterate through
rc = cursor.fetchall()
for r in rc:
  print("Key: ",r[0], "Value: ", r[1])

conn.close()
```



The stored procedure I’m calling there above looks like this:
    
```
CREATE PROCEDURE GetConfig
    @Environment VARCHAR(10)
AS
SET NOCOUNT ON;

SELECT 'Option1' [Key], '1' [Value]
UNION ALL
SELECT 'Option2' [Key], '2' [Value]
```

And that’s it. Really, really simple huh. It took me a good few hours to figure this out - so I hope it helps you out.

Now you may remember that I said executing SQL statements this way is bad idea. Let me explain (hopefully you are still with me).

Firstly, this will run the SQL from the Driver node, so whilst it runs all of your worker nodes will be idle. That can be expensive - you’ve essentially left lots of machines idle and put work onto a single machine. So never do this for long running commands.

You will have noticed that getting a recordset back also uses cursors. Cursors are generally seen as bad - they hold connections, cause blocking issues on the database and all sorts of other badness. So again you want to fetch your records quickly and close the connection. Ideally pull back the records into a list (and don’t have more than a handful of records), then close the connection before you do anything with them.

I’ve seen people looking you use Spark to load large volumes of data into SQL Server, then want to execute a store procedure to transform/process the data. This is a really bad idea (for the idle workers reason above). You should do your transforms in Spark - otherwise use a different technology. If you are using ADF you could execute the procedure from there instead and let your cluster spin down. But you really want to do this work in Spark - thats what it’s good at.

Please note this was tested on a Python 3 cluster only.

Thanks for reading
