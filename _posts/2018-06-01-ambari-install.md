---
title: Ambari构建安装
categories:
 - hadoop
tags:
 - hadoop
 - ambari
---

# Installation Guide for Ambari
## yum install Ambari 2.4.3

Refer [Ambari Development](2018-06-01-ambari-install-problems.md) for prerequisites and additional information on how to build Apache Ambari.

Ambari安装之部署本地库：https://yq.aliyun.com/articles/368491?spm=5176.10695662.1996646101.searchclickresult.12d592d72cdPEF

Ambari安装之安装并配置Ambari-server：https://yq.aliyun.com/articles/368496?spm=5176.10695662.1996646101.searchclickresult.12d592d72cdPEF

参考：https://yq.aliyun.com/articles/368491?spm=a2c4e.11153940.blogcont368496.26.557eabf4EGl7cT



## Build and install Ambari 2.6.2
**Step 1:** Download and build Ambari 2.6.2 source  
Go to http://www.apache.org/dyn/closer.cgi/ambari/ambari-2.6.2 and find the suggested mirror for download. The process to verify the download is described is at http://www.apache.org/dyn/closer.cgi#verify
``` 
wget http://www.apache.org/dist/ambari/ambari-2.6.2/apache-ambari-2.6.2-src.tar.gz (use the suggested mirror from above)
tar xfvz apache-ambari-2.6.2-src.tar.gz
cd apache-ambari-2.6.2-src
mvn versions:set -DnewVersion=2.6.2.0.0
 
pushd ambari-metrics
mvn versions:set -DnewVersion=2.6.2.0.0
popd
```
**Note:** If running into errors while compiling the ambari-metrics package due to missing the artifacts of jms, jmxri, jmxtools:    
`[ERROR] Failed to execute goal on project ambari-metrics-kafka-sink: Could not resolve dependencies for project org.apache.ambari:ambari-metrics-kafka-sink:jar:2.2.2-0: The following artifacts could not be resolved: javax.jms:jms:jar:1.1, com.sun.jdmk:jmxtools:jar:1.2.1, com.sun.jmx:jmxri:jar:1.2.1: Could not transfer artifact javax.jms:jms:jar:1.1 from/to java.net (https://maven-repository.dev.java.net/nonav/repository): No connector available to access repository java.net (https://maven-repository.dev.java.net/nonav/repository) of type legacy using the available factories WagonRepositoryConnectorFactory`    
The work around is to manually install the three missing artifacts:
``` 
mvn install:install-file -Dfile=jms-1.1.pom -DgroupId=javax.jms -DartifactId=jms -Dversion=1.1 -Dpackaging=jar
mvn install:install-file -Dfile=jmxtools-1.2.1.pom -DgroupId=com.sun.jdmk -DartifactId=jmxtools -Dversion=1.2.1 -Dpackaging=jar
mvn install:install-file -Dfile=jmxri-1.2.1.pom -DgroupId=com.sun.jmx -DartifactId=jmxri -Dversion=1.2.1 -Dpackaging=jar
```
The three poms are:
``` 
$ cat jms-1.1.pom
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>javax.jms</groupId>
  <artifactId>jms</artifactId>
  <version>1.1</version>
  <name>Java Message Service</name>
  <description>
    The Java Message Service (JMS) API is a messaging standard that allows application components based on the Java 2 Platform, Enterprise Edition (J2EE) to create, send, receive, and read messages. It enables distributed communication that is loosely coupled, reliable, and asynchronous.
  </description>
  <url>http://java.sun.com/products/jms</url>
  <distributionManagement>
    <downloadUrl>http://java.sun.com/products/jms/docs.html</downloadUrl>
  </distributionManagement>
```
``` 
$ cat jmxri-1.2.1.pom
<?xml version="1.0" encoding="UTF-8"?><project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.sun.jmx</groupId>
  <artifactId>jmxri</artifactId>
  <version>1.2.1</version>
  <distributionManagement>
    <status>deployed</status>
  </distributionManagement>
```
``` 
$ cat jmxtools-1.2.1.pom
<?xml version="1.0" encoding="UTF-8"?><project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.sun.jdmk</groupId>
  <artifactId>jmxtools</artifactId>
  <version>1.2.1</version>
  <distributionManagement>
    <status>deployed</status>
  </distributionManagement>
```
### RHEL (CentOS 6 or 7) & SUSE (SLES 11):
`mvn -B clean install rpm:rpm -DnewVersion=2.6.2.0.0 -DbuildNumber=631319b00937a8d04667d93714241d2a0cb17275 -DskipTests -Dpython.ver="python >= 2.6"`

### Ubuntu/Debian:
`mvn -B clean install jdeb:jdeb -DnewVersion=2.6.2.0.0 -DbuildNumber=631319b00937a8d04667d93714241d2a0cb17275 -DskipTests -Dpython.ver="python >= 2.6"`

**Note:** You need to have tools such as rpm-build tool, brunch, etc.  For details on prerequisites, please see Ambari Development. 

**Step 2: Install Ambari Server**    
Install the rpm package from ambari-server/target/rpm/ambari-server/RPMS/noarch/    
_[For CentOS 6 or 7]_
`yum install ambari-server*.rpm    #This should also pull in postgres packages as well.`    
_[For SLES 11]_  
`zypper install ambari-server*.rpm    #This should also pull in postgres packages as well.`  
_[For Ubuntu/Debian]_  
`apt-get install ./ambari-server*.deb   #This should also pull in postgres packages as well.`  

**Step 3: Setup and Start Ambari Server**    
Run the setup command to configure your Ambari Server, Database, JDK, LDAP, and other options:    
`ambari-server setup`    
Follow the on-screen instructions to proceed.    
Once set up is done, start Ambari Server:    
`ambari-server start`

**Step 4: Install and Start Ambari Agent on All Hosts**    
**Note:** This step needs to be run on all hosts that will be managed by Ambari.  
Copy the rpm package from ambari-agent/target/rpm/ambari-agent/RPMS/x86_64/ and run:  
_[For CentOS 6 or 7]_  
`yum install ambari-agent*.rpm`  
[Ubuntu/Debian]  
`apt-get install ./ambari-agent*.deb`  
Edit /etc/ambari-agent/ambari.ini  
```
...
[server]
hostname=localhost
...
```
Make sure hostname under the [server] section points to the actual Ambari Server host, rather than "localhost".  
`ambari-agent start`  
**Step 5: Deploy Cluster using Ambari Web UI**
Open up a web browser and go to http://<ambari-server-host>:8080.    
Log in with username **admin** and password **admin** and follow on-screen instructions. Secure your environment by ensuring your administrator details are changed from the default values as soon as possible.    
Under Install Options page, enter the hosts to add to the cluster.  Do not supply any SSH key, and check "Perform manual registration on hosts and do not use SSH" and hit "Next".

转载自：https://cwiki.apache.org/confluence/display/AMBARI/Installation+Guide+for+Ambari+2.6.2


