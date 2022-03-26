---
author: Simon D'Morias
date: '2019-03-07 20:43:52'
draft: false
title: Databricks-Connect Limitations
type: post
url: /blog/2019/3/7/databricks-connect-limitations
tags:
- Databricks
- Databricks-Connect
---

[Databricks-Connect](https://pypi.org/project/databricks-connect/) is the feature I’ve been [waiting for](/blog/2019/3/7/databricks-connect-finally). It is a complete game changer for developing data pipelines - previously you could develop locally using Spark but that meant you couldn’t get all the nice Databricks runtime features - like Delta, DBUtils etc.

Databricks-Connect allows teams to start developing in a more Enterprise fashion than Notebooks allow. I’m a big fan.

There are however a few limitations of what you can do - I’m sure more features will be added in the future so this list will probably change. But to save you pulling your hair out these are the things I cannot get to work natively:

## Azure Key Vault Secret Scope 

Because AAD authentication is required this should be no surprise. These are in preview anyway.

## DBUtils

Currently fs and secrets work (locally). Widgets (!!!), libraries etc do not work. This shouldn’t be a major issue. If you execute on Databricks using the Python Task dbutils will fail with the error:

ImportError: No module named 'pyspark.dbutils'

This is a known issue which should hopefully be fixed soon. 

### Workaround

When running on the cluster do not import dbutils - it is already available globally. So you need a check to work out if you are running locally or not. 
    
```
setting = spark.conf.get("spark.master")
if "local" in setting:
    from pyspark.dbutils import DBUtils
    dbutils = DBUtils(spark.sparkContext)
else:
    print("Do nothing - dbutils should be available already")
```
## Azure Data Lake Gen 2

I cannot get this to work using AAD authentication (no real surprise there). You will see errors along the lines of:

<blockquote>An error occurred while calling o23.parquet. : java.lang.RuntimeException: java.lang.ClassNotFoundException: Class shaded.databricks.v20180920_b33d810.org.apache.hadoop.fs.azurebfs.SecureAzureBlobFileSystem not found   at org.apache.hadoop.conf.Configuration.getClass(Configuration.java:2195)       
> 
> </blockquote>

### Workaround

I have found Gen 2 Data Lake works if you mount the storage account from a notebook first, and then access it from Databricks-Connect using the mount path. To mount it:
    
 ```
configs = {"fs.azure.account.auth.type": "OAuth",
       "fs.azure.account.oauth.provider.type": "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider",
       "fs.azure.account.oauth2.client.id": [ServicePrincipalID],
       "fs.azure.account.oauth2.client.secret": [ServicePrincipalKey],
       "fs.azure.account.oauth2.client.endpoint": "https://login.microsoftonline.com/[TenantID]/oauth2/token",
       "fs.azure.createRemoteFileSystemDuringInitialization": "true"}

dbutils.fs.mount(
source = "abfss://[FileSystem]@[StorageAccountName].dfs.core.windows.net/",
mount_point = "/mnt/lakegen2",
extra_configs = configs)
```

**Note**: I used Databricks Runtime 5.3 (BETA) and a beta version of Databricks-Connect  (pip install databricks-connect==[5.3.0b1](https://pypi.org/project/databricks-connect/5.3.0b1/)) to get this working.

**Also note**: The fs.azure.createRemoteFileSystemDuringInitialization part of the config is very important.

You should now be able to see the storage account from Databricks-Connect.

## Structured Streaming

I guess this should also be expected - it would require a permanent session.

## Hive Metastore

I’m seeing issues when you try to create a table in any database other than the default. It seems to look at your local filesystem in error. Parquet returns this error:

Can not create the managed table('`{table_name}`'). The associated location ({path}) already exists.

Delta returns:

/_delta_log/00000000000000000000.json does not exist It is possible the underlying files have been updated. You can explicitly invalidate the cache in Spark by running 'REFRESH TABLE tableName' command in SQL or by recreating the Dataset/DataFrame involved

## Accumulators

These always return 0 when running locally - but work when deployed to the server.

### Workaround

Resolved in 5.3

I will update this list over time.
