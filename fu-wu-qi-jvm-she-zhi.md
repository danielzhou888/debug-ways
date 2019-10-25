### 服务器JVM设置



### 如何配置Web服务

**tomcat启动脚本配置：**

tomcat启动脚本

> 目录：/export/servers/tomcat
>
> 名称：tomcat.sh

![](/assets/import6.png)添加配置：

```
java -server -Xms1024m -Xmx1024m -XX:MaxMetaspaceSize=256m -Djava.net.preferIPv4Stack=true -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=8888 -classpath /export/servers/ddky-user-impl-1.0.0/lib com.ddky.user.server.Startup > service.log 2>&1 &
```

脚本内容：

```
#!/bin/sh
#export JAVA_HOME=/usr/java/jdk1.7.0_51
#export PATH=$JAVA_HOME/bin:$PATH
export JAVA_OPTS="-server -Xms1024m -Xmx2048m -XX:MaxMetaspaceSize=512m -XX:MetaspaceSize=128m -Xdebug -Xrunjdwp:transport=dt_socket,suspend=n,server=y,address=8889 -Djava.awt.headless=true -Dsun.net.client.defaultConnectTimeout=60000 -Dsun.net.client.defaultReadTimeout=60000 -Djmagick.systemclassloader=no -Dnetworkaddress.cache.ttl=300 -Dsun.net.inetaddr.ttl=300"
#export JAVA_OPTS="-Djava.awt.headless=true -d64 -Djava.library.path=/usr/local/lib -server -Xms1200M -Xmx1200M -XX:PermSize=368M -XX:MaxPermSize=368M -XX:NewSize=512M -XX:MaxNewSize=512M -XX:ReservedCodeCacheSize=256m -XX:SurvivorRatio=2 -XX:+CMSParallelRemarkEnabled -XX:+CMSIncrementalMode -XX:+CMSIncrementalPacing -XX:CMSInitiatingOccupancyFraction=40 -XX:CMSIncrementalDutyCycleMin=0 -XX:CMSIncrementalDutyCycle=10 -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:ParallelGCThreads=20 -XX:MaxTenuringThreshold=18 -XX:+DisableExplicitGC -XX:HeapDumpPath=OOM.hprof -XX:+HeapDumpOnOutOfMemoryError"
#export LANG=zh_CN.GBK
#export LC_ALL=zh_CN.GBK
export CATALINA_HOME=/export/servers/tomcat/apache-tomcat-8.5.33
cd /export/servers/tomcat/
$CATALINA_HOME/bin/catalina.sh start -config "/export/servers/tomcat/server-tomcat-8.xml"
```

在automan构建下，会执行此脚本，开放远程端口；

预发布环境，需要在运维同学同意与辅助下，开启相应端口；（建议统一端口，此文web远程端口设置为8889）

