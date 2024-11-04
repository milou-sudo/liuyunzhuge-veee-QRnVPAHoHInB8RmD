[合集 \- .NET云原生应用实践(5\)](https://github.com)[1\..NET云原生应用实践（二）：Sticker微服务RESTful API的实现10\-13](https://github.com/daxnet/p/18456878)[2\..NET云原生应用实践（一）：从搭建项目框架结构开始10\-09](https://github.com/daxnet/p/18172088)[3\..NET云原生应用实践（三）：连接到PostgreSQL数据库10\-22](https://github.com/daxnet/p/18470813)[4\..NET云原生应用实践（四）：基于Keycloak的认证与授权10\-28](https://github.com/daxnet/p/18500344)5\..NET云原生应用实践（五）：使用Blazor WebAssembly实现前端页面11\-03收起
# 本章目标


* 使用Blazor WebAssembly实现管理“贴纸”页面
* 集成认证与授权机制



> 如果你对Blazor WebAssembly的使用不感兴趣，可以跳过本章的阅读。你也可以使用自己熟悉的前端技术完成案例的界面部分，之前我们开发的后端API比较简单，所以自己实现一套前端界面并不会是一个困难的事情。


完成本章内容后，我们会得到下面的效果（点击查看大图），是不是跟第一章中所画的概念图已经很接近了？


![](https://img2024.cnblogs.com/blog/119825/202411/119825-20241102191307919-807286168.png)


# 我们到哪里了？


在进一步介绍后续内容之前，先看看目前实现了哪些内容。回顾之前的一张架构简图（其实也不算是架构图），彩色部分是目前我们已经实现的部分，虽然目前有些地方还并不完善，只是在开发环境能够正常运行起来，并且我们开发的RESTful API都还没有容器化。


![](https://img2024.cnblogs.com/blog/119825/202410/119825-20241031211238602-2107027013.png)


本章会完成“Sticker前端应用”这个部分，在完成这部分内容后，我们就可以在开发环境中调试运行整个应用程序了，由于还没有引入基于nginx的API网关，所以，整个系统的结构跟上图相比还是会有些差异。


# Blazor WebAssembly是什么？


如果问ChatGPT，它的回答是这样的：Blazor WebAssembly是一个基于WebAssembly的现代Web应用程序框架，由微软开发。它允许开发人员使用C\#和.NET技术构建客户端Web应用程序，而无需使用JavaScript。Blazor WebAssembly利用WebAssembly的性能优势，将C\#代码编译为WebAssembly字节码，从而在浏览器中运行高性能的客户端应用程序。开发人员可以使用Blazor组件模型构建交互式和动态的用户界面，同时利用.NET的强大功能和生态系统。Blazor WebAssembly还支持与服务器端Blazor应用程序的通信，以及与现有JavaScript库的集成，为开发人员提供了灵活和强大的工具来构建现代的Web应用程序。


Blazor应用程序基本上可以分为两种：


* Blazor服务端应用：它基于ASP.NET Core基础设施实现服务端Hosting，并通过一种通信方式（比如SignalR）实现用户交互
* Blazor WebAssembly：它是在客户端浏览器中运行的Web应用程序，它将C\#代码编译为WebAssembly字节码，直接在浏览器中执行。Blazor WebAssembly应用程序完全在客户端执行，可以实现更快的加载速度和更高的性能，适用于需要在客户端独立运行的应用程序，以及对实时性要求较高的应用



> 从.NET 8开始，Visual Studio引入新的Blazor应用程序构建模板：Blazor Web App，它整合了Blazor服务端和Blazor WebAssembly的优势，并且利用了.NET 8中新引入的Blazor相关功能，比如静态服务端渲染（static SSR）、流式渲染（Streaming Rendering）等。原有的Blazor Server App和Blazor WebAssembly Standalone App在.NET 8 中仍然支持，只不过可以考虑将这些类型的应用迁移到Blazor Web App上。详见：[https://learn.microsoft.com/en\-us/aspnet/core/release\-notes/aspnetcore\-8\.0?view\=aspnetcore\-7\.0\#new\-blazor\-web\-app\-template](https://github.com)


本系列文章案例代码选用Blazor WebAssembly项目模板作为基础进行开发。


# 为什么选择Blazor WebAssembly？


现在前端技术非常成熟，体系也很庞大，为何抛弃React、Angular、Vue这些前端框架不选，偏偏选择Blazor WebAssembly呢？我想大概几个方面吧：


* 我想尝试一下微软原生WebAssembly的开发框架，看看开发者体验如何
* 我打算仍然选择微软技术来展示案例（之前有读者对我使用Java系的Keycloak有质疑，其实Keycloak在整个案例架构中只是一个IdP，它跟PostgreSQL、Redis这些组件等价，这么说能理解吧？）
* 我对C\#技术栈更为熟悉，功能开发和问题调查都会更加方便快捷，而且不容易出错。在微服务的开发模式中，技术选择其实跟团队成员的偏好也有一定的关系，在能够满足各种功能性和非功能性需求的前提下，团队当然希望采用更为熟知的技术来完成研发。聊到我的前端技术，我个人对Angular比较熟悉，因为之前做过Angular的前端项目，React和Vue一直都没有机会实践（或许我也不应该再“卷”下去了）


除了微软的Microsoft Learn和在线教育平台Edink之外，还是有[不少站点是基于Blazor技术构建](https://github.com):[楚门加速器p](https://tianchuang88.com)的，微软官方也给了[几个客户案例](https://github.com)，它们大多数都是US的公司，国内很少使用。


从上面三点可以看到，我在这个案例中选择Blazor WebAssembly，主观因素更多一些，在实际项目中，大概率大家也不会选择Blazor WebAssembly来构建自己的前端应用，原因也会是多方面的。由于本系列文章所介绍的案例比较简单，前端部分暂时也不会有特别高的要求，所以我就基于自己的主观需求，选择了Blazor WebAssembly。读者完全可以基于本案例的服务端代码，使用自己熟悉的前端技术来重制“贴纸墙”的前端部分。


# 构建Stickers.Web应用


首先就是创建一个Blazor WebAssembly的应用，并启用认证机制，因为后面需要集成认证和授权流程。此外，我还在项目中使用了[Blazor Bootstrap](https://github.com)组件库，这个组件库对主要的Bootstrap组件进行了封装，并让其在Blazor应用中完美运行。使用Blazor Bootstrap需要有一些配置工作，这里不多介绍了，官方文档有[Get Started](https://github.com)操作流程。


Blazor WebAssembly的开发过程这里也不多做介绍了，请直接参考本文的源代码。这里主要介绍三个话题：自定义组件、使用HttpClient访问后端服务，以及认证与授权。


## 自定义组件


通常我们会把一些能够重复使用的前端代码封装成一个组件，并通过参数来接受数据并定制业务逻辑，执行过程中又通过事件与其它组件交互。比如，一个分页功能就可以封装成一个组件，它可以通过参数来设置分页按钮的样式以及一次展现多少个分页按钮，当用户点击某个页码时，它又以事件的方式通知相关的其它组件（比如父页面）被点击的页码数，以便触发页面更新等后续操作。


下面的代码是案例中的“编辑贴纸”的组件，这个组件有一个参数：`StickerEditModel`，用来指定用户操作行为类型（新建/编辑）以及将要新建/被编辑贴纸的数据模型，此外还包含两个事件：`OnCloseClickCallback`和`OnSaveClickCallback`，当组件界面上的“关闭”和“保存”按钮被点击时，会触发这两个事件。`StickerEditModel`的定义如下：



```


|  | public enum EditMode |
| --- | --- |
|  | { |
|  | Create, |
|  | Edit |
|  | } |
|  |  |
|  | public class StickerEditModel |
|  | { |
|  | public string? Content { get; set; } |
|  | public int Id { get; set; } |
|  | public string? Title { get; set; } |
|  | public EditMode EditMode { get; set; } |
|  | } |


```

`StickerEditModel`看起来跟`Sticker`业务对象很像，但它只关注界面上所需的数据，所以，在`StickerEditModel`中，并没有`CreatedOn`、`ModifiedOn`这些属性，因为这些属性都是在创建或者修改贴纸时由系统自动生成的，新建/编辑贴纸的界面上并不需要这些信息。以下是“编辑贴纸”的组件`EditStickerComponent`的代码：



```


|  | @using Stickers.Web.ViewModels |
| --- | --- |
|  |  |
|  | @if (Model is not null) |
|  | { |
|  | <div class="mb-3"> |
|  | <input @ref="_txtTitleRef" type="text" class="form-control" placeholder="请输入贴纸标题" @bind-value="Model.Title"> |
|  | div> |
|  | <div class="mb-3"> |
|  | <InputTextArea class="form-control" placeholder="请输入贴纸内容" @bind-Value="Model.Content"/> |
|  | div> |
|  |  |
|  | <div class="d-grid gap-2 d-md-flex justify-content-md-end mt-2"> |
|  | <Button Color="ButtonColor.Secondary" @onclick="OnCloseClickCallback"> 取消 Button> |
|  | <Button Color="ButtonColor.Primary" @onclick="OnSaveClick"> 保存 Button> |
|  | div> |
|  | } |
|  |  |
|  | @code { |
|  | [Parameter] |
|  | public StickerEditModel? Model { get; set; } |
|  |  |
|  | [Parameter] |
|  | public EventCallback<MouseEventArgs> OnCloseClickCallback { get; set; } |
|  |  |
|  | [Parameter] |
|  | public EventCallback<StickerEditModel> OnSaveClickCallback { get; set; } |
|  |  |
|  | private ElementReference? _txtTitleRef; |
|  |  |
|  | protected override async Task OnAfterRenderAsync(bool firstRender) |
|  | { |
|  | if (_txtTitleRef.HasValue) |
|  | { |
|  | await _txtTitleRef.Value.FocusAsync(true); |
|  | } |
|  | } |
|  |  |
|  | private async Task OnSaveClick() |
|  | { |
|  | await InvokeAsync(() => OnSaveClickCallback.InvokeAsync(Model)); |
|  | } |
|  | } |


```

它提供了两个文本框，分别让用户输入贴纸的标题和内容，还有两个按钮，让用户保存所做的修改或者取消所做的操作。调用组件会生成一个StickerEditModel的实例，通过Model参数传入这个组件，然后以对话框的形式显示该组件以接收用户输入，当用户完成操作并点击保存或者取消按钮时，通过事件将用户的输入返回给调用方。


## 使用HttpClient访问后端服务


在Blazor WebAssembly中访问后端服务是非常方便的，只需在`Program.cs`中加入HttpClient的支持，比如：



```


|  | builder.Services.AddHttpClient( |
| --- | --- |
|  | "myHttpClient", |
|  | client => client.BaseAddress = new Uri("http://localhost:5000") |
|  | ); |


```

然后，在Razor页面或者组件中，通过注入`HttpClientFactory`，就可以使用注册的HttpClient了：



```


|  | @inject IHttpClientFactory HttpClientFactory |
| --- | --- |
|  |  |
|  | @code { |
|  | private override async Task OnInitializedAsync() |
|  | { |
|  | // ... |
|  | using var httpClient = HttpClientFactory.GetClient("myHttpClient"); |
|  | var responseMessage = await httpClient.GetAsync("api/any-api"); |
|  | // ... |
|  | } |
|  | } |


```

HttpClient在Blazor中的使用，跟ASP.NET Core中非常类似，可以直接阅读[官方文档](https://github.com)来了解详细内容，这里就不多做介绍了。


## 认证与授权


在`Stickers.Web`项目中需要调用后端的`Stickers.WebApi` RESTful API来实现其功能，而后端API是需要认证和授权的，所以，前端界面在HttpClient发送API调用请求时，就需要把access token带上，否则API调用是不会成功的。在Blazor WebAssembly中，要实现这个逻辑，就需要自定义`DelegatingHandler`，然后在HttpClient中使用这个自定义的Handler。


Blazor WebAssembly支持一种称之为`AuthorizationMessageHandler`的`DelegatingHandler`，它可以直接拿来使用，以便将access token附加到发出的HTTP请求上。只需要在添加HttpClient的时候，指定HttpMessageHandler即可：



```


|  | builder.Services.AddHttpClient( |
| --- | --- |
|  | "myHttpClient", |
|  | client => client.BaseAddress = new Uri("http://localhost:5000") |
|  | ).AddHttpMessageHandler(); |


```

认证用户可以登录站点，并不表示该用户可以访问所有的页面并进行所有的操作，比如前文中所创建的nobody用户，它只能被认证，却没有任何授权，所以，对于该用户而言，它是无法使用“贴纸”功能的。在这个用户登录之后，即便登录没有问题，使用该用户的access token去访问后端API服务仍然会得到`403 Forbidden`的错误，比如，在这个用户点击“我的贴纸墙”页面时，下面的代码就会抛出未授权异常：



```


|  | @code { |
| --- | --- |
|  | protected override async Task OnInitializedAsync() |
|  | { |
|  | CurrentPage = await ReadStickersAsync(); |
|  | // 此处由于异常未被处理，造成页面出错 |
|  | await base.OnInitializedAsync(); |
|  | } |
|  |  |
|  | private async Task<StickersPage?> ReadStickersAsync( |
|  | int pageNumber = 1, |
|  | int pageSize = DefaultPageSize) |
|  | { |
|  | using var httpClient = HttpClientFactory.CreateClient("stickersHttpClient"); |
|  | var httpResponseMessage = await httpClient |
|  | .GetAsync($"api/stickers?page={pageNumber}&size={pageSize}"); |
|  | httpResponseMessage.EnsureSuccessStatusCode(); // 此处抛出异常 |
|  | var responseJson = await httpResponseMessage.Content.ReadAsStringAsync(); |
|  | return JsonSerializer.Deserialize(responseJson); |
|  | } |
|  | } |


```

解决这个问题的思路有两种：


1. 由于WebAssembly是可以得到用户的access token的，所以也可以像之前Stickers API里设计的那样，获得用户的授权信息，然后根据用户的授权信息来设计前端的授权机制（Blazor WebAssembly默认基于角色授权，也可以自己开发自定义的Policy来实现更为灵活的授权方案），再根据这套机制和用户本身的授权信息以判定某个组件是否应该显示、是否可以被该用户使用
2. 简单粗暴，在调用API时，如果异常，则捕获异常并直接跳转到登录界面或者错误界面，提示用户没有权限


第一种方案其实更为合理，一方面如果用户本来就没有权限，那就可以直接把不可以访问的组件隐藏掉或者禁用，没必要等到用户点击的时候才报错；另一方面，设计一个前端授权机制也会使得组件和页面的访问控制变得更为灵活，如果设计合理，还可以跟Blazor WebAssembly的授权机制无缝整合，大大减少出错的几率。而第二种方案则相对简单一些，适用于像本文这样的demo场景（Blazor应用的授权设计不是本案例的重点）。


首先可以自定义一个`AuthorizationMessageHandler`，然后通过`AddHttpMessageHandler`方法，将这个Handler注册到HttpClient上：



```


|  | public class StickersMessageHandler : AuthorizationMessageHandler |
| --- | --- |
|  | { |
|  | private readonly NavigationManager _navigationManager; |
|  |  |
|  | public StickersMessageHandler(IAccessTokenProvider provider, NavigationManager navigation) : base(provider, |
|  | navigation) |
|  | { |
|  | _navigationManager = navigation; |
|  | } |
|  |  |
|  | protected override async Task SendAsync( |
|  | HttpRequestMessage request, |
|  | CancellationToken cancellationToken) |
|  | { |
|  | try |
|  | { |
|  | var responseMessage = await base.SendAsync(request, cancellationToken); |
|  | if (responseMessage.StatusCode == HttpStatusCode.Forbidden) |
|  | { |
|  | _navigationManager.NavigateTo("/forbidden"); |
|  | } |
|  | return responseMessage; |
|  | } |
|  | catch (AccessTokenNotAvailableException ex) |
|  | { |
|  | ex.Redirect(); |
|  | return new HttpResponseMessage(); |
|  | } |
|  | } |
|  | } |


```

这个类首先注入一个`NavigationManager`实例，然后在重载的`SendAsync`方法中，判断返回的状态码是否为`403 Forbidden`，如果是的话，就直接跳转到/forbidden页面就可以了。这里的代码虽然对状态码进行了判断，但是在调用端的`EnsureSuccessStatusCode`方法仍然会因为状态码不是2XX而抛出异常。这里只要稍微处理一下就可以了：



```


|  | protected override async Task OnInitializedAsync() |
| --- | --- |
|  | { |
|  | try |
|  | { |
|  | CurrentPage = await ReadStickersAsync(); |
|  | } |
|  | catch (HttpRequestException ex) when (ex.StatusCode is HttpStatusCode.Forbidden) |
|  | { |
|  | return; |
|  | } |
|  |  |
|  | await base.OnInitializedAsync(); |
|  | } |


```

# 总结


本文简单介绍了基于Blazor WebAssembly实现前端的几个主要方面，前端代码很多，一篇文章也无法全部介绍完整，有兴趣的读者请直接下载源码阅读。在运行本案例的过程中，你会发现，登录用户之前还能互相看到对方所创建的贴纸，这是一个bug，在下一讲中，我将通过引入多租户的初步设计，将这个bug修复掉。


# 源代码


本章源代码在chapter\_5这个分支中：[https://gitee.com/daxnet/stickers/tree/chapter\_5/](https://github.com)


下载源代码前，请先删除已有的`stickers-pgsql:dev`和`stickers-keycloak:dev`两个容器镜像，并删除`docker_stickers_postgres_data`数据卷。


下载源代码后，进入docker目录，然后编译并启动容器：



```


|  | $ docker compose -f docker-compose.dev.yaml build |
| --- | --- |
|  | $ docker compose -f docker-compose.dev.yaml up |


```

现在就可以直接用Visual Studio 2022或者JetBrains Rider打开stickers.sln解决方案文件，然后同时启动Stickers.WebApi和Stickers.Web两个项目进行调试运行了。


