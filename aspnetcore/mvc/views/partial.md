---
title: ASP.NET Core 中的分部视图
author: ardalis
description: 了解如何使用分部视图来分解大型标记文件，并减少 ASP.NET Core 应用程序中跨网页的常见标记重复情况。
ms.author: riande
ms.custom: mvc
ms.date: 06/12/2019
uid: mvc/views/partial
ms.openlocfilehash: 901fd52f89969141713e443890781a77308bd901
ms.sourcegitcommit: 335a88c1b6e7f0caa8a3a27db57c56664d676d34
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/12/2019
ms.locfileid: "67034914"
---
# <a name="partial-views-in-aspnet-core"></a>ASP.NET Core 中的分部视图

作者：[Steve Smith](https://ardalis.com/)、[Luke Latham](https://github.com/guardrex)、[Maher JENDOUBI](https://twitter.com/maherjend)、[Rick Anderson](https://twitter.com/RickAndMSFT) 和 [Scott Sauber](https://twitter.com/scottsauber)

分部视图是 [Razor](xref:mvc/views/razor) 标记文件 (.cshtml  )，它在另一个标记文件呈现的输出中呈现 HTML 输出  。

::: moniker range=">= aspnetcore-2.1"

在开发 MVC 应用程序（其中标记文件称为“视图”  ）或 Razor Pages 应用程序（其中标记文件称为“页”）时，均会使用术语  “分部视图”  。 本主题通常将 MVC 视图和 Razor Pages 页面称为“标记文件”  。

::: moniker-end

[查看或下载示例代码](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/views/partial/sample)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="when-to-use-partial-views"></a>何时使用分部视图

分部视图是执行下列操作的有效方式：

* 将大型标记文件分解为更小的组件。

  在由多个逻辑部分组成的大型复杂标记文件中，在分部视图中处理隔开的每个部分是有利的。 标记文件中的代码是可管理的，因为标记仅包含整体页面结构和对分部视图的引用。
* 减少跨标记文件中常见标记内容的重复。

  当在标记文件中使用相同的标记元素时，分部视图会将重复的标记内容移到一个分部视图文件中。 在分部视图中更改标记后，它会更新使用该分部视图的标记文件呈现的输出。

不应使用分部视图来维护常见布局元素。 常见布局元素应在 [_Layout.cshtml](xref:mvc/views/layout) 文件中指定。

请勿使用需要复杂呈现逻辑或代码执行来呈现标记的分部视图。 使用[视图组件](xref:mvc/views/view-components)而不是分部视图。

## <a name="declare-partial-views"></a>声明分部视图

::: moniker range=">= aspnetcore-2.0"

分部视图是在 Views  文件夹 (MVC) 或 Pages  文件夹 (Razor Pages) 中维护的 .cshtml  标记文件。

在 ASP.NET Core MVC 中，控制器的 <xref:Microsoft.AspNetCore.Mvc.ViewResult> 能够返回视图或分部视图。 在 Razor Pages 中，<xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel> 可以返回表示为 <xref:Microsoft.AspNetCore.Mvc.PartialViewResult> 对象的分部视图。 [引用分部视图](#reference-a-partial-view)部分介绍了引用和呈现分部视图。

与 MVC 视图或页面呈现不同，分部视图不会运行 _ViewStart.cshtml  。 有关 _ViewStart.cshtml  的详细信息，请参阅 <xref:mvc/views/layout>。

分部视图的文件名通常以下划线 (`_`) 开头。 虽然未强制要求遵从此命名约定，但它有助于直观地将分部视图与视图和页面区分开来。

::: moniker-end

::: moniker range="< aspnetcore-2.0"

分部视图是在 Views  文件夹中维护的 .cshtml  标记文件。

控制器的 <xref:Microsoft.AspNetCore.Mvc.ViewResult> 能够返回视图或分部视图。 [引用分部视图](#reference-a-partial-view)部分介绍了引用和呈现分部视图。

与 MVC 视图呈现不同，分部视图不会运行 _ViewStart.cshtml  。 有关 _ViewStart.cshtml  的详细信息，请参阅 <xref:mvc/views/layout>。

分部视图的文件名通常以下划线 (`_`) 开头。 虽然未强制要求遵从此命名约定，但它有助于直观地将分部视图与视图区分开来。

::: moniker-end

## <a name="reference-a-partial-view"></a>引用分部视图

::: moniker range=">= aspnetcore-2.0"

### <a name="use-a-partial-view-in-a-razor-pages-pagemodel"></a>在 Razor Pages PageModel 中使用分部视图

在 ASP.NET Core 2.0 或 2.1 中，以下处理程序方法将 \_AuthorPartialRP.cshtml 分部视图呈现给响应  ：

```csharp
public IActionResult OnGetPartial() =>
    new PartialViewResult
    {
        ViewName = "_AuthorPartialRP",
        ViewData = ViewData,
    };
```

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

在 ASP.NET Core 2.2 或更高版本中，处理程序方法也可以调用 <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageBase.Partial*> 方法来生成 `PartialViewResult` 对象：

[!code-csharp[](partial/sample/PartialViewsSample/Pages/DiscoveryRP.cshtml.cs?name=snippet_OnGetPartial)]

::: moniker-end

### <a name="use-a-partial-view-in-a-markup-file"></a>在标记文件中使用分部视图

::: moniker range=">= aspnetcore-2.1"

在标记文件中，有多种方法可引用分部视图。 我们建议应用程序使用以下异步呈现方法之一：

* [部分标记帮助程序](#partial-tag-helper)
* [异步 HTML 帮助程序](#asynchronous-html-helper)

::: moniker-end

::: moniker range="< aspnetcore-2.1"

在标记文件中，有两种方法可引用分部视图：

* [异步 HTML 帮助程序](#asynchronous-html-helper)
* [同步 HTML 帮助程序](#synchronous-html-helper)

我们建议应用程序使用[异步 HTML 帮助程序](#asynchronous-html-helper)。

::: moniker-end

::: moniker range=">= aspnetcore-2.1"

### <a name="partial-tag-helper"></a>分部标记帮助程序

[分部标记帮助程序](xref:mvc/views/tag-helpers/builtin-th/partial-tag-helper)要求 ASP.NET Core 2.1 或更高版本。

分部标记帮助程序会异步呈现内容并使用类似 HTML 的语法：

```cshtml
<partial name="_PartialName" />
```

当存在文件扩展名时，标记帮助程序会引用分部视图，该视图必须与调用分部视图的标记文件位于同一文件夹中：

```cshtml
<partial name="_PartialName.cshtml" />
```

以下示例从应用程序根目录引用分部视图。 以波形符斜杠 (`~/`) 或斜杠 (`/`) 开头的路径指代应用程序根目录：

**Razor 页面**

```cshtml
<partial name="~/Pages/Folder/_PartialName.cshtml" />
<partial name="/Pages/Folder/_PartialName.cshtml" />
```

**MVC**

```cshtml
<partial name="~/Views/Folder/_PartialName.cshtml" />
<partial name="/Views/Folder/_PartialName.cshtml" />
```

以下示例引用使用相对路径的分部视图：

```cshtml
<partial name="../Account/_PartialName.cshtml" />
```

有关更多信息，请参见<xref:mvc/views/tag-helpers/builtin-th/partial-tag-helper>。

::: moniker-end

### <a name="asynchronous-html-helper"></a>异步 HTML 帮助程序

使用 HTML 帮助程序时，最佳做法是使用 <xref:Microsoft.AspNetCore.Mvc.Rendering.HtmlHelperPartialExtensions.PartialAsync*>。 `PartialAsync` 返回包含在 <xref:System.Threading.Tasks.Task%601> 中的 <xref:Microsoft.AspNetCore.Html.IHtmlContent> 类型。 通过在等待的调用前添加 `@` 字符前缀来引用该方法：

```cshtml
@await Html.PartialAsync("_PartialName")
```

当存在文件扩展名时，HTML 帮助程序会引用分部视图，该视图必须与调用分部视图的标记文件位于同一文件夹中：

```cshtml
@await Html.PartialAsync("_PartialName.cshtml")
```

以下示例从应用程序根目录引用分部视图。 以波形符斜杠 (`~/`) 或斜杠 (`/`) 开头的路径指代应用程序根目录：

::: moniker range=">= aspnetcore-2.1"

**Razor 页面**

```cshtml
@await Html.PartialAsync("~/Pages/Folder/_PartialName.cshtml")
@await Html.PartialAsync("/Pages/Folder/_PartialName.cshtml")
```

**MVC**

::: moniker-end

```cshtml
@await Html.PartialAsync("~/Views/Folder/_PartialName.cshtml")
@await Html.PartialAsync("/Views/Folder/_PartialName.cshtml")
```

以下示例引用使用相对路径的分部视图：

```cshtml
@await Html.PartialAsync("../Account/_LoginPartial.cshtml")
```

或者，也可以使用 <xref:Microsoft.AspNetCore.Mvc.Rendering.HtmlHelperPartialExtensions.RenderPartialAsync*> 呈现分部视图。 此方法不返回 <xref:Microsoft.AspNetCore.Html.IHtmlContent>。 它将呈现的输出直接流式传输到响应。 因为该方法不返回结果，所以必须在 Razor 代码块内调用它：

[!code-cshtml[](partial/sample/PartialViewsSample/Views/Home/Discovery.cshtml?name=snippet_RenderPartialAsync)]

由于 `RenderPartialAsync` 流式传输呈现的内容，因此在某些情况下它可提供更好的性能。 在性能起关键作用的情况下，使用两种方法对页面进行基准测试，并使用生成更快响应的方法。

### <a name="synchronous-html-helper"></a>同步 HTML 帮助程序

<xref:Microsoft.AspNetCore.Mvc.Rendering.HtmlHelperPartialExtensions.Partial*> 和 <xref:Microsoft.AspNetCore.Mvc.Rendering.HtmlHelperPartialExtensions.RenderPartial*> 分别是 `PartialAsync` 和 `RenderPartialAsync` 的同步等效项。 但不建议使用同步等效项，因为可能会出现死锁的情况。 同步方法针对以后版本中的删除功能。

> [!IMPORTANT]
> 如果需要执行代码，请使用[视图组件](xref:mvc/views/view-components)，而不是使用分部视图。

::: moniker range=">= aspnetcore-2.1"

调用 `Partial` 或 `RenderPartial` 会导致 Visual Studio 分析器警告。 例如，使用 `Partial` 会产生以下警告消息：

> 使用 IHtmlHelper.Partial 可能会导致应用程序死锁。 考虑使用 &lt;分部&gt; 标记帮助程序或 IHtmlHelper.PartialAsync。

将对 `@Html.Partial` 的调用替换为 `@await Html.PartialAsync` 或[分部标记帮助程序](xref:mvc/views/tag-helpers/builtin-th/partial-tag-helper)。 有关分部标记帮助程序迁移的详细信息，请参阅[从 HTML 帮助程序迁移](xref:mvc/views/tag-helpers/builtin-th/partial-tag-helper#migrate-from-an-html-helper)。

::: moniker-end

## <a name="partial-view-discovery"></a>分部视图发现

如果按名称（无文件扩展名）引用分部视图，则按所述顺序搜索以下位置：

::: moniker range=">= aspnetcore-2.1"

**Razor 页面**

1. 当前正在执行页面的文件夹
1. 该页面文件夹上方的目录图
1. `/Shared`
1. `/Pages/Shared`
1. `/Views/Shared`

**MVC**

::: moniker-end

::: moniker range=">= aspnetcore-2.0"

1. `/Areas/<Area-Name>/Views/<Controller-Name>`
1. `/Areas/<Area-Name>/Views/Shared`
1. `/Views/Shared`
1. `/Pages/Shared`

::: moniker-end

::: moniker range="< aspnetcore-2.0"

1. `/Areas/<Area-Name>/Views/<Controller-Name>`
1. `/Areas/<Area-Name>/Views/Shared`
1. `/Views/Shared`

::: moniker-end

以下约定适用于分部视图发现：

* 当分部视图位于不同的文件夹中时，允许使用具有相同文件名的不同分部视图。
* 当按名称（无文件扩展名）引用分部视图且分部视图出现在调用方的文件夹和  文件夹中时，调用方文件夹中的分部视图会提供分部视图。 如果调用方文件夹中不存在分部视图，则会从  文件夹中提供分部视图。  文件夹中的分部视图称为“共享分部视图”  或“默认分部视图”  。
* 可以链接分部视图  &mdash;如果调用没有形成循环引用，则分部视图可以调用另一个分部视图。 相对路径始终相对于当前文件，而不是相对于文件的根视图或父视图。

> [!NOTE]
> 分部视图中定义的 [Razor](xref:mvc/views/razor) `section` 对父标记文件不可见。 `section` 仅对定义它时所在的分部视图可见。

## <a name="access-data-from-partial-views"></a>通过分部视图访问数据

实例化分部视图时，它会获得父视图的 `ViewData` 字典的副本  。 在分部视图内对数据所做的更新不会保存到父视图中。 在分部视图中的 `ViewData` 更改会在分部视图返回时丢失。

以下示例演示如何将 [ViewDataDictionary](/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.viewdatadictionary)的实例传递给分部视图：

```cshtml
@await Html.PartialAsync("_PartialName", customViewData)
```

还可将模型传入分部视图。 模型可以是自定义对象。 你可以使用 `PartialAsync`（向调用方呈现内容块）或 `RenderPartialAsync`（将内容流式传输到输出）传递模型：

```cshtml
@await Html.PartialAsync("_PartialName", model)
```

::: moniker range=">= aspnetcore-2.1"

**Razor 页面**

示例应用程序中的以下标记来自 *Pages/ArticlesRP/ReadRP.cshtml* 页面。 此页包含两个分部视图。 第二个分部视图将模型和 `ViewData` 传入分部视图。 `ViewDataDictionary` 构造函数重载可用于传递新 `ViewData` 字典，同时保留现有的 `ViewData` 字典。

[!code-cshtml[](partial/sample/PartialViewsSample/Pages/ArticlesRP/ReadRP.cshtml?name=snippet_ReadPartialViewRP&highlight=5,15-20)]

 Pages/Shared/_AuthorPartialRP.cshtml 是  ReadRP.cshtml 标记文件引用的第一个分部视图：

[!code-cshtml[](partial/sample/PartialViewsSample/Pages/Shared/_AuthorPartialRP.cshtml)]

 Pages/ArticlesRP/_ArticleSectionRP.cshtml 是  ReadRP.cshtml 标记文件引用的第二个分部视图：

[!code-cshtml[](partial/sample/PartialViewsSample/Pages/ArticlesRP/_ArticleSectionRP.cshtml)]

**MVC**

::: moniker-end

示例应用中的以下标记显示  Views/Articles/Read.cshtml 视图。 此视图包含两个分部视图。 第二个分部视图将模型和 `ViewData` 传入分部视图。 `ViewDataDictionary` 构造函数重载可用于传递新 `ViewData` 字典，同时保留现有的 `ViewData` 字典。

[!code-cshtml[](partial/sample/PartialViewsSample/Views/Articles/Read.cshtml?name=snippet_ReadPartialView&highlight=5,15-20)]

 Views/Shared/_AuthorPartial.cshtml 是  Read.cshtml 标记文件引用的第一个分部视图：

[!code-cshtml[](partial/sample/PartialViewsSample/Views/Shared/_AuthorPartial.cshtml)]

 Views/Articles/_ArticleSection.cshtml 是  Read.cshtml 标记文件引用的第二个分部视图：

[!code-cshtml[](partial/sample/PartialViewsSample/Views/Articles/_ArticleSection.cshtml)]

在运行时，分部视图在父标记文件呈现的输出中呈现，而父标记文件本身在共享的 _Layout.cshtml 内呈现  。 第一个分部视图呈现文章作者的姓名和发布日期：

> Abraham Lincoln
>
> 来自 &lt;共享分部视图文件路径&gt; 的分部视图。
> 1863 年 11 月 19 日中午 12:00:00

第二个分部视图呈现文章的各部分：

> 第一节索引：0
>
> 八十七年前...
>
> 第二节索引：1
>
> 如今，我们正在进行一场伟大的内战，考验着......
>
> 第三节索引：2
>
> 然而，从更广泛的意义上说，我们无法奉献...

## <a name="additional-resources"></a>其他资源

::: moniker range=">= aspnetcore-2.1"

* <xref:mvc/views/razor>
* <xref:mvc/views/tag-helpers/intro>
* <xref:mvc/views/tag-helpers/builtin-th/partial-tag-helper>
* <xref:mvc/views/view-components>
* <xref:mvc/controllers/areas>

::: moniker-end

::: moniker range="< aspnetcore-2.1"

* <xref:mvc/views/razor>
* <xref:mvc/views/view-components>
* <xref:mvc/controllers/areas>

::: moniker-end
