---
title: ASP.NET Core 表单中的标记帮助程序
author: rick-anderson
description: 介绍与表单配合使用的内置标记帮助程序。
ms.author: riande
ms.custom: mvc
ms.date: 04/06/2019
uid: mvc/views/working-with-forms
ms.openlocfilehash: a07bb4f539c8bd38b08402c598924e14c748921d
ms.sourcegitcommit: 8516b586541e6ba402e57228e356639b85dfb2b9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/11/2019
ms.locfileid: "67815239"
---
# <a name="tag-helpers-in-forms-in-aspnet-core"></a>ASP.NET Core 表单中的标记帮助程序

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)、[N. Taylor Mullen](https://github.com/NTaylorMullen)、[Dave Paquette](https://twitter.com/Dave_Paquette) 和 [Jerrie Pelser](https://github.com/jerriep)

本文档演示如何使用表单和表单中常用的 HTML 元素。 HTML [Form](https://www.w3.org/TR/html401/interact/forms.html) 元素提供 Web 应用用于向服务器回发数据的主要机制。 本文档的大部分内容介绍[标记帮助程序](tag-helpers/intro.md)及其如何帮助高效创建可靠的 HTML 表单。 建议在阅读本文档前先阅读[标记帮助程序简介](tag-helpers/intro.md)。

在很多情况下，HTML 帮助程序为特定标记帮助程序提供了一种替代方法，但标记帮助程序不会替代 HTML 帮助程序，且并非每个 HTML 帮助程序都有对应的标记帮助程序，认识到这点也很重要。 如果存在 HTML 帮助程序替代项，文中会提到。

<a name="my-asp-route-param-ref-label"></a>

## <a name="the-form-tag-helper"></a>表单标记帮助程序

[表单](https://www.w3.org/TR/html401/interact/forms.html)标记帮助程序：

* 为 MVC 控制器操作或命名路由生成 HTML [\<FORM>](https://www.w3.org/TR/html401/interact/forms.html) `action` 属性值

* 生成隐藏的[请求验证令牌](/aspnet/mvc/overview/security/xsrfcsrf-prevention-in-aspnet-mvc-and-web-pages)，防止跨站点请求伪造（在 HTTP Post 操作方法中与 `[ValidateAntiForgeryToken]` 属性配合使用时）

* 提供 `asp-route-<Parameter Name>` 属性，其中 `<Parameter Name>` 添加到路由值。 `Html.BeginForm` 和 `Html.BeginRouteForm` 的 `routeValues` 参数提供类似的功能。

* 具有 HTML 帮助程序替代项 `Html.BeginForm` 和 `Html.BeginRouteForm`

示例:

[!code-CSHTML[](working-with-forms/sample/final/Views/Demo/RegisterFormOnly.cshtml)]

上述表单标记帮助程序生成以下 HTML：

```HTML
<form method="post" action="/Demo/Register">
    <!-- Input and Submit elements -->
    <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>">
</form>
```

MVC 运行时通过表单标记帮助程序属性 `asp-controller` 和 `asp-action` 生成 `action` 属性值。 表单标记帮助程序还会生成隐藏的[请求验证令牌](/aspnet/mvc/overview/security/xsrfcsrf-prevention-in-aspnet-mvc-and-web-pages)，防止跨站点请求伪造（在 HTTP Post 操作方法中与 `[ValidateAntiForgeryToken]` 属性配合使用时）。 保护纯 HTML 表单免受跨站点请求伪造的影响很难，但表单标记帮助程序可提供此服务。

### <a name="using-a-named-route"></a>使用命名路由

`asp-route` 标记帮助程序属性还可为 HTML `action` 属性生成标记。 具有名为 `register` 的[路由](../../fundamentals/routing.md)的应用可将以下标记用于注册页：

[!code-HTML[](../../mvc/views/working-with-forms/sample/final/Views/Demo/RegisterRoute.cshtml)]

*Views/Account* 文件夹中的许多视图（在新建使用*个人用户帐户*身份验证的 Web 应用时生成）包含 [asp-route-returnurl](xref:mvc/views/working-with-forms) 属性：

```cshtml
<form asp-controller="Account" asp-action="Login"
     asp-route-returnurl="@ViewData["ReturnUrl"]"
     method="post" class="form-horizontal" role="form">
```

>[!NOTE]
>使用内置模板时，`returnUrl` 仅会在用户尝试访问授权资源，但未验证身份或未获得授权的情况下自动填充。 如果尝试执行未经授权的访问，安全中间件会使用 `returnUrl` 集将用户重定向至登录页。

## <a name="the-form-action-tag-helper"></a>窗体操作标记帮助程序

窗体操作标记帮助程序在生成的 `<button ...>` 或 `<input type="image" ...>` 标记上生成 `formaction` 属性。 `formaction` 属性控制窗体在何处提交数据。 它绑定到 `image` 类型的 [\<input>](https://www.w3.org/wiki/HTML/Elements/input) 元素以及 [\<button>](https://www.w3.org/wiki/HTML/Elements/button) 元素。 窗体操作标记帮助程序允许使用多个 [AnchorTagHelper](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper)`asp-` 属性来控制为相应元素生成的 `formaction` 链接。

用于控制 `formaction` 值的受支持的 [AnchorTagHelper](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper) 属性：

|特性|说明|
|---|---|
|[asp-controller](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper#asp-controller)|控制器的名称。|
|[asp-action](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper#asp-action)|操作方法的名称。|
|[asp-area](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper#asp-area)|区域名称。|
|[asp-page](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper#asp-page)|Razor Page 的名称。|
|[asp-page-handler](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper#asp-page-handler)|Razor Page 处理程序的名称。|
|[asp-route](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper#asp-route)|路由的名称。|
|[asp-route-{value}](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper#asp-route-value)|单个 URL 路由值。 例如 `asp-route-id="1234"`。|
|[asp-all-route-data](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper#asp-all-route-data)|所有路由值。|
|[asp-fragment](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper#asp-fragment)|URL 片段。|

### <a name="submit-to-controller-example"></a>提交到控制器示例

选中输入或按钮时，下面的标记将窗体提交到 `HomeController` 的 `Index` 操作：

```cshtml
<form method="post">
    <button asp-controller="Home" asp-action="Index">Click Me</button>
    <input type="image" src="..." alt="Or Click Me" asp-controller="Home" 
                                asp-action="Index">
</form>
```

以前的标记将生成以下 HTML：

```html
<form method="post">
    <button formaction="/Home">Click Me</button>
    <input type="image" src="..." alt="Or Click Me" formaction="/Home">
</form>
```

### <a name="submit-to-page-example"></a>提交到页示例

以下标记将窗体提交到 `About` Razor Page：

```cshtml
<form method="post">
    <button asp-page="About">Click Me</button>
    <input type="image" src="..." alt="Or Click Me" asp-page="About">
</form>
```

以前的标记将生成以下 HTML：

```html
<form method="post">
    <button formaction="/About">Click Me</button>
    <input type="image" src="..." alt="Or Click Me" formaction="/About">
</form>
```

### <a name="submit-to-route-example"></a>提交到路由示例

请考虑使用 `/Home/Test` 终结点：

```csharp
public class HomeController : Controller
{
    [Route("/Home/Test", Name = "Custom")]
    public string Test()
    {
        return "This is the test page";
    }
}
```

以下标记将窗体提交到 `/Home/Test` 终结点。

```cshtml
<form method="post">
    <button asp-route="Custom">Click Me</button>
    <input type="image" src="..." alt="Or Click Me" asp-route="Custom">
</form>
```

以前的标记将生成以下 HTML：

```html
<form method="post">
    <button formaction="/Home/Test">Click Me</button>
    <input type="image" src="..." alt="Or Click Me" formaction="/Home/Test">
</form>
```

## <a name="the-input-tag-helper"></a>输入标记帮助程序

输入标记帮助程序将 HTML [\<input>](https://www.w3.org/wiki/HTML/Elements/input) 元素绑定到 Razor 视图中的模型表达式。

语法：

```HTML
<input asp-for="<Expression Name>">
```

输入标记帮助程序：

* 为 `asp-for` 属性中指定的表达式名称生成 `id` 和 `name` HTML 属性。 `asp-for="Property1.Property2"` 与 `m => m.Property1.Property2` 相等。 表达式的名称用于 `asp-for` 属性值。 有关其他信息，请参阅[表达式名称](#expression-names)部分。

* 根据模型类型和应用于模型属性的[数据注释](/dotnet/api/microsoft.aspnetcore.mvc.dataannotations.iattributeadapter)特性设置 HTML `type` 特性值

* 如果已经指定，不会覆盖 HTML `type` 属性值

* 通过应用于模型属性的[数据注释](/dotnet/api/microsoft.aspnetcore.mvc.dataannotations.iattributeadapter)特性生成 [HTML5](https://developer.mozilla.org/docs/Web/Guide/HTML/HTML5) 验证特性

* 具有与 `Html.TextBoxFor` 和 `Html.EditorFor` 重叠的 HTML 帮助程序功能。 有关详细信息，请参阅**输入标记帮助程序的 HTML 帮助程序替代项**部分。

* 提供强类型化。 如果属性的名称更改，但未更新标记帮助程序，则会收到类似如下内容的错误：

```HTML
An error occurred during the compilation of a resource required to process
this request. Please review the following specific error details and modify
your source code appropriately.

Type expected
 'RegisterViewModel' does not contain a definition for 'Email' and no
 extension method 'Email' accepting a first argument of type 'RegisterViewModel'
 could be found (are you missing a using directive or an assembly reference?)
```

`Input` 标记帮助程序根据 .NET 类型设置 HTML `type` 属性。 下表列出一些常见的 .NET 类型和生成的 HTML 类型（并未列出每个 .NET 类型）。

|.NET 类型|输入类型|
|---|---|
|Bool|type="checkbox"|
|String|type="text"|
|DateTime|type=["datetime-local"](https://developer.mozilla.org/docs/Web/HTML/Element/input/datetime-local)|
|Byte|type="number"|
|Int|type="number"|
|Single、Double|type="number"|

下表显示输入标记帮助程序会映射到特定输入类型的一些常见[数据注释](/dotnet/api/microsoft.aspnetcore.mvc.dataannotations.iattributeadapter)属性（并未列出每个验证属性）：

|特性|输入类型|
|---|---|
|[EmailAddress]|type="email"|
|[Url]|type="url"|
|[HiddenInput]|type="hidden"|
|[Phone]|type="tel"|
|[DataType(DataType.Password)]|type="password"|
|[DataType(DataType.Date)]|type="date"|
|[DataType(DataType.Time)]|type="time"|

示例:

[!code-csharp[](working-with-forms/sample/final/ViewModels/RegisterViewModel.cs)]

[!code-HTML[](working-with-forms/sample/final/Views/Demo/RegisterInput.cshtml)]

上述代码生成以下 HTML：

```HTML
  <form method="post" action="/Demo/RegisterInput">
      Email:
      <input type="email" data-val="true"
             data-val-email="The Email Address field is not a valid email address."
             data-val-required="The Email Address field is required."
             id="Email" name="Email" value=""><br>
      Password:
      <input type="password" data-val="true"
             data-val-required="The Password field is required."
             id="Password" name="Password"><br>
      <button type="submit">Register</button>
      <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>">
   </form>
```

应用于 `Email` 和 `Password` 属性的数据注释在模型中生成元数据。 输入标记帮助程序使用模型元数据并生成 [HTML5](https://developer.mozilla.org/docs/Web/Guide/HTML/HTML5) `data-val-*` 属性（请参阅[模型验证](../models/validation.md)）。 这些属性描述要附加到输入字段的验证程序。 这样可以提供非介入式 HTML5 和 [jQuery](https://jquery.com/) 验证。 非介入式属性具有格式 `data-val-rule="Error Message"`，其中规则是验证规则的名称（例如 `data-val-required`、`data-val-email`、`data-val-maxlength` 等）。如果在属性中提供错误消息，则该错误消息会作为 `data-val-rule` 属性的值显示。 还有表单 `data-val-ruleName-argumentName="argumentValue"` 的属性，这些属性提供有关规则的其他详细信息，例如，`data-val-maxlength-max="1024"`。

### <a name="html-helper-alternatives-to-input-tag-helper"></a>输入标记帮助程序的 HTML 帮助程序替代项

`Html.TextBox`、`Html.TextBoxFor`、`Html.Editor` 和 `Html.EditorFor` 与输入标记帮助程序的功能存在重叠。 输入标记帮助程序会自动设置 `type` 属性；而 `Html.TextBox` 和 `Html.TextBoxFor` 不会。 `Html.Editor` 和 `Html.EditorFor` 处理集合、复杂对象和模板；而输入标记帮助程序不会。 输入标记帮助程序、`Html.EditorFor` 和 `Html.TextBoxFor` 是强类型（使用 lambda 表达式）；而 `Html.TextBox` 和 `Html.Editor` 不是（使用表达式名称）。

### <a name="htmlattributes"></a>HtmlAttributes

`@Html.Editor()` 和 `@Html.EditorFor()` 在执行其默认模板时使用名为 `htmlAttributes` 的特殊 `ViewDataDictionary` 条目。 此行为可选择使用 `additionalViewData` 参数增强。 键“htmlAttributes”区分大小写。 键“htmlAttributes”的处理方式与传递到输入帮助程序的 `htmlAttributes` 对象（例如 `@Html.TextBox()`）的处理方式类似。

```HTML
@Html.EditorFor(model => model.YourProperty, 
  new { htmlAttributes = new { @class="myCssClass", style="Width:100px" } })
```

### <a name="expression-names"></a>表达式名称

`asp-for` 属性值是 `ModelExpression`，并且是 lambda 表达式的右侧。 因此，`asp-for="Property1"` 在生成的代码中变成 `m => m.Property1`，这也是无需使用 `Model` 前缀的原因。 “\@”字符可用作内联表达式的开头，并移到 `m.` 前面：

```HTML
@{
       var joe = "Joe";
   }
   <input asp-for="@joe">
```

生成以下 HTML：

```HTML
<input type="text" id="joe" name="joe" value="Joe">
```

使用集合属性时，`asp-for="CollectionProperty[23].Member"` 在 `i` 具有值 `23` 时生成与 `asp-for="CollectionProperty[i].Member"` 相同的名称。

在 ASP.NET Core MVC 计算 `ModelExpression` 的值时，它会检查多个源，包括 `ModelState`。 以 `<input type="text" asp-for="@Name">` 为例。 计算出的 `value` 属性是第一个非 null 值，属于：

* 带有“Name”键的 `ModelState` 条目。
* `Model.Name` 表达式的结果。

### <a name="navigating-child-properties"></a>导航子属性

还可使用视图模型的属性路径导航到子属性。 设想一个包含子 `Address` 属性的更复杂的模型类。

[!code-csharp[](../../mvc/views/working-with-forms/sample/final/ViewModels/AddressViewModel.cs?highlight=1,2,3,4&range=5-8)]

[!code-csharp[](../../mvc/views/working-with-forms/sample/final/ViewModels/RegisterAddressViewModel.cs?highlight=8&range=5-13)]

在视图中，绑定到 `Address.AddressLine1`：

[!code-HTML[](../../mvc/views/working-with-forms/sample/final/Views/Demo/RegisterAddress.cshtml?highlight=6)]

为 `Address.AddressLine1` 生成以下 HTML：

```HTML
<input type="text" id="Address_AddressLine1" name="Address.AddressLine1" value="">
```

### <a name="expression-names-and-collections"></a>表达式名称和集合

包含 `Colors` 数组的模型示例：

[!code-csharp[](../../mvc/views/working-with-forms/sample/final/ViewModels/Person.cs?highlight=3&range=5-10)]

操作方法：

```csharp
public IActionResult Edit(int id, int colorIndex)
   {
       ViewData["Index"] = colorIndex;
       return View(GetPerson(id));
   }
```

以下 Razor 显示如何访问特定 `Color` 元素：

[!code-HTML[](working-with-forms/sample/final/Views/Demo/EditColor.cshtml)]

*Views/Shared/EditorTemplates/String.cshtml* 模板：

[!code-HTML[](working-with-forms/sample/final/Views/Shared/EditorTemplates/String.cshtml)]

使用 `List<T>` 的示例：

[!code-csharp[](working-with-forms/sample/final/ViewModels/ToDoItem.cs?range=3-8)]

以下 Razor 演示如何循环访问集合：

[!code-HTML[](working-with-forms/sample/final/Views/Demo/Edit.cshtml)]

*Views/Shared/EditorTemplates/ToDoItem.cshtml* 模板：

[!code-HTML[](working-with-forms/sample/final/Views/Shared/EditorTemplates/ToDoItem.cshtml)]

应尽量在 `asp-for` 或 `Html.DisplayFor` 等效上下文中使用值 `foreach`。 一般情况下，`for` 优于 `foreach`（如果情况允许使用的话），因为它不需要分配枚举器，但在 LINQ 表达式中评估索引器的成本高昂，应最大限度地减少使用它。

&nbsp;

>[!NOTE]
>上述带有注释的示例代码演示如何将 lambda 表达式替换为 `@` 运算符来访问列表中的每个 `ToDoItem`。

## <a name="the-textarea-tag-helper"></a>文本区标记帮助程序

`Textarea Tag Helper` 标记帮助程序类似于输入标记帮助程序。

* 通过模型为 [\<textarea>](https://www.w3.org/wiki/HTML/Elements/textarea) 元素生成 `id` 和 `name` 属性以及数据验证属性。

* 提供强类型化。

* HTML 帮助程序替代项：`Html.TextAreaFor`

示例:

[!code-csharp[](working-with-forms/sample/final/ViewModels/DescriptionViewModel.cs)]

[!code-HTML[](../../mvc/views/working-with-forms/sample/final/Views/Demo/RegisterTextArea.cshtml?highlight=4)]

生成以下 HTML：

```HTML
<form method="post" action="/Demo/RegisterTextArea">
  <textarea data-val="true"
   data-val-maxlength="The field Description must be a string or array type with a maximum length of &#x27;1024&#x27;."
   data-val-maxlength-max="1024"
   data-val-minlength="The field Description must be a string or array type with a minimum length of &#x27;5&#x27;."
   data-val-minlength-min="5"
   id="Description" name="Description">
  </textarea>
  <button type="submit">Test</button>
  <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>">
</form>
```

## <a name="the-label-tag-helper"></a>标签标记帮助程序

* 在 [\<label>](https://www.w3.org/wiki/HTML/Elements/label) 元素中为表达式名称生成标签描述和 `for` 属性

* HTML 帮助程序替代项：`Html.LabelFor`。

`Label Tag Helper` 通过纯 HTML 标签元素提供如下优势：

* 可自动从 `Display` 属性中获取描述性标签值。 预期的显示名称可能会随时间变化，`Display` 属性和标签标记帮助程序的组合会在其被使用的所有位置应用 `Display`。

* 源代码中的标记更少

* 模型属性的强类型化。

示例:

[!code-csharp[](working-with-forms/sample/final/ViewModels/SimpleViewModel.cs)]

[!code-HTML[](../../mvc/views/working-with-forms/sample/final/Views/Demo/RegisterLabel.cshtml?highlight=4)]

为 `<label>` 元素生成以下 HTML：

```HTML
<label for="Email">Email Address</label>
```

标签标记帮助程序生成“Email”的 `for` 属性值，即与 `<input>` 元素关联的 ID。 标记帮助程序生成一致的 `id` 和 `for` 元素，方便将其正确关联。 本示例中的描述来自 `Display` 属性。 如果模型不包含 `Display` 特性，描述将为表达式的属性名称。

## <a name="the-validation-tag-helpers"></a>验证标记帮助程序

有两个验证标记帮助程序。 `Validation Message Tag Helper`（为模型中的单个属性显示验证消息）和 `Validation Summary Tag Helper`（显示验证错误的摘要）。 `Input Tag Helper` 根据模型类的数据注释属性将 HTML5 客户端验证属性添加到输入元素中。 同时在服务器上执行验证。 验证标记帮助程序会在发生验证错误时显示这些错误消息。

### <a name="the-validation-message-tag-helper"></a>验证消息标记帮助程序

* 将 [HTML5](https://developer.mozilla.org/docs/Web/Guide/HTML/HTML5) `data-valmsg-for="property"` 属性添加到 [span](https://developer.mozilla.org/docs/Web/HTML/Element/span) 元素中，该元素会附加指定模型属性的输入字段中的验证错误消息。 [jQuery](https://jquery.com/) 会在发生客户端验证错误时在 `<span>` 元素中显示错误消息。

* 还会在服务器上执行验证。 客户端可能已禁用 JavaScript，一些验证仅可在服务器端执行。

* HTML 帮助程序替代项：`Html.ValidationMessageFor`

`Validation Message Tag Helper` 与 HTML [span](https://developer.mozilla.org/docs/Web/HTML/Element/span) 元素中的 `asp-validation-for` 属性配合使用。

```HTML
<span asp-validation-for="Email"></span>
```

验证消息标记帮助程序会生成以下 HTML：

```HTML
<span class="field-validation-valid"
  data-valmsg-for="Email"
  data-valmsg-replace="true"></span>
```

对于同一属性，通常在 `Input` 标记帮助程序后使用 `Validation Message Tag Helper`。 这样做可在导致错误的输入附近显示所有验证错误消息。

> [!NOTE]
> 必须拥有包含正确的 JavaScript 和 [jQuery](https://jquery.com/) 脚本引用的视图才能执行客户端验证。 有关详细信息，请参阅[模型验证](../models/validation.md)。

发生服务器端验证错误时（例如，禁用自定义服务器端验证或客户端验证时），MVC 会将该错误消息作为 `<span>` 元素的主体。

```HTML
<span class="field-validation-error" data-valmsg-for="Email"
            data-valmsg-replace="true">
   The Email Address field is required.
</span>
```

### <a name="the-validation-summary-tag-helper"></a>验证摘要标记帮助程序

* 针对具有 `asp-validation-summary` 属性的 `<div>` 元素

* HTML 帮助程序替代项：`@Html.ValidationSummary`

`Validation Summary Tag Helper` 用于显示验证消息的摘要。 `asp-validation-summary` 属性值可以是以下任意值：

|asp-validation-summary|显示的验证消息|
|--- |--- |
|ValidationSummary.All|属性和模型级别|
|ValidationSummary.ModelOnly|模型|
|ValidationSummary.None|无|

### <a name="sample"></a>示例

在以下示例中，数据模型使用 `DataAnnotation` 属性修饰，在 `<input>` 元素中生成验证错误消息。  验证标记帮助程序会在发生验证错误时显示错误消息：

[!code-csharp[](working-with-forms/sample/final/ViewModels/RegisterViewModel.cs)]

[!code-HTML[](../../mvc/views/working-with-forms/sample/final/Views/Demo/RegisterValidation.cshtml?highlight=4,6,8&range=1-10)]

生成的 HTML（如果模型有效）：

```HTML
<form action="/DemoReg/Register" method="post">
  <div class="validation-summary-valid" data-valmsg-summary="true">
  <ul><li style="display:none"></li></ul></div>
  Email:  <input name="Email" id="Email" type="email" value=""
   data-val-required="The Email field is required."
   data-val-email="The Email field is not a valid email address."
   data-val="true"><br>
  <span class="field-validation-valid" data-valmsg-replace="true"
   data-valmsg-for="Email"></span><br>
  Password: <input name="Password" id="Password" type="password"
   data-val-required="The Password field is required." data-val="true"><br>
  <span class="field-validation-valid" data-valmsg-replace="true"
   data-valmsg-for="Password"></span><br>
  <button type="submit">Register</button>
  <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>">
</form>
```

## <a name="the-select-tag-helper"></a>选择标记帮助程序

* 为模型属性生成 [select](https://www.w3.org/wiki/HTML/Elements/select) 元素和关联的 [option](https://www.w3.org/wiki/HTML/Elements/option) 元素。

* 具有 HTML 帮助程序替代项 `Html.DropDownListFor` 和 `Html.ListBoxFor`

`Select Tag Helper` `asp-for` 为 [select](https://www.w3.org/wiki/HTML/Elements/select) 元素指定模型属性名称，`asp-items` 指定 [option](https://www.w3.org/wiki/HTML/Elements/option) 元素。  例如:

[!code-HTML[](working-with-forms/sample/final/Views/Home/Index.cshtml?range=4)]

示例:

[!code-csharp[](working-with-forms/sample/final/ViewModels/CountryViewModel.cs)]

`Index` 方法初始化 `CountryViewModel`，设置选定的国家/地区并将其传递到 `Index` 视图。

[!code-csharp[](working-with-forms/sample/final/Controllers/HomeController.cs?range=8-13)]

HTTP POST `Index` 方法显示选定内容：

[!code-csharp[](working-with-forms/sample/final/Controllers/HomeController.cs?range=15-27)]

`Index` 视图：

[!code-cshtml[](working-with-forms/sample/final/Views/Home/Index.cshtml?highlight=4)]

生成以下 HTML（选择“CA”时）：

```html
<form method="post" action="/">
     <select id="Country" name="Country">
       <option value="MX">Mexico</option>
       <option selected="selected" value="CA">Canada</option>
       <option value="US">USA</option>
     </select>
       <br /><button type="submit">Register</button>
     <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>">
   </form>
```

> [!NOTE]
> 不建议将 `ViewBag` 或 `ViewData` 与选择标记帮助程序配合使用。 视图模型在提供 MVC 元数据方面更可靠且通常更不容易出现问题。

`asp-for` 属性值是特殊情况，它不要求提供 `Model` 前缀，但其他标记帮助程序属性需要该前缀（例如 `asp-items`）

[!code-HTML[](working-with-forms/sample/final/Views/Home/Index.cshtml?range=4)]

### <a name="enum-binding"></a>枚举绑定

通常可方便地将 `<select>` 与 `enum` 属性配合使用并通过 `enum` 值生成 `SelectListItem` 元素。

示例:

[!code-csharp[](working-with-forms/sample/final/ViewModels/CountryEnumViewModel.cs?range=3-7)]

[!code-csharp[](working-with-forms/sample/final/ViewModels/CountryEnum.cs)]

`GetEnumSelectList` 方法为枚举生成 `SelectList` 对象。

[!code-HTML[](../../mvc/views/working-with-forms/sample/final/Views/Home/IndexEnum.cshtml?highlight=5)]

可使用 `Display` 属性修饰枚举器列表，以获取更丰富的 UI：

[!code-csharp[](working-with-forms/sample/final/ViewModels/CountryEnum.cs?highlight=5,7)]

生成以下 HTML：

```HTML
  <form method="post" action="/Home/IndexEnum">
         <select data-val="true" data-val-required="The EnumCountry field is required."
                 id="EnumCountry" name="EnumCountry">
             <option value="0">United Mexican States</option>
             <option value="1">United States of America</option>
             <option value="2">Canada</option>
             <option value="3">France</option>
             <option value="4">Germany</option>
             <option selected="selected" value="5">Spain</option>
         </select>
         <br /><button type="submit">Register</button>
         <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>">
    </form>
```

### <a name="option-group"></a>选项组

如果视图模型包含一个或多个 `SelectListGroup` 对象，则会生成 HTML [\<optgroup>](https://www.w3.org/wiki/HTML/Elements/optgroup) 元素。

`CountryViewModelGroup` 将 `SelectListItem` 元素分组为“North America”组和“Europe”组：

[!code-csharp[](../../mvc/views/working-with-forms/sample/final/ViewModels/CountryViewModelGroup.cs?highlight=5,6,14,20,26,32,38,44&range=6-56)]

两个组如下所示：

![选项组示例](working-with-forms/_static/grp.png)

生成的 HTML：

```HTML
 <form method="post" action="/Home/IndexGroup">
      <select id="Country" name="Country">
          <optgroup label="North America">
              <option value="MEX">Mexico</option>
              <option value="CAN">Canada</option>
              <option value="US">USA</option>
          </optgroup>
          <optgroup label="Europe">
              <option value="FR">France</option>
              <option value="ES">Spain</option>
              <option value="DE">Germany</option>
          </optgroup>
      </select>
      <br /><button type="submit">Register</button>
      <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>">
 </form>
```

### <a name="multiple-select"></a>多重选择

如果 `asp-for` 属性中指定的属性为 `IEnumerable`，选择标记帮助程序会自动生成 [multiple = "multiple"](https://w3c.github.io/html-reference/select.html) 属性。 例如，如果给定以下模型：

[!code-csharp[](../../mvc/views/working-with-forms/sample/final/ViewModels/CountryViewModelIEnumerable.cs?highlight=6)]

及以下视图：

[!code-HTML[](../../mvc/views/working-with-forms/sample/final/Views/Home/IndexMultiSelect.cshtml?highlight=4)]

则会生成以下 HTML：

```HTML
<form method="post" action="/Home/IndexMultiSelect">
    <select id="CountryCodes"
    multiple="multiple"
    name="CountryCodes"><option value="MX">Mexico</option>
<option value="CA">Canada</option>
<option value="US">USA</option>
<option value="FR">France</option>
<option value="ES">Spain</option>
<option value="DE">Germany</option>
</select>
    <br /><button type="submit">Register</button>
  <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>">
</form>
```

### <a name="no-selection"></a>无选定内容

如果发现自己在多个页面中使用“未指定”选项，可创建模板用于消除重复的 HTML：

[!code-HTML[](../../mvc/views/working-with-forms/sample/final/Views/Home/IndexEmptyTemplate.cshtml?highlight=5)]

*Views/Shared/EditorTemplates/CountryViewModel.cshtml* 模板：

[!code-HTML[](working-with-forms/sample/final/Views/Shared/EditorTemplates/CountryViewModel.cshtml)]

添加 HTML [\<option>](https://www.w3.org/wiki/HTML/Elements/option) 元素并不局限于*无选定内容*用例。 例如，以下视图和操作方法会生成与上述代码类似的 HTML：

[!code-csharp[](working-with-forms/sample/final/Controllers/HomeController.cs?range=114-119)]

[!code-HTML[](working-with-forms/sample/final/Views/Home/IndexOption.cshtml)]

根据当前的 `Country` 值选择正确的 `<option>` 元素（包含 `selected="selected"` 属性）。

```HTML
 <form method="post" action="/Home/IndexEmpty">
      <select id="Country" name="Country">
          <option value="">&lt;none&gt;</option>
          <option value="MX">Mexico</option>
          <option value="CA" selected="selected">Canada</option>
          <option value="US">USA</option>
      </select>
      <br /><button type="submit">Register</button>
   <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>">
 </form>
 ```

## <a name="additional-resources"></a>其他资源

* <xref:mvc/views/tag-helpers/intro>
* [HTML Form 元素](https://www.w3.org/TR/html401/interact/forms.html)
* [请求验证令牌](/aspnet/mvc/overview/security/xsrfcsrf-prevention-in-aspnet-mvc-and-web-pages)
* <xref:mvc/models/model-binding>
* <xref:mvc/models/validation>
* [IAttributeAdapter 接口](/dotnet/api/Microsoft.AspNetCore.Mvc.DataAnnotations.IAttributeAdapter)
* [本文档的代码片段](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/views/working-with-forms/sample/final)
