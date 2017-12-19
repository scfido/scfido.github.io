---
   layout: default
   title: Visual Studio Code开发环境中管理User Secrets
   tags: Git
---

# {{ page.title }}
{{ page.date | date: "%Y-%m-%d" }}

# 简介

项目中常常会把一些机密数据以明文写入源代码或者配置文件中，例如数据库连接字符串、登陆用户名密码等，这些信息如果上传到源代码管理器就很难控制知晓范围了，更糟的情况是开源项目把机密信息上传到Github。如何安全的存储这些用户机密信息呢，ASP.NET Core提供SecretManager来管理机密数据。并且Visual Studio还提供了非常便利的工具。从下图可以看出在Web项目点击右键，就可以进入用户机密文件编辑器。

![](/assets/misc/VSCodeUserSecrets/img/2017-12-19-17-32-12.png)

但是在Visual Studio Code开发环境下如何使用该特性呢？本文将详细介绍。

# 开始
## 创建项目
在一个空目录下创建web模板项目。

```
dotnet new web
```

## 安装Microsoft.Extensions.SecretManager.Tools
安装Microsoft.Extensions.SecretManager.Tools依赖库。

```
dotnet add package Microsoft.Extensions.SecretManager.Tools
dotnet restore
```
## 设置机密信息
我们添加QQ登陆使用到的AppId和AppKey两个参数。
```
dotnet user-secrets set connect.qq.appid 123
dotnet user-secrets set connect.qq.appkey 456
```

如果设置参数时遇到这个错误。
![](/assets/misc/VSCodeUserSecrets/img/2017-12-19-17-54-59.png)

打开项目`.csproj`文件，添加以下内容。
```xml
<ItemGroup>
    <DotNetCliToolReference Include="Microsoft.Extensions.SecretManager.Tools" Version="2.0.0" />
</ItemGroup>
```

再执行设置机密参数命令，又遇到缺少`UserSecretsId`的错误。
![](/assets/misc/VSCodeUserSecrets/img/2017-12-19-18-06-30.png)

这个简单，还是打开项目`.csproj`文件，添加以下内容，`UserSecretsId`的值任意设置就好了，这串字符将作为机密文件的目录名存到你的用户目录。
```xml
  <PropertyGroup>
    <TargetFramework>netcoreapp2.0</TargetFramework>
    <UserSecretsId>aspnet-Demo-04C6939F-E672-4E56-B4A5-5F064EB67F23</UserSecretsId>
  </PropertyGroup>
```
好了，这下成功了。
![](/assets/misc/VSCodeUserSecrets/img/2017-12-19-18-12-59.png)

## 使用机密信息
机密信息并非存与我们项目目录中，如何读取呢？

打开`Startup.cs`，修改构造函数。  
> 注意，有些模板生成的项目使用的是`Startup(Configuration configuration)`重载，要更换成下面的内容。

看到了吗？我加了一行`.AddUserSecrets<Startup>();`来读取机密配置，他会自动到你的用户目录中读取。

```cs
public Startup(IHostingEnvironment env)
{
    var builder = new ConfigurationBuilder()
        .SetBasePath(env.ContentRootPath)
        //.AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
        //.AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
        .AddEnvironmentVariables()
        .AddUserSecrets<Startup>();

    var configuration = builder.Build();
}
```

读取配置就很简单了。
```cs
Console.WriteLine($"AppId:{configuration["connect.qq.appid"]}");
Console.WriteLine($"AppKey:{configuration["connect.qq.appkey"]}");
```

![](/assets/misc/VSCodeUserSecrets/img/2017-12-19-18-27-26.png)

到此我们已经成功的在Visual Studio Code开发环境下设置和读取用户机密信息。

## User Secrets文件存放路径

- Windows: `%APPDATA%\microsoft\UserSecrets\<userSecretsId>\secrets.json`
- Linux: `~/.microsoft/usersecrets/<userSecretsId>/secrets.json`
- Mac: `~/.microsoft/usersecrets/<userSecretsId>/secrets.json`

## 参考文档
- Safe storage of app secrets during development in ASP.NET Core  
https://docs.microsoft.com/en-us/aspnet/core/security/app-secrets?tabs=visual-studio-code
