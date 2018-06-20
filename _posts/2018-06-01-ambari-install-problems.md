---
title: Ambari构建问题
categories:
 - hadoop
tags:
 - hadoop
 - ambari
---
# Ambari构建问题

## 构建基础环境 
* jdk8 http://public-repo-1.hortonworks.com/ARTIFACTS/jdk-8u77-linux-x64.tar.gz
* git `yum install git -y`
* maven 3.1及以上  http://maven.apache.org/install.html
* nodejs 最新版 https://nodejs.org/en/download/package-manager/#enterprise-linux-and-fedora
* python-devel `yum install python-devel.x86_64 -y`
* python降级 https://blog.csdn.net/gcangle/article/details/50098151
* OpenSSL 降级 http://www.mamicode.com/info-detail-2025612.html

## 常见异常
* _Too many files with unapproved license: 13948_   
   * 解决方案：编译命令加入 `-Drat.skip=true` 参数
``` 
`mvn -e -B clean install rpm:rpm -Drat.skip=true -DnewVersion=2.6.2.0.0 -DbuildNumber=631319b00937a8d04667d93714241d2a0cb17275 -DskipTests -Dpython.ver="python >= 2.6"`
```
* _ambari-admin编译失败_    
   **解决：**  个人将ambari-admin（包括子目录ambari-admin/src/main/resources/ui/admin-web）里面生成的node、node_modules、package-lock.json删除，保持干净的源码环境，再单独编译ambari-admin
   * `cd ambari-admin/src/main/resources/ui/admin-web`目录下，编辑 .bowerrc ，修改后的内容如下：
 ```
    {
        "directory": "app/bower_components",
        "allow_root": true
    } 
 ```
   在上述目录下，安装npm依赖包，全局安装gulp、bower,安装bower的依赖包,安装 gulp-webserver
  ``` 
     npm install
     npm install -g bower
     npm install -g gulp
     bower install
     npm install gulp-webserver --save-dev
  ```
   在ambari-admin目录下运行命令重新单独编译ambari-admin模块    
    `mvn install -Drat.skip=true -Preplaceurl -X`
   将 `apache-ambari-2.6.2-src/pom.xml`中的ambari-admin模块注释  
    `<!--<module>ambari-admin</module>-->`  
   转载：https://blog.csdn.net/ZhouyuanLinli/article/details/79399287

*  _ambari-metrics编译时间过长，下载超时_
   * 手动下载文件后搭建服务器提供本地下载
   * 搭建nginx/httpd服务器，将下载文件放入服务器路径中
   * 修改源文件下载路径 `apache-ambari-2.6.2-src/ambari-metrics/pom.xml`
  ``` 
    <hbase.tar>http://10.122.64.230/hbase-1.1.2.2.6.4.0-91.tar.gz</hbase.tar>
    <hbase.folder>hbase-1.1.2.2.6.4.0-91</hbase.folder>
    <hadoop.tar>http://10.122.64.230/hadoop-2.7.3.2.6.4.0-91.tar.gz</hadoop.tar>
    <hadoop.folder>hadoop-2.7.3.2.6.4.0-91</hadoop.folder>
    <hbase.winpkg.zip>https://msibuilds.blob.core.windows.net/hdp/2.x/2.2.4.2/2/hbase-0.98.4.2.2.4.2-0002-hadoop2.winpkg.zip</hbase.winpkg.zip>
    <hbase.winpkg.folder>hbase-0.98.4.2.2.4.2-0002-hadoop2</hbase.winpkg.folder>
    <hadoop.winpkg.zip>https://msibuilds.blob.core.windows.net/hdp/2.x/2.2.4.2/2/hadoop-2.6.0.2.2.4.2-0002.winpkg.zip</hadoop.winpkg.zip>
    <hadoop.winpkg.folder>hadoop-2.6.0.2.2.4.2-0002</hadoop.winpkg.folder>
    <grafana.folder>grafana-2.6.0</grafana.folder>
    <grafana.tar>http://10.122.64.230/grafana-2.6.0.linux-x64.tar.gz</grafana.tar>
    <phoenix.tar>http://10.122.64.230/phoenix-4.7.0.2.6.4.0-91.tar.gz</phoenix.tar>
	
	<!--
    <hbase.tar>https://public-repo-1.hortonworks.com/HDP/centos6/2.x/updates/2.6.4.0/tars/hbase/hbase-1.1.2.2.6.4.0-91.tar.gz</hbase.tar>
    <hbase.folder>hbase-1.1.2.2.6.4.0-91</hbase.folder>
    <hadoop.tar>https://public-repo-1.hortonworks.com/HDP/centos6/2.x/updates/2.6.4.0/tars/hadoop/hadoop-2.7.3.2.6.4.0-91.tar.gz</hadoop.tar>
    <hadoop.folder>hadoop-2.7.3.2.6.4.0-91</hadoop.folder>
    <hbase.winpkg.zip>https://msibuilds.blob.core.windows.net/hdp/2.x/2.2.4.2/2/hbase-0.98.4.2.2.4.2-0002-hadoop2.winpkg.zip</hbase.winpkg.zip>
    <hbase.winpkg.folder>hbase-0.98.4.2.2.4.2-0002-hadoop2</hbase.winpkg.folder>
    <hadoop.winpkg.zip>https://msibuilds.blob.core.windows.net/hdp/2.x/2.2.4.2/2/hadoop-2.6.0.2.2.4.2-0002.winpkg.zip</hadoop.winpkg.zip>
    <hadoop.winpkg.folder>hadoop-2.6.0.2.2.4.2-0002</hadoop.winpkg.folder>
    <grafana.folder>grafana-2.6.0</grafana.folder>
    <grafana.tar>https://grafanarel.s3.amazonaws.com/builds/grafana-2.6.0.linux-x64.tar.gz</grafana.tar>
    <phoenix.tar>https://public-repo-1.hortonworks.com/HDP/centos6/2.x/updates/2.6.4.0/tars/phoenix/phoenix-4.7.0.2.6.4.0-91.tar.gz</phoenix.tar>
    -->
```

