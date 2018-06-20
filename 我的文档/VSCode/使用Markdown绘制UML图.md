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