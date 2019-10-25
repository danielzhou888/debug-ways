### intellij idea 热部署

参考：[https://zhidao.baidu.com/question/2203232949939704068.html](https://zhidao.baidu.com/question/2203232949939704068.html)

对于需要依赖tomcat打包服务的Web服务，在修改.java、js、jsp文件时，如何做到热加载，无需重启项目呢？

#### 配置步骤

![](/assets/import11.png)

**配置热部署方式：**

![](/assets/import21.png)

> on ‘update‘ action：当用户主动执行更新的时候更新　　　　快捷键:Ctrl + F9
>
> on frame deactication:在编辑窗口失去焦点的时候更新

**注意：**如果你的工程中没有 Update classes and resources 这个选项，只有如下选项

![](/assets/import14.png)

在这种情况下你更新后只能更新classes文件中的变动，并不能更新静态文件中的变动。

出现这种选项情况的原因是因为在Deployment的选项中使用的是先将工程打成war包然后再去运行的。

修改如下：

先remove掉原先设置的war包

![](/assets/import17.png)

点击添加Artifact --&gt;  添加对应的war exploded

这里需要说明下：

> \_\_\_:war exploded  
> 　　展开部署\(相当于将资源文件进行展开后进行部署\)  
> \_\_\_:war  
>         发布模式,这是先打成war包,再部署

![](/assets/import18.png)![](/assets/import19.png)

![](/assets/import23.png)

试验：---&gt; 成功

