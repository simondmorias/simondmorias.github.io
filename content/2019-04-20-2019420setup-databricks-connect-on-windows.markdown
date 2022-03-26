---
author: Simon D'Morias
date: '2019-04-20 17:57:12'
draft: false
title: Setup Databricks-Connect on Windows 10
type: post
url: /blog/2019/4/20/setup-databricks-connect-on-windows
tags:
- Databricks-Connect
- Databricks
- Python
- PySpark
---

**UPDATE June 2020** - How about using a container instead? It’s much easier than installing all this stuff:  [Prebuilt container](https://datathirst.net/blog/2020/6/7/databricks-connect-in-a-container)

Having recently tried to get DBConnect working on a Windows 10 machine I’ve realised things are not as easy as you might think.

These are the steps I have found to setup a new machine and get Databricks-Connect working.

## Install Java

Download and install [Java SE Runtime Version 8](https://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html). It’s very important you only get version 8 nothing later. There was a major change in Java 9 and Spark simply doesn’t work with it (or v10 or v11). 

When you install Java change the install location to be C:\Java. By default it will try to install into Program Files - this is a problem for Spark as it does not like spaces in the path.

Once installed set JAVA_HOME using PowerShell (running as Administrator) execute:
    
```[Environment]::SetEnvironmentVariable("JAVA_HOME", "C:\Java", "Machine")```

## Install Conda

You can either download and install full blown [Anaconda ](https://www.anaconda.com/distribution/)or use [miniconda](https://docs.conda.io/en/latest/miniconda.html). 

## User Profile Path

By default Windows creates your user profile in a folder C:\Users. Depending on how your username is setup on your machine you may a space or special character in it. Such as C:\Users\Simon DMorias - this is a problem because again Spark (and Python to be fair) hate spaces in paths.

If you profile path has no space in it - then lucky you, carry on. If not follow these steps - note that this means hacking the registry - I take no responsibility for the damage you could do here.

To fix this login in as another user on the PC - the user must be an administrator. This is only a temporary user so you can create a new one if need be. Open regedit and find the key:
    
```HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList\<User SID>\ ```

Change the path so that there is no space.

Then find the folder in Windows Explorer and change the folder name. So in my example I would now have C:\Users\SimonDMorias

To prevent any existing application breaking you can create a symlink to the old folder, using command prompt (as Administrator):
    
```mklink /j "C:\Users\Simon DMorias" "C:\Users\SimonDMorias"```

Now log back in as the original user and everything should be peachy.

## Download WinUtils

This pesty bugger is part of Hadoop and required by Spark to work on Windows. Quick install, open Powershell (as an admin) and run (if you are on a corporate network with funky security you may need to download the exe manually):
    
```New-Item -Path "C:\Hadoop\Bin" -ItemType Directory -Force```
    
```Invoke-WebRequest -Uri https://github.com/steveloughran/winutils/raw/master/hadoop-2.7.1/bin/winutils.exe -OutFile "C:\Hadoop\Bin\winutils.exe"```
    
```[Environment]::SetEnvironmentVariable("HADOOP_HOME", "C:\Hadoop", "Machine")```

Read more about this [here](https://github.com/steveloughran/winutils).

## Create Virtual Environment

We are now a new Virtual Environment. I recommend creating one environment per project you are working on. This allow us to install different versions of Databricks-Connect per project and upgrade them separately.

From the Start menu find the Anaconda Prompt. When it opens it will have a default prompt of something like:
    
```(base) C:\Users\Simon```

The base part means you are not in a virtual environment, rather the base install. To create a new environment execute this:
    
```conda create --name dbconnect python=3.5```

**UPDATE**: On Databricks Runtime 6+ Python has been upgraded. You should use:
    
```conda create --name dbconnect python=3.7.3```

Where dbconnect is the name of your environment and can be what you want. Databricks currently runs Python 3.5 (Runtime 6 updates this to 3.7.3) - your Python version must match. Again this is another good reason for having an environment per project as this may change in the future.

Now activate the environment:
    
```conda activate dbconnect```

## Install Databricks-Connect

You are now good to go:
    
```pip install -U databricks-connect==5.3.*```

The 5.3 part MUST match the Databricks Runtime version of your cluster.

### UPDATE: Troubleshooting SSL Errors

Some corporate networks perform man in the middle interceptions of network traffic which causes pip to error with the message:
    
```Download error for https://files.pythonhosted.org/packages/...: [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed (_ssl.c:719)```

To work around this you can ignore SSL error for the domains using:
    
```pip install --trusted-host pypi.org --trusted-host files.pythonhosted.org -U databricks-connect==5.3.*```

Secondly on one machine I still got the error above but found that manually installing one dependancy fixed the issue:
    
```pip install -U pypandoc```

## Finally

**One last thing - we have set a few environment variables in this post. If you had your IDE open make sure you restart it before you try using it!**

To configure the environment please see the [original ](https://datathirst.net/blog/2019/3/7/databricks-connect-finally)post.
