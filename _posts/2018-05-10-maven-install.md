---
title: Maven安装
categories:
 - linux
tags:
 - maven
---
# Installing Apache Maven
The installation of Apache Maven is a simple process of extracting the archive and adding the _bin_ folder with the `mvn` command to the _PATH_.

Detailed steps are:
* Ensure _JAVA_HOME_ environment variable is set and points to your JDK installation
* Extract distribution archive in any directory
``` 
unzip apache-maven-3.5.3-bin.zip
```  
or  
``` 
tar xzvf apache-maven-3.5.3-bin.tar.gz
```  
Alternatively use your preferred archive extraction tool.
* Add the bin directory of the created directory apache-maven-3.5.3 to the PATH environment variable
* Confirm with mvn -v in a new shell. The result should look similar to
``` 
Apache Maven 3.5.3 (138edd61fd100ec658bfa2d307c43b76940a5d7d; 2017-10-18T08:58:13+01:00)
Maven home: /opt/apache-maven-3.5.3
Java version: 1.8.0_45, vendor: Oracle Corporation
Java home: /Library/Java/JavaVirtualMachines/jdk1.8.0_45.jdk/Contents/Home/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "mac os x", version: "10.8.5", arch: "x86_64", family: "mac"
```
## Windows Tips
* Check environment variable value e.g.
``` 
echo %JAVA_HOME% 
C:\Program Files\Java\jdk1.7.0_51
```
* Adding to PATH: Add the unpacked distribution’s bin directory to your user PATH environment variable by opening up the system properties (WinKey + Pause), selecting the “Advanced” tab, and the “Environment Variables” button, then adding or selecting the PATH variable in the user variables with the value C:\Program Files\apache-maven-3.5.3\bin. The same dialog can be used to set JAVA_HOME to the location of your JDK, e.g. C:\Program Files\Java\jdk1.7.0_51
* Open a new command prompt (Winkey + R then type cmd) and run mvn -v to verify the installation.

## Unix-based Operating System (Linux, Solaris and Mac OS X) Tips
* Check environment variable value
``` 
echo $JAVA_HOME
/Library/Java/JavaVirtualMachines/jdk1.8.0_45.jdk/Contents/Home
```
* Adding to PATH
``` 
export PATH=/opt/apache-maven-3.5.3/bin:$PATH
```