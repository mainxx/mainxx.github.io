---
layout:     post
title:      "(翻译)ASP.NET Zero入门指南-ASP.NET Core & Angular"
subtitle:   "这个文档的目的是在几分钟之内创建一个基于ASP.NET Core & Angular项目"
date:       2018-05-13
updatedate: 2018-06-23
author:     "Joven"
header-img: "img/post-bg-aspnet-zero.jpg"
catalog: true
tags:
    - ABP
    - ASP.NET Zero
    - ASP.NET Core & Angular
---

# ABP入门指南-ASP.NET Core & Angular

点击这里[穿越到原文]

[穿越到原文]: <https://aspnetzero.com/Documents/Getting-Started-Angula> "Getting-Started-Angula"

> (原文)This document is aimed to create and run an ASP.NET Zero based project in just 10 minutes. It's assumed that you already purchased and created your ASP.NET Zero account.
>  
> 本文档的目的是创建并运行ASP。在10分钟内完成基于零的项目。假设您已经购买并创建了您的ASP.NET Zero账户.

## 登录 LOGIN

Login to this web site with your user name and password. Then you will see Download link on the main menu.

使用您的用户名和密码登录到这个web站点。然后你会在主菜单上看到下载链接。

## 创建一个项目 CREATE A PROJECT

Go to the download page. You will see a form as shown below:

转到下载页面。您将看到如下所示的表单：

![download-angular](https://raw.githubusercontent.com/aspnetzero/documents/master/doc/images/download-angular-3.png)

Select **ASP.NET Core & Angular** as Project Type and fill other required fields. Your project will be ready in one minute. When you open the downloaded zip file, you will see two folders:

选择**ASP.NET Core & Angular**作为项目类型，并填充其他必需的字段。你的项目将在一分钟内完成。当您打开下载的zip文件时，您将看到两个文件夹：

![angular-solution-folders](https://raw.githubusercontent.com/aspnetzero/documents/master/doc/images/angular-solution-folders.png)

* **angular** folder contains the Angular application which is configured to work with angular-cli.
* **aspnet-core** folder contains the server side ASP.NET Core solution and configured to work with Visual Studio.
* **angular**文件夹包含有[Angular的应用程序]，它被配置为使用[angular-cli].
* **aspnet-core**文件夹包含[服务器]端ASP。NET核心解决方案，并配置为与[Visual Studio]一起工作.

[Angular的应用程序]:<https://aspnetzero.com/Documents/Development-Guide-Angular>
[服务器]:<https://aspnetzero.com/Documents/Development-Guide-Core>
[angular-cli]:<https://cli.angular.io/>
[Visual Studio]:<https://www.visualstudio.com/vs/community/>

### 合并客户端和服务器解决方案 MERGING CLIENT AND SERVER SOLUTIONS

Client and Server solutions are designed to work separately by default but if you want to work on a single Visual Studio solution, you can select "One Solution" checkbox while downloading your project.

客户端和服务器解决方案被设计成在默认情况下单独工作，但是如果您想要在一个Visual Studio解决方案中工作，您可以在下载项目时选择“一个解决方案”复选框。

## ASP.NET CORE APPLICATION

When you open Server side solution ***.Web.sln** in **Visual Studio 2017+**, you will see the solution structure as like below:

当你打开服务器端解决方案时***.Web.sln**。在Visual Studio 2017+中，您将看到如下的解决方案结构：

If you want to work on only Xamarin project, open ***.Mobile.sln** solution. If you want to work on both Xamarin and Web projects, open ***.All.sln** solution.

如果你想在Xamarin项目上工作,那就打开 **\*.Mobile.sln** 解决方案。如果你想在Xamarin和Web项目上工作，请打开 ***.All.sln** 解决方案。

![aspnet-core-host-solution](https://raw.githubusercontent.com/aspnetzero/documents/master/doc/images/aspnet-core-host-solution-4.png)

Right click the .Web.Host project and select "Set as StartUp project": Then build the solution. It may take a longer time during the first build since all nuget packages will be restored.

对右键单击.Web.Host项目并选择“设置为启动项目”：然后构建解决方案。在第一次构建过程中，可能需要更长的时间，因为所有nuget包都将被恢复。

### 数据库连接 DATABASE CONNECTION

Open **appsettings.json** in **.Web.Host** project and change the Default connection string if you want:

在 ***.Web.Host** 项目中打开 **appsettings.json** 并更改连接字符串：

```json
"ConnectionStrings": {
    "Default": "Server=localhost; Database=PhoneBookDemoDb; Trusted_Connection=True;"
}
```

### 数据库迁移 DATABASE MIGRATIONS

We have two options to create and migrate database to the latest version.

我们有两个选项来创建和迁移数据库到最新版本。

#### ASP.NET ZERO MIGRATOR APPLICATION

ASP.NET Zero solution includes a .Migrator (like Acme.PhoneBookDemo.Migrator) project in the solution. You can run this tool for database migrations on development and production (see development guide for more information).

ASP。NET Zero解决方案包括一个 **.Migrator**（如Acme.PhoneBookDemo.Migrator）的解决方案。您可以在开发和生产上运行这个工具来进行数据库迁移（参见[开发指南]了解更多信息）

[开发指南]:<https://aspnetzero.com/Documents/Development-Guide-Angular>

#### EF迁移命令 ENTITY FRAMEWORK MIGRATION COMMAND

You can also use Entity Framework's built-in tools for migrations. Open Package Manager Console in Visual Studio, set **.**EntityFrameworkCore as the Default Project and run the Update-Database command as shown below:Â

您还可以使用Entity Framework的内置工具进行迁移。打开包管理器控制台，设置EntityFrameworkCore作为默认项目，并运行Update-Database命令如下所示：

![update-database-ef-core](https://raw.githubusercontent.com/aspnetzero/documents/master/doc/images/update-database-ef-core.png)

This command will create your database and fill initial data. You can open SQL Server Management Studio to check if database is created:

这个命令将创建您的数据库并填充初始数据。您可以打开SQL Server Management Studio检查数据库是否创建：

![created-database-tables-4](https://raw.githubusercontent.com/aspnetzero/documents/master/doc/images/created-database-tables-4.png)

You can use EF console commands for development and Migrator.exe for production. But notice that; Migrator.exe supports running migrations in multiple databases at once, which can be useful in development/production for multi tenant applications.

您可以使用EF控制台命令进行开发，以及用于生产的 Migrator.exe 。但请注意 Migrator.exe 支持同时在多个数据库中运行迁移，这对于多租户应用程序的开发/生产是非常有用的。

### 多租户 MULTI-TENANCY

ASP.NET Zero supports multi-tenant and single-tenant applications. Multi-tenancy is **enabled by default**. If you don't have an idea about Multi-Tenancy, you can read <https://en.wikipedia.org/wiki/Multitenancy>. If you don't want to create a multi-tenant application, you can **disable** it by setting **AbpZeroTemplateConsts.MultiTenancyEnabled** to false in the .Core.Shared project.

ASP.NET Zero 支持多承租者和单租户应用程序。在默认情况下启用多租户。如果您对多租户没有概念，您可以阅读<https://en.wikipedia.org/wiki/Multitenancy>。如果您不想创建多租户应用程序，如果您不想创建一个多租户应用程序，您可以通过在 ***.Core.Shared** 项目中设置 **AbpZeroTemplateConsts.MultiTenancyEnabled** 为false来禁用它。

### RUN API HOST

Once you've done the configuration, you can run the application. Server side application only contains APIs. So, default page is a swagger UI which can be used to investigate the API:

一旦您完成了配置，您就可以运行应用程序了。服务器端应用程序只包含api。因此，默认页面是一个可以用来调试API的swagger界面：

![swagger-ui](https://raw.githubusercontent.com/aspnetzero/documents/master/doc/images/swagger-ui-ng2-1.png)

## ANGULAR APPLICATION

### PREREQUIREMENTS

Angular application needs the following tools to be installed:

ANGULAR应用程序需要安装以下工具：

* [nodejs] 6.9+ with npm 3.10+
* Typescript 2.0+
* [yarn]

[nodejs]:<https://nodejs.org/en/download/>
[yarn]:<https://yarnpkg.com/>

### 还原Angular包 RESTORE PACKAGES

Navigate to the Angular folder, open a command line and run the following command to restore packages:

导航到Angular文件夹，打开一个命令行并运行下面的命令来恢复包裹：

```bash
yarn
```

We suggest to use [yarn] because npm has some problems. It is slow and can not consistently resolve dependencies, [yarn] solves those problems and it is compatible to npm as well.

我们建议使用[yarn]，因为npm有一些问题。它是缓慢的，不能始终如一地解决依赖关系，[yarn]解决了这些问题，而且它也与npm兼容。

### RUN THE APPLICATION

Open the command line and run the following command:

打开命令行并运行以下命令：

```bash
npm start
```

Once the application compiled, you can browse http://localhost:4200 in your browser. ASP.NET Zero also has also HMR (Hot Module Replacement) enabled. You can use the following command (instead of npm start) to enable HMR on development time:

一旦应用程序编译，您就可以在浏览器中浏览http://localhost:4200 ASP.NET Zero也启用了HMR（热模块替换）。您可以使用以下命令（而不是npm start）来启用HMR开发时间：

```bash
npm run hmr
```

### LOGIN

All ready.. just run your solution to enter to the login page:

都准备好了. .只需运行您的解决方案，进入登录页面：

![login-screen](https://raw.githubusercontent.com/aspnetzero/documents/master/doc/images/login-screen-3.png)

If multi-tenancy is enabled, you will see the current tenant and a **change** link. If so, click to Change like and enter **default** as tenant name. If you leave it empty, you login as the host admin user. Then enter **admin** as user name and **123qwe** as password. You should change your password at first login.

如果启用了多租户，您将看到当前租户和变更链接。如果是这样，单击以更改为租户名称，并以默认方式输入默认值。如果您让它空着，您就会以主机管理员的身份登录。然后输入admin作为用户名和123qwe作为密码。您应该在第一次登录时更改您的密码。

### APPLICATION UI

After you login to the application, you will see the sample dashboard screen:

在登录到应用程序之后，您将看到示例仪表板屏幕：

![dashboardV3](https://raw.githubusercontent.com/aspnetzero/documents/master/doc/images/dashboardV3.png)

## ASP.NET ZERO POWER TOOLS

You can download our rapid application development tool from the following link:

您可以从以下链接下载我们的快速应用程序开发工具：

<https://marketplace.visualstudio.com/items?itemName=Volosoft.AspNetZeroPowerTools>

## 更多

Your solution is up and working. See [development guide for Xamarin] application, [development guide] document for more information.

你的解决方案已经运行了。那么请参阅[Xamarin应用程序开发指南]，了解更多信息的[开发指南文档]。

[Xamarin应用程序开发指南]:(https://aspnetzero.com/Documents/Development-Guide-Xamarin)
[开发指南文档]:(https://aspnetzero.com/Documents/Development-Guide-Xamarin)
[development guide]:(https://aspnetzero.com/Documents/Development-Guide-Angular)
[development guide]:(https://aspnetzero.com/Documents/Development-Guide-Angular)
