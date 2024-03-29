## JRebel热部署\(推荐\)

参考：[https://www.jianshu.com/p/2627e15d25a1](https://www.jianshu.com/p/2627e15d25a1)

### 介绍JRebel：

JRebel使你能即时看到代码、类和资源的变化，你可以一个个地上传而不是一次性全部部署。当程序员在开发环境中对任何一个类或者资源作出修改的时候，这个变化会直接反应在部署好的应用程序上，从而跳过了构建和部署的过程，节省大量的构建时间。

JRebel是一款Java虚拟机插件，它使得我们能在不进行重部署的情况下，即时看到代码的改变对一个应用程序带来的影响。

### 安装JRebel

安装和使用JRebel需要注意两点：激活和设置

1、在IDEA中一次点击 File-&gt;Settings-&gt;Plugins-&gt;Brows Repositories  
 2、在搜索框中输入JRebel进行搜索  
 3、找到JRebel for intellij  
 4、install  
 5、安装好之后需要restart IDEA

![](/assets/import27.png)

### 免费激活JRebel

选择License server方式

> Url：    [https://jrebel.hexianwei.com/a2db8523-3c85-4a8c-a22d-f8fe1c143dd4](https://jrebel.hexianwei.com/a2db8523-3c85-4a8c-a22d-f8fe1c143dd4)
>
> email：随便输入

![](/assets/import29.png)

### 验证

按照顺序先在本地使用`debug-jrebel`启动`order-service`, 然后启动`order-web`，查询dubbo-admin页面，当前服务是否注册成功。

![](/assets/import31.png)![](/assets/import32.png)

可以看到我要测试的这个Service服务是正常的

![](/assets/import33.png)
原先的方法getTotalCountByType2获取到的total值为100

![](/assets/import34.png)

新增方法getTotalCountByType3，返回200

然后窗口Build -&gt; ReCompiler刚才更改的几个类（如果更改类过多，可以build整个项目）

![](/assets/import37.png)

可以看到在服务未重启的情况下，新增方法，通过build \| Recomplier，可使修改近乎实时生效。

![](/assets/import38.png)

验证 ---&gt; 成功

