---
title: ASP.NET Core 中的文件提供程序
author: guardrex
description: 了解 ASP.NET Core 如何通过文件提供程序来抽象化文件系统访问。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 03/30/2019
uid: fundamentals/file-providers
ms.openlocfilehash: 93eb48d81a853061a874641e84b4875849690a93
ms.sourcegitcommit: 5b0eca8c21550f95de3bb21096bd4fd4d9098026
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/27/2019
ms.locfileid: "64886612"
---
# <a name="file-providers-in-aspnet-core"></a>ASP.NET Core 中的文件提供程序

作者：[Steve Smith](https://ardalis.com/) 和 [Luke Latham](https://github.com/guardrex)

ASP.NET Core 通过文件提供程序来抽象化文件系统访问。 在 ASP.NET Core 框架中使用文件提供程序：

* [IHostingEnvironment](/dotnet/api/microsoft.extensions.hosting.ihostingenvironment) 将应用的内容根和 Web 根作为 `IFileProvider` 类型公开。
* [静态文件中间件](xref:fundamentals/static-files)使用文件提供程序来查找静态文件。
* [Razor](xref:mvc/views/razor) 使用文件提供程序来查找页面和视图。
* .NET Core 工具使用文件提供程序和 glob 模式来指定应该发布哪些文件。

[查看或下载示例代码](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/file-providers/samples)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="file-provider-interfaces"></a>文件提供程序接口

主接口为 [IFileProvider](/dotnet/api/microsoft.extensions.fileproviders.ifileprovider)。 `IFileProvider` 公开方法以实现以下目的：

* 获取文件信息 ([IFileInfo](/dotnet/api/microsoft.extensions.fileproviders.ifileinfo))。
* 获取目录信息 ([IDirectoryContents](/dotnet/api/microsoft.extensions.fileproviders.idirectorycontents))。
* 设置更改通知（使用 [IChangeToken](/dotnet/api/microsoft.extensions.primitives.ichangetoken)）。

`IFileInfo` 提供用于处理文件的方法和属性：

* [Exists](/dotnet/api/microsoft.extensions.fileproviders.ifileinfo.exists)
* [IsDirectory](/dotnet/api/microsoft.extensions.fileproviders.ifileinfo.isdirectory)
* [名称](/dotnet/api/microsoft.extensions.fileproviders.ifileinfo.name)
* [Length](/dotnet/api/microsoft.extensions.fileproviders.ifileinfo.length)（以字节为单位）
* [LastModified](/dotnet/api/microsoft.extensions.fileproviders.ifileinfo.lastmodified) 日期

可以使用 [IFileInfo.CreateReadStream](/dotnet/api/microsoft.extensions.fileproviders.ifileinfo.createreadstream) 方法从文件读取内容。

示例应用演示了如何在 `Startup.ConfigureServices` 中配置文件提供程序，以便通过[依赖关系注入](xref:fundamentals/dependency-injection)在应用中使用。

## <a name="file-provider-implementations"></a>文件提供程序实现

`IFileProvider` 有三种实现。

| 实现 | 说明 |
| -------------- | ----------- |
| [PhysicalFileProvider](#physicalfileprovider) | 物理提供程序用于访问系统的物理文件。 |
| [ManifestEmbeddedFileProvider](#manifestembeddedfileprovider) | 清单嵌入式提供程序用于访问程序集中嵌入的文件。 |
| [CompositeFileProvider](#compositefileprovider) | 复合式提供程序提供对一个或多个其他提供程序中的文件和目录的合并访问。 |

### <a name="physicalfileprovider"></a>PhysicalFileProvider

[PhysicalFileProvider](/dotnet/api/microsoft.extensions.fileproviders.physicalfileprovider) 提供对物理文件系统的访问。 `PhysicalFileProvider` 使用（物理提供程序的）[System.IO.File](/dotnet/api/system.io.file) 类型并将所有路径范围限制到目录及其子目录。 此范围界定可防止访问指定目录和子目录之外的文件系统。 实例化此提供程序时，必须提供目录路径并将其作为使用提供程序发出的所有请求的基路径。 可以直接实例化 `PhysicalFileProvider` 提供程序，也可以通过[依赖关系注入](xref:fundamentals/dependency-injection)请求构造函数中的 `IFileProvider`。

**静态类型**

下面的代码演示如何创建 `PhysicalFileProvider` 并用它来获取目录内容和文件信息：

```csharp
var provider = new PhysicalFileProvider(applicationRoot);
var contents = provider.GetDirectoryContents(string.Empty);
var fileInfo = provider.GetFileInfo("wwwroot/js/site.js");
```

前面的示例中的类型：

* `provider` 是 `IFileProvider`。
* `contents` 是 `IDirectoryContents`。
* `fileInfo` 是 `IFileInfo`。

文件提供程序可用于循环访问 `applicationRoot` 指定的目录或调用 `GetFileInfo` 来获取文件信息。 该文件提供程序无法访问 `applicationRoot` 目录外部的任何内容。

示例应用使用 [IHostingEnvironment.ContentRootFileProvider](/dotnet/api/microsoft.extensions.hosting.ihostingenvironment.contentrootfileprovider) 在应用的 `Startup.ConfigureServices` 类中创建提供程序：

```csharp
var physicalProvider = _env.ContentRootFileProvider;
```

**使用依赖关系注入获取文件提供程序类型**

将提供程序注入任何类构造函数，并将其分配给本地字段。 在整个类的方法中使用该字段来访问文件。

在示例应用中，`IndexModel` 类接收 `IFileProvider` 实例，获取应用的基路径的目录内容。

Pages/Index.cshtml.cs：

[!code-csharp[](file-providers/samples/2.x/FileProviderSample/Pages/Index.cshtml.cs?name=snippet1)]

在页中循环访问 `IDirectoryContents`。

Pages/Index.cshtml：

[!code-cshtml[](file-providers/samples/2.x/FileProviderSample/Pages/Index.cshtml?name=snippet1)]

### <a name="manifestembeddedfileprovider"></a>ManifestEmbeddedFileProvider

[ManifestEmbeddedFileProvider](/dotnet/api/microsoft.extensions.fileproviders.manifestembeddedfileprovider) 用于访问程序集内嵌入的文件。 `ManifestEmbeddedFileProvider` 使用编译到程序集中的某个清单来重建嵌入的文件的原始路径。

若要生成嵌入的文件清单，请将 `<GenerateEmbeddedFilesManifest>` 属性设置为 `true`。 指定要使用 [&lt;EmbeddedResource&gt;](/dotnet/core/tools/csproj#default-compilation-includes-in-net-core-projects) 嵌入的文件：

[!code-csharp[](file-providers/samples/2.x/FileProviderSample/FileProviderSample.csproj?highlight=6,14)]

使用 [glob 模式](#glob-patterns)指定要嵌入到程序集中的一个或多个文件。

示例应用创建 `ManifestEmbeddedFileProvider` 并将当前正在执行的程序集传递给其构造函数。

*Startup.cs*：

```csharp
var manifestEmbeddedProvider = 
    new ManifestEmbeddedFileProvider(Assembly.GetEntryAssembly());
```

其他重载使你能够：

* 指定相对文件路径。
* 将文件范围限制到上次修改日期。
* 为包含嵌入文件清单的嵌入资源命名。

| 重载 | 说明 |
| -------- | ----------- |
| [ManifestEmbeddedFileProvider(Assembly, String)](/dotnet/api/microsoft.extensions.fileproviders.manifestembeddedfileprovider.-ctor#Microsoft_Extensions_FileProviders_ManifestEmbeddedFileProvider__ctor_System_Reflection_Assembly_System_String_) | 接受一个可选的 `root` 相对路径参数。 指定 `root` 将对 [GetDirectoryContents](/dotnet/api/microsoft.extensions.fileproviders.ifileprovider.getdirectorycontents) 的调用范围限制为提供的路径下的那些资源。 |
| [ManifestEmbeddedFileProvider(Assembly, String, DateTimeOffset)](/dotnet/api/microsoft.extensions.fileproviders.manifestembeddedfileprovider.-ctor#Microsoft_Extensions_FileProviders_ManifestEmbeddedFileProvider__ctor_System_Reflection_Assembly_System_String_System_DateTimeOffset_) | 接受一个可选的 `root` 相对路径参数和一个 `lastModified` 日期 ([DateTimeOffset](/dotnet/api/system.datetimeoffset)) 参数。 `lastModified` 日期限制 [IFileProvider](/dotnet/api/microsoft.extensions.fileproviders.ifileprovider) 返回的 [IFileInfo](/dotnet/api/microsoft.extensions.fileproviders.ifileinfo) 实例的上次修改日期范围。 |
| [ManifestEmbeddedFileProvider(Assembly, String, String, DateTimeOffset)](/dotnet/api/microsoft.extensions.fileproviders.manifestembeddedfileprovider.-ctor#Microsoft_Extensions_FileProviders_ManifestEmbeddedFileProvider__ctor_System_Reflection_Assembly_System_String_System_String_System_DateTimeOffset_) | 接受一个可选的 `root` 相对路径、`lastModified` 日期和 `manifestName` 参数。 `manifestName` 表示包含清单的嵌入资源的名称。 |

### <a name="compositefileprovider"></a>CompositeFileProvider

[CompositeFileProvider](/dotnet/api/microsoft.extensions.fileproviders.compositefileprovider) 结合了 `IFileProvider` 实例，以便公开一个接口来处理来自多个提供程序的文件。 创建 `CompositeFileProvider` 时，将一个或多个 `IFileProvider` 实例传递给其构造函数。

在示例应用中，`PhysicalFileProvider` 和 `ManifestEmbeddedFileProvider` 向在应用的服务容器中注册的 `CompositeFileProvider` 提供文件：

[!code-csharp[](file-providers/samples/2.x/FileProviderSample/Startup.cs?name=snippet1)]

## <a name="watch-for-changes"></a>监视更改

[IFileProvider.Watch](/dotnet/api/microsoft.extensions.fileproviders.ifileprovider.watch) 方法提供了一个方案来监视一个或多个文件或目录是否发生更改。 `Watch` 接受路径字符串，它可以使用 [glob 模式](#glob-patterns)指定多个文件。 `Watch` 返回 [IChangeToken](/dotnet/api/microsoft.extensions.primitives.ichangetoken)。 更改令牌会公开以下内容：

* [HasChanged](/dotnet/api/microsoft.extensions.primitives.ichangetoken.haschanged)：可检查此属性以确定是否已发生更改。
* [RegisterChangeCallback](/dotnet/api/microsoft.extensions.primitives.ichangetoken.registerchangecallback)：检测到指定的路径字符串发生更改时调用此属性。 每个更改令牌仅调用其关联的回调来响应单个更改。 若要启用持续监视，请使用 [TaskCompletionSource](/dotnet/api/system.threading.tasks.taskcompletionsource-1)（如下所示）或重新创建 `IChangeToken` 实例以响应更改。

在示例应用中，WatchConsole 控制台应用配置为每次修改了文本文件时显示一条消息：

[!code-csharp[](file-providers/samples/2.x/WatchConsole/Program.cs?name=snippet1&highlight=1-2,16,19-20)]

某些文件系统（例如 Docker 容器和网络共享）可能无法可靠地发送更改通知。 将 `DOTNET_USE_POLLING_FILE_WATCHER` 环境变量设置为 `1` 或 `true` 以每隔四秒轮询一次文件系统，确认是否发生更改（不可配置）。

## <a name="glob-patterns"></a>glob 模式

文件系统路径使用名为 glob（或通配）模式的通配符模式。 使用这些模式指定文件的组。 两个通配符分别是 `*` 和 `**`：

**`*`**  
匹配当前文件夹级别的任何内容、任何文件名或任何文件扩展名。 匹配由文件路径中的 `/` 和 `.` 字符终止。

**`**`**  
匹配多个目录级别的任何内容。 可用于以递归方式匹配目录层次结构中的许多文件。

**glob 模式示例**

**`directory/file.txt`**  
匹配特定目录中的特定文件。

**`directory/*.txt`**  
匹配特定目录中带 .txt 扩展名的所有文件。

**`directory/*/appsettings.json`**  
匹配正好位于“目录”文件夹中下一级目录中的所有 `appsettings.json` 文件。

**`directory/**/*.txt`**  
匹配在“目录”文件夹下任何位置找到的带 .txt 扩展名的所有文件。
