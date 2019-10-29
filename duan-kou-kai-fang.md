### 端口开放
请参见：linux端口开放指定端口的两种方法

开放端口的方法：

**方法一：命令行方式**
               1. 开放端口命令： /sbin/iptables -I INPUT -p tcp --dport 8080 -j ACCEPT
               2.保存：/etc/rc.d/init.d/iptables save
               3.重启服务：/etc/init.d/iptables restart
               4.查看端口是否开放：/sbin/iptables -L -n
    

** 方法二：直接编辑/etc/sysconfig/iptables文件**
               1.编辑/etc/sysconfig/iptables文件：vi /etc/sysconfig/iptables
                   加入内容并保存：-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 8080 -j ACCEPT
               2.重启服务：/etc/init.d/iptables restart
               3.查看端口是否开放：/sbin/iptables -L -n