---
layout:     post
title:      "[转]解决iis中LocalDB无法连接的问题"
subtitle:   "[转]解决iis中LocalDB无法连接的问题"
date:       2018-06-19
permalink: /IISConnectToLocalDB/
author:     "Joven"
header-img: "img/post-bg-universe.jpg"
catalog: true
tags:
    - IIS
    - LocalDB
---
# 解决iis中LocalDB无法连接的问题

## 解决方案：

1、在命令行中启用共享LocalDB连接（需要管理员权限）所共享的连接写全称，可用sqllocaldb i 命令查询：

sqllocaldb share ProjectsV13 IIS_DB

2、使用Microsoft SQL Server Management Studio连接LocalDB，此处可以使用sqllocaldb i 命令查询到，如无法连接可以重启机器再试：

服务器名称：(localdb)\.\IIS_DB

身份验证：Windows身份验证

3、新建查询，为IIS应用程序池添加登录和数据库权限，经测试在iis10下没有执行这一步同样可以访问：

create login [IIS APPPOOL\ASP.NET v4.0] from windows;
exec sp_addsrvrolemember N'IIS APPPOOL\ASP.NET v4.0', sysadmin
（注：具体应用程序池请根据需要修改）

4、修改连接字符串的data source属性：

原：data source=(LocalDB)\ProjectsV13

改：data source=(LocalDB)\.\IIS_DB;

```text
作者：slg1210
来源：CSDN
原文：https://blog.csdn.net/slg1210/article/details/82797798
参考: https://www.cnblogs.com/VAllen/articles/iis-not-run-localdb.html
```