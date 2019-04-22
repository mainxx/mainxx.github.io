# JAVA配置环境变量

* 系统变量→新建 JAVA_HOME 变量 。
* 变量值填写jdk的安装目录（本人是 E:\Java\jdk1.7.0)
* 系统变量→寻找 Path 变量→编辑
* 在变量值最后输入 %JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;
* （注意原来Path的变量值末尾有没有;号，如果没有，先输入；号再输入上面的代码）
* 系统变量→新建 CLASSPATH 变量
* 变量值填写   .;%JAVA_HOME%\lib;%JAVA_HOME%\lib\tools.jar（注意最前面有一点）
* java -version