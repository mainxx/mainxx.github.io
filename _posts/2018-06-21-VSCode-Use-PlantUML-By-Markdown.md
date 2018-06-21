---
layout:     post
title:      "在VSCode使用Markdown绘制UML图"
subtitle:   "在VSCode使用PlantUML"
date:       2018-06-21
author:     "Joven"
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
    - VSCode
    - Markdown
    - UML
    - PlantUML
---

# 在VSCode使用Markdown绘制UML图

## 需要插件

* Markdown All in One
* Markdown Preview Enhanced
* PlantUML
* markdownlint
1. Markdown All in One,VSCode中支持Markdown(键盘快捷键、目录、自动预览等)
2. Markdown Preview Enhanced可以对Markdown做增强预览,比如支持各种绘图等
3. PlantUML,一款很强大的，并且可以绘制各种图形的脚本语言。需要安装java
4. markdownlint是让VSCode对Markdown文档进行标记，检查。他可以提示你写的markdown是否标准

## 需要安装工具

* VSCode
* graphviz
* java
1. Visual Studio Code (简称 VS Code / VSC) 是一款免费开源的现代化轻量级代码编辑器，支持几乎所有主流的开发语言的语法高亮、智能代码补全、自定义热键、括号匹配、代码片段、代码对比 Diff、GIT 等特性，支持插件扩展，并针对网页开发和云端应用开发做了优化。软件跨平台支持 Win、Mac 以及 Linux。<https://code.visualstudio.com/>
2. Graphviz是开源图形可视化软件。图形可视化是将结构信息表示为抽象图形和网络的图表的一种方式。它在网络，生物信息学，软件工程，数据库和网页设计，机器学习以及其他技术领域的可视化界面中有重要的应用。 <http://graphviz.org/>
3. Java 运行时环境:<http://java.org>

## 使用PlantUML插件预览

1. 在<http://plantuml.com/>下载**plantuml.jar**
2. 在path配置plantuml.jar环境变量
3. 在VSCode使用快捷键Alt+D预览PlantUML

## 命令

您**必须**设置以下环境变量才能使扩展工作：[**原文链接**](https://marketplace.visualstudio.com/items?itemName=okazuki.okazukiplantuml)

---

* JAVA_HOME：Java SDK安装目录（必须有一个bin子目录）
* Windows示例： C:\Program Files\Java\jdk1.8.0_101)
* macOS例子： /Library/Java/JavaVirtualMachines/jdk1.8.0_101.jdk/Contents/Home
* PLANTUML_JAR：plantuml.jar文件的路径
* Windows示例： C:\Users\UserName\bin\plantuml\plantuml.jar
* macOS例子： /usr/local/Cellar/plantuml/8048/libexec/plantuml.jar

如果你想使用PlantUML的功能，需要GraphViz的，你需要设置GRAPHVIZ_DOT环境变量，解释[在这里](http://plantuml.com/graphviz-dot)：

* GRAPHVIZ_DOT：dot可执行二进制文件的路径
* Windows示例： C:\Program Files\Graphviz\bin\dot.exe
* macOS例子： /usr/local/Cellar/graphviz/2.38.0_1/bin/dot

设置这些环境变量后，您需要重新启动VSCode才能使扩展工作。

**注意**：配置完成之后，大家最好在cmd命令测试以下是否可以访问

```bash
$> plantuml.jar
$> dot.exe
```

```markdown
### 在Markdown使用PlantUML

 @```plantuml  //代码块标记为plantuml便于书写
    @startuml
    interface  ICollectBeganManagerFactory<T>{
    ..创建一个ICollectBegan..
    IDisposableDependencyObjectWrapper<ICollectBegan> Create(CollectTypes collectClassify);
    }
    note right:采集开始管理工厂接口
    interface IDisposableDependencyObjectWrapper{
        Object Object{get;}
    }
    note left of IDisposableDependencyObjectWrapper:ABP内置接口：对象包装器
    IDisposableDependencyObjectWrapper <.. ICollectBeganManagerFactory : 依赖
    class CollectBeganManagerFactory<T>
    ICollectBeganManagerFactory <|.. CollectBeganManagerFactory : 实现
    @enduml
 @``` 

### 其它Marckdown

* test1
* test2
1. test1描述
2. test2描述
```

> **PlantUML官网**:<http://plantuml.com/>
>  
> **PlantUML之UML文档**:<http://plantuml.com/class-diagram>

---

> **在MPE Preview预览**

---

### 在Markdown使用PlantUML

![MPE Preview预览](/img/in-post/2018-06-21-VSCode-Use-PlantUML-By-Markdown.png)

### 其它Marckdown

* test1
* test2
1. test1描述
2. test2描述
