### 服务布局

#### service 服务

启动：service服务启动是通过`*-impl`项目bin目录下的start.sh脚本启动（jenkins打包，automan构建）

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




