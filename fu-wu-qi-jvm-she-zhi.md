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

#### 一、服务通过默认脚本

##### 优缺点

> 优点：  
> 1. 方便，快速，配置简单；  
> 2. 无需借助外部操作，构建打包，重启服务时，即可开放远程端口
>
> 缺点：  
> 1. 可能被误提交、合并到trunk上，然后线上重启时会开启配置的远程端口  
> 2. 存在端口占用问题

##### 实现

在原脚本基础上，添加如下配置：

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

#### 二、外部脚本

避免误提交、合并到trunk操作，通过外部脚本，重启服务，实现间接开启远程端口的目的；

**优缺点：**

> 优点：
>
> 1. 按需开放远程端口，不会潜在对线上造成不必要的影响与损耗
>
> 缺点：
> 1. 操作繁琐，需要远程调试的服务都需要配置一套外部脚本；
> 2. 重复工作，通过automan重启后，远程端口被关闭，需要手动执行外部脚本，重启服务，开放端口调试。

**外部脚本：**

```

```



