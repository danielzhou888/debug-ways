### intellij idea 热部署

[https://zhidao.baidu.com/question/2203232949939704068.html](https://zhidao.baidu.com/question/2203232949939704068.html)

对于需要依赖tomcat打包服务的Web服务，在修改代码、js、jsp文件时，如何做到热加载，无需重启项目呢？

#### 配置步骤

![](/assets/import11.png)

**配置热加载方式：**

![](/assets/import12.png)

> on ‘update‘ action：当用户主动执行更新的时候更新　　　　快捷键:Ctrl + F9
>
> on frame deactication:在编辑窗口失去焦点的时候更新

**注意：**如果你的工程中没有 Update classes and resources 这个选项，只有如下选项

![](/assets/import14.png)

在这种情况下你更新后只能更新classes文件中的变动，并不能更新静态文件中的变动。

![](/assets/import13.png)

