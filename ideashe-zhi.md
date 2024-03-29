### intellij idea设置

#### Java远程调试

> -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt\_socket,server=y,address=5005,suspend=n

##### 参数解释

> -XDebug : 启用调试。  
> -Xnoagent : 禁用默认sun.tools.debug调试器。  
> -Djava.compiler=NONE : 禁止JIT编译器的加载。  
> -Xrunjdwp : 加载JDWP的JPDA参考执行实例。  
> transport : 用于在调试程序和VM使用的进程之间通讯。  
> dt\_socket : 套接字传输。  
> dt\_shmem : 共享内存传输，仅限于 Windows。  
> server=y/n : VM是否需要作为调试服务器执行。  
> address=5005 : 调试服务器的端口号，客户端用来连接服务器的端口号。  
> suspend=y/n : 是否在调试客户端建立连接之后启动 VM 。

##### 几个例子

> -Xrunjdwp:transport=dt\_socket,server=y,address=8000
> 在8000端口监听Socket连接，挂起VM并且不加载运行主函数直到调试请求到达
>
> -Xrunjdwp:transport=dt\_shmem,server=y,suspend=n
> 选择一个可用的共享内存（因为没有指address）并监听该内存连接，同时加载运行主函数
>
> -Xrunjdwp:transport=dt\_socket,address=myhost:8000
> 连接到myhost:8000提供的调试服务（server=n，以调试客户端存在），挂起VM并且不加载运行主函数
>
> -Xrunjdwp:transport=dt\_shmem,address=mysharedmemory 通过共享内存的方式连接到调试服务，挂起VM并且不加载运行主函数
>
> -Xrunjdwp:transport=dt\_socket,server=y,address=8000,onthrow=java.io.IOException,launch=/usr/local/bin/debugstub
> 等待java.io.IOException被抛出，然后挂起VM并监听8000端口连接，在接到调试请求后以命令/usr/local/bin/debugstub dt\_socket myhost:8000执行
>
> -Xrunjdwp:transport=dt\_shmem,server=y,onuncaught=y,launch=d:\bin\debugstub.exe
> 等待一个RuntimeException被抛出，然后挂起VM并监听一个可用的共享内存，在接到调试请求后以命令d:\bin\debugstub.exe dt\_shmem执行,是可用的共享内存

##### 如何用Intellij-IDEA进行java项目的远程调试

**步骤如下：**

1. 配置应用进入debug模式，在启动项目时加入虚拟机参数，或者配置"\_JAVA\_OPTIONS"  
   “**-Xdebug -Xrunjdwp:transport=dt\_socket,server=y,suspend=n,address=\*:5005**”：![](/assets/import1.png)

2. 注意address的写法，自从Java9.0以来，JDWP默认只支持到本地，即如果你写成address=5005，那么只能在本地进行调试，并不能连接到远程\([http://www.oracle.com/technetwork/java/javase/9-notes-3745703.html\#JDK-8041435\)](http://www.oracle.com/technetwork/java/javase/9-notes-3745703.html#JDK-8041435%29。)，（Java8不用考虑）

3. 若要远程进行调试则应该在address这一参数之前增加\*::

   > -Xdebug -Xrunjdwp:transport=dt\_socket,server=y,suspend=n,address=\*:5005

4. Run -&gt; Edit Configurations...进入添加启动项页面：

5. 点击 "+" ，选择"Remote"添加，自定义一个名字（比如我命名为"remote"）：

   > 配置好你的 HostIp，以及开启的调试端口，点击 "OK" 保存：
   >
   > ![](/assets/import2.png)

6. 点击 "debug" 图标，启动调试：

7. 给项目打上断点，就能像在本地一样调试远程项目了：![](/assets/import3.png)

##### 如何设置只挂在vm一次，不影响服务正常功能

1. 右击断点处 ----&gt; 勾选Thread   ----&gt; 点击More

![](/assets/import.png)

2. 勾选Remove once hit ----&gt; 点击Done

这样就仅对断点命中一次，不会将整个VM挂起（影响服务正常功能），如果自己并未触发断点，而由别人无意触发，此时需要本地释放断点，自己重新触发（避免影响他人正常使用）。

![](/assets/import4.png)


