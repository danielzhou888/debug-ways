### 服务器JVM设置

### 服务布局

#### service 服务

启动：service服务启动是通过`*-impl`项目bin目录下的start.sh脚本启动

原脚本：

    #!/bin/bash

    source /etc/profile
    cd `dirname $0`
    cd ..

    DEPLOY_DIR=`pwd`
    #修改这里
    APP_MAINCLASS="com.ddky.user.server.Startup"
    #修改这里
    appname="user"


    psid=0

    checkpid() {  
        javaps=`$JAVA_HOME/bin/jps -l | grep $APP_MAINCLASS`  
        if [ -n "$javaps" ]; then  
            psid=`echo $javaps | awk '{print $1}'`  
        else  
            psid=0  
        fi  
    }


    LIB_DIR=$DEPLOY_DIR/lib

    if [ -d "/tmp/project" ]
    then
        echo "/tmp/project is exist"
    else
        mkdir -p /tmp/project
    fi
    PID_FILE="/tmp/project/$appname-pid.log"



    if [ -f "$DEPLOY_DIR/bin/stop.sh" ]
    then
       PID_NUM=`cat $PID_FILE`
       sh "$DEPLOY_DIR/bin/stop.sh"
       while true
       do
        lsof -p $PID_NUM
        if [ $? -ne 0 ]
        then
            break
        else
            sh "$DEPLOY_DIR/bin/stop.sh"
            echo -e "\033[31m stop faild \033[0m"
            sleep 1
        fi
       done
    fi


    LIB_DIR=$DEPLOY_DIR/lib

    LIB_JARS=`ls $LIB_DIR|grep .jar|awk '{print "'$LIB_DIR'/"$0}'|tr "\n" ":"`

    checkpid
    if [ $psid -ne 0 ]; then  
            echo "================================"  
            echo "warn: $APP_MAINCLASS already started! (pid=$psid)"  
            echo "================================"  
    else
        echo -n "Starting $APP_MAINCLASS ..." 
        #启动
        JAVA_OPTS="-server -Xms1024m -Xmx1024m  -XX:MaxMetaspaceSize=256m -Djava.net.preferIPv4Stack=true
        java $JAVA_OPTS -classpath $LIB_JARS $APP_MAINCLASS >  service.log 2>&1 &

    sleep 10
        #查看启动状态
        checkpid
        if [ $psid -ne 0 ]; then  
            echo -e "\033[32m (pid=$psid) [OK] \033[0m"  

            PID=$!
            echo $PID > $PID_FILE
        else  
            echo -e  "\033[31m [Failed] \033[0m"  
        fi
    fi

#### web服务

启动：web服务是通过打war包到tomcat，再通过启动tomcat对外提供服务

### 如何配置Service服务

#### 一、服务通过默认启动脚本（不推荐）

##### 优缺点

> 优点：  
> 1. 方便，快速，配置简单；  
> 2. 无需借助外部操作，构建打包，重启服务时，即可开放远程端口
>
> 缺点：  
> 1. 可能被误提交、合并到trunk上，然后线上重启时会开启配置的远程端口  
> 2. 存在端口占用问题

##### 脚本位置

> 在各工程下的bin包下有三个脚本：

##### ![](/assets/import9.png)

##### 实现

在原启动脚本基础上，添加如下配置：

```
-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=8888"
```

**修改后：**

    #!/bin/bash

    source /etc/profile
    cd `dirname $0`
    cd ..

    DEPLOY_DIR=`pwd`
    #修改这里
    APP_MAINCLASS="com.ddky.user.server.Startup"
    #修改这里
    appname="user"


    psid=0

    checkpid() {  
        javaps=`$JAVA_HOME/bin/jps -l | grep $APP_MAINCLASS`  
        if [ -n "$javaps" ]; then  
            psid=`echo $javaps | awk '{print $1}'`  
        else  
            psid=0  
        fi  
    }


    LIB_DIR=$DEPLOY_DIR/lib

    if [ -d "/tmp/project" ]
    then
        echo "/tmp/project is exist"
    else
        mkdir -p /tmp/project
    fi
    PID_FILE="/tmp/project/$appname-pid.log"



    if [ -f "$DEPLOY_DIR/bin/stop.sh" ]
    then
       PID_NUM=`cat $PID_FILE`
       sh "$DEPLOY_DIR/bin/stop.sh"
       while true
       do
        lsof -p $PID_NUM
        if [ $? -ne 0 ]
        then
            break
        else
            sh "$DEPLOY_DIR/bin/stop.sh"
            echo -e "\033[31m stop faild \033[0m"
            sleep 1
        fi
       done
    fi


    LIB_DIR=$DEPLOY_DIR/lib

    LIB_JARS=`ls $LIB_DIR|grep .jar|awk '{print "'$LIB_DIR'/"$0}'|tr "\n" ":"`

    checkpid
    if [ $psid -ne 0 ]; then  
            echo "================================"  
            echo "warn: $APP_MAINCLASS already started! (pid=$psid)"  
            echo "================================"  
    else
        echo -n "Starting $APP_MAINCLASS ..." 
        #启动
        JAVA_OPTS="-server -Xms1024m -Xmx1024m  -XX:MaxMetaspaceSize=256m -Djava.net.preferIPv4Stack=true -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=8888"
        java $JAVA_OPTS -classpath $LIB_JARS $APP_MAINCLASS >  service.log 2>&1 &

    sleep 10
        #查看启动状态
        checkpid
        if [ $psid -ne 0 ]; then  
            echo -e "\033[32m (pid=$psid) [OK] \033[0m"  

            PID=$!
            echo $PID > $PID_FILE
        else  
            echo -e  "\033[31m [Failed] \033[0m"  
        fi
    fi

提交至开发分支与trunk分支，服务启动时候会开启此远程端口；

测试环境未开启防火墙，此远程端口可被外部直接访问；

预发布环境，需要在运维同学同意与辅助下，开启相应端口；（建议统一端口，此文service远程端口设置为8888）

#### 二、外部脚本（推荐）

避免误提交、合并到trunk操作，通过外部脚本，重启服务，实现间接开启远程端口的目的；

**优缺点：**

> 优点：
>
> 1.按需开放远程端口，不会潜在对线上造成不必要的影响与损耗  
> 缺点：  
> 1. 操作繁琐，需要远程调试的服务都需要配置一套外部脚本；  
> 2. 重复工作，通过automan重启后，远程端口被关闭，需要手动执行外部脚本，重启服务，开放端口调试。

**外部脚本：**

位置：固定在/export/servers目录下

debug-start.sh

    #!/bin/bash

    source /etc/profile
    cd /export/servers/ddky-user-impl-1.0.0

    DEPLOY_DIR=`pwd`
    #修改这里
    APP_MAINCLASS="com.ddky.user.server.Startup"
    #修改这里
    appname="user"


    psid=0

    checkpid() {  
        javaps=`$JAVA_HOME/bin/jps -l | grep $APP_MAINCLASS`  
        if [ -n "$javaps" ]; then  
            psid=`echo $javaps | awk '{print $1}'`  
        else  
            psid=0  
        fi  
    }


    LIB_DIR=$DEPLOY_DIR/lib

    if [ -d "/tmp/project" ]
    then
        echo "/tmp/project is exist"
    else
        mkdir -p /tmp/project
    fi
    PID_FILE="/tmp/project/$appname-pid.log"



    if [ -f "$DEPLOY_DIR/bin/stop.sh" ]
    then
       PID_NUM=`cat $PID_FILE`
       sh "$DEPLOY_DIR/bin/stop.sh"
       while true
       do
        lsof -p $PID_NUM
        if [ $? -ne 0 ]
        then
            break
        else
            sh "$DEPLOY_DIR/bin/stop.sh"
            echo -e "\033[31m stop faild \033[0m"
            sleep 1
        fi
       done
    fi


    LIB_DIR=$DEPLOY_DIR/lib

    LIB_JARS=`ls $LIB_DIR|grep .jar|awk '{print "'$LIB_DIR'/"$0}'|tr "\n" ":"`

    checkpid
    if [ $psid -ne 0 ]; then  
            echo "================================"  
            echo "warn: $APP_MAINCLASS already started! (pid=$psid)"  
            echo "================================"  
    else
        echo -n "Starting $APP_MAINCLASS ..." 
        #启动
        JAVA_OPTS="-server -Xms1024m -Xmx1024m  -XX:MaxMetaspaceSize=256m -Djava.net.preferIPv4Stack=true -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=8888"
        java $JAVA_OPTS -classpath $LIB_JARS $APP_MAINCLASS >  service.log 2>&1 &
            echo pwd
            echo "java $JAVA_OPTS -classpath $LIB_JARS $APP_MAINCLASS >  service.log 2>&1 &"

    sleep 10
        #查看启动状态
        checkpid
        if [ $psid -ne 0 ]; then  
            echo -e "\033[32m (pid=$psid) [OK] \033[0m"  

            PID=$!
            echo $PID > $PID_FILE
        else  
            echo -e  "\033[31m [Failed] \033[0m"  
        fi
    fi

**存放目录：**

> 默认存放在/export/servers目录下，文件名称固定为：debug-service-restart.sh

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