# 官方文档
## Tools needed to build Ambari
The following tools are needed to build Ambari from source.  
Alternatively, you can easily launch a VM that is preconfigured with all the tools that you need.  See the Pre-Configured Development Environment section in the Quick [Start Guide](https://cwiki.apache.org/confluence/display/AMBARI/Quick+Start+Guide).
* xCode (if using Mac - free download from the apple store)
* JDK 8 (Ambari 2.6 and below can be compiled with JDK 7, from Ambari 2.7, it can be compiled with at least JDK 8)
* Apache Maven 3.3.9 or later  
Tip: In order to persist your changes to the JAVA_HOME environment variable and add Maven to your path, create the following files:
File: ~/.profile  
`source ~/.bashrc`    
File: ~/.bashrc  
 ```
export PATH=/usr/local/apache-maven-3.3.9/bin:$PATH
export JAVA_HOME=$(/usr/libexec/java_home)
export _JAVA_OPTIONS="-Xmx2048m -XX:MaxPermSize=512m -Djava.awt.headless=true"
 ```
 * Python 2.6
 * Python setuptools - python 2.6: Download or python 2.7: Download and run:  
 2.6:  
 `sh setuptools-0.6c11-py2.6.egg`  
 2.7  
 `sh setuptools-0.6c11-py2.7.egg`
 * rpmbuild (rpm-build package)
 * g++ (gcc-c++ package)  
 ## Running Unit Tests
 * mvn clean test
  * Run unit tests in a single module:  
 `mvn -pl ambari-server test`  
  * Run only Java tests:  
 `mvn -pl ambari-server -DskipPythonTests`  
  * Run only specific Java tests:  
 `mvn -pl ambari-server -DskipPythonTests -Dtest=AgentHostInfoTest test`  
  * Run only Python tests:  
 `mvn -pl ambari-server -DskipSurefireTests test`  
  * Run only specific Python tests:  
 `mvn -pl ambari-server -DskipSurefireTests -Dpython.test.mask=TestUtils.py test`  
  * Run only Checkstyle and RAT checks:  
` mvn -pl ambari-server -DskipTests test`  

 NOTE: Please make sure you have npm in the path before running the unit tests.
 ## Generating Findbugs Report
 * mvn clean install  
 
 This will generate xml and html report unders target/findbugs. You can also add flags to skip unit tests to generate report faster.
 ## Building Ambari
 Note: if you can an error that too many files are open while building, then run: ulimit -n 10000 (for example)    
 To build Ambari RPMs, run the following.    
 Note: Replace ${AMBARI_VERSION} with a 4-digit version you want the artifacts to be (e.g., -DnewVersion=1.6.1.1)    
 Note: If running into errors while compiling the ambari-metrics package due to missing the artifacts of jms, jmxri, jmxtools:    
 `[ERROR] Failed to execute goal on project ambari-metrics-kafka-sink: Could not resolve dependencies for project org.apache.ambari:ambari-metrics-kafka-sink:jar:2.0.0-0: The following artifacts could not be resolved: javax.jms:jms:jar:1.1, com.sun.jdmk:jmxtools:jar:1.2.1, com.sun.jmx:jmxri:jar:1.2.1: Could not transfer artifact javax.jms:jms:jar:1.1 from/to java.net (https://maven-repository.dev.java.net/nonav/repository): No connector available to access repository java.net (https://maven-repository.dev.java.net/nonav/repository) of type legacy using the available factories WagonRepositoryConnectorFactory`    
 The work around is to manually install the three missing artifacts:
 ``` 
 mvn install:install-file -Dfile=jms-1.1.pom -DgroupId=javax.jms -DartifactId=jms -Dversion=1.1 -Dpackaging=jar
 mvn install:install-file -Dfile=jmxtools-1.2.1.pom -DgroupId=com.sun.jdmk -DartifactId=jmxtools -Dversion=1.2.1 -Dpackaging=jar
 mvn install:install-file -Dfile=jmxri-1.2.1.pom -DgroupId=com.sun.jmx -DartifactId=jmxri -Dversion=1.2.1 -Dpackaging=jar
 ```
 If when compiling it seems stuck, and you've already increased Java and Maven heapsize, it could be that Ambari Views has a lot of artifacts, and the rat-check is choking up. In this case, try running
``` 
git clean -df (this will remove untracked files and directories)
mvn clean package -DskipTests -Drat.ignoreErrors=true
or  
mvn clean package -DskipTests -Drat.skip
```
### RHEL/CentOS 6:
``` 
mvn versions:set -DnewVersion=${AMBARI_VERSION}
 
#Note: The ambari-metrics project is not wired up to the main ambari project. However there is a dependency on ambari-metrics-common to build the ambari-server RPM. 
#Hence you also need to set ambari-metrics project version as well.
pushd ambari-metrics
mvn versions:set -DnewVersion=${AMBARI_VERSION}
popd
 
mvn -B clean install package rpm:rpm -DskipTests -Dpython.ver="python >= 2.6" -Preplaceurl
```
### Ubuntu 12:
``` 
mvn versions:set -DnewVersion=${AMBARI_VERSION}
 
#Note: The ambari-metrics project is not wired up to the main ambari project. However there is a dependency on ambari-metrics-common to build the ambari-server RPM. 
#Hence you also need to set ambari-metrics project version as well.
pushd ambari-metrics
mvn versions:set -DnewVersion=${AMBARI_VERSION}
popd
 
mvn -B clean install package jdeb:jdeb -DskipTests -Dpython.ver="python >= 2.6" -Preplaceurl
```
Ambari Server will create following packages
* RPM will be created under AMBARI_DIR/ambari-server/target/rpm/ambari-server/RPMS/noarch.
* DEB will be created under AMBARI_DIR/ambari-server/target/
Ambari Agent will create following packages
* RPM will be created under AMBARI_DIR/ambari-agent/target/rpm/ambari-agent/RPMS/x86_64.
* DEB will be created under AMBARI_DIR/ambari-agent/target
Optional parameters:
* -X -e: add these options for more verbose output by Maven.  Useful when debugging Maven issues.
* -DdefaultStackVersion=STACK-VERSION
* Sets the default stack and version to be used for installation (e.g., -DdefaultStackVersion=HDP-1.3.0)
* -DenableExperimental=true
* Enables experimental features to be available via Ambari Web (default is false)
* All views can be packaged in RPM by adding -Dviews parameter
  * mvn -B clean install package rpm:rpm -Dviews -DskipTests
* Specific views can be built by adding --projects parameter to the -Dviews
  * mvn -B clean install package rpm:rpm --projects ambari-web,ambari-project,ambari-views,ambari-admin,contrib/views/files,contrib/views/pig,ambari-server,ambari-agent,ambari-client,ambari-shell -Dviews -DskipTests
  
_NOTE: Run everything as root below._
### Building Ambari Metrics
If you plan on installing the Ambari Metrics service, you will also need to build the Ambari Metrics project. 
``` 
cd ambari-metrics
mvn clean package -Dbuild-rpm -DskipTests

For Ubuntu:
cd ambari-metrics
mvn clean package -Dbuild-deb -DskipTests
```
**Note:**
The metrics rpms will be found at: ambari-metrics-assembly/target/. These would be need for installing the Ambari Metrics service.
### Running the Ambari Server
First, install the Ambari Server RPM.  
**On RHEL/CentOS:**   
`yum install ambari-server/target/rpm/ambari-server/RPMS/noarch/ambari-server-*.noarch.rpm`
**On Ubuntu 12:**    
``` 
dpkg --install ambari-server/target/ambari-server-*.deb          # Will fail with missing dependencies errors
apt-get update                                                   # Update locations of dependencies
apt-get install -f                                               # Install all failed dependencies
dpkg --install ambari-server/target/ambari-server-*.deb          # Will succeed
```  
Initialize Ambari Server:  
`ambari-server setup`  
Start up Ambari Server:  
`ambari-server start`   
See Ambari Server log:  
`tail -f /var/log/ambari-server/ambari-server.log`  
To access Ambari, go to  
`http://{ambari-server-hostname}:8080`  
from your web browser and log in with username admin and password admin.
### Install and Start the Ambari Agent Manually on Each Host in the Cluster
Install the Ambari Agent RPM.    
On RHEL/CentOS:    
`yum install ambari-agent/target/rpm/ambari-agent/RPMS/x86_64/ambari-agent-*.rpm`  
Ubuntu12:    
`dpkg --install ambari-agent/target/ambari-agent-*.deb`    
Edit the location of Ambari Server in /etc/ambari-agent/conf/ambari-agent.ini by editing the hostname line.    
Start Ambari Agent:  
`ambari-agent start`  
See Ambari Agent log:  
`tail -f /var/log/ambari-agent/ambari-agent.log`  

source https://cwiki.apache.org/confluence/display/AMBARI/Ambari+Development