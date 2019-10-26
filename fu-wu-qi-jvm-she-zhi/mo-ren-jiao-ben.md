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

**存放目录：**

> 默认存放在/export/servers目录下，文件名称固定为：debug-service-restart.sh

**使用：**

> 需要调试此服务时，保持服务器上代码与本地代码相同，执行debug-service-restart.sh脚本，会重启当前服务，并开启远程端口，本地debug当前服务即可。

debug-service-restart.sh

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

**说明**
预发布环境，需要在运维同学同意与辅助下，开启相应端口；（建议统一端口，此文service远程端口设置为8888）


