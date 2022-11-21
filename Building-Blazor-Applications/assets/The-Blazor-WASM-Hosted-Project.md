# A Short Tour of the Blazor WASM hosted Project

The Blazor Web Assembly Core Hosted project is a little confusing for newcomers to Blazor.  This note hopefully sheds some light on it.

When you add a Web Assembly hosted project to a solution you get a directory project with three sub-directories.  Each contains a Visual Studio project.  These are:

1. Client
2. Shared
3. Server

## Client 

This is your Web Assembly project.  It's where you put all your Blazor code.

The project file looks like this:

```xml
<Project Sdk="Microsoft.NET.Sdk.BlazorWebAssembly">

  <PropertyGroup>
    <TargetFramework>net6.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.Components.WebAssembly" Version="6.0.0" />
    <PackageReference Include="Microsoft.AspNetCore.Components.WebAssembly.DevServer" Version="6.0.0" PrivateAssets="all" />
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="..\Shared\BlazorApp1.Shared.csproj" />
  </ItemGroup>

</Project>
```

Note the Sdk is set to `Microsoft.NET.Sdk.BlazorWebAssembly`, and it has a dependancy on *Shared*.

When you compile this project it builds the code and resource files you need to deploy the project to a web browser and run in WASM mode.

Note that the project has a `wwwroot` directory that contains the startup page `index.html` and the Css resources.

`index.html` looks like this:

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
    <title>BlazorApp1</title>
    <base href="/" />
    <link href="css/bootstrap/bootstrap.min.css" rel="stylesheet" />
    <link href="css/app.css" rel="stylesheet" />
    <link href="BlazorApp1.Client.styles.css" rel="stylesheet" />
</head>

<body>
    <div id="app">Loading...</div>

    <div id="blazor-error-ui">
        An unhandled error has occurred.
        <a href="" class="reload">Reload</a>
        <a class="dismiss">🗙</a>
    </div>
    <script src="_framework/blazor.webassembly.js"></script>
</body>

</html>
```
More shortly on how this fires up the WASM SPA.

## Shared

Shared is a library project.  It contains code shared between the Client and Server projects: the `WeatherForecast` data class.  It's used in the application and by the API controller.

## Server

Server is a standard AspNetCore web server project: it's not Blazor.  It has two purposes:

1. Act as a host for serving the WASM project files.
2. Provide the controllers for API calls from the Client project.

The project looks like this:

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net6.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.Components.WebAssembly.Server" Version="6.0.0" />
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="..\Client\BlazorApp1.Client.csproj" />
    <ProjectReference Include="..\Shared\BlazorApp1.Shared.csproj" />
  </ItemGroup>
</Project>
```

`Program` looks like this:


```csharp
using Microsoft.AspNetCore.ResponseCompression;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.

builder.Services.AddControllersWithViews();
builder.Services.AddRazorPages();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseWebAssemblyDebugging();
}
else
{
    app.UseExceptionHandler("/Error");
    // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
    app.UseHsts();
}

app.UseHttpsRedirection();

app.UseBlazorFrameworkFiles();
app.UseStaticFiles();

app.UseRouting();

app.MapRazorPages();
app.MapControllers();
app.MapFallbackToFile("index.html");

app.Run();
```

`app.UseBlazorFrameworkFiles()` adds the middleware to map and process the requests for the WASM files from any `BlazorWebAssembly` Sdk dependant project.  The WASM files are mapped to the `_framework` virtual directory and any `wwwroot` files in the `BlazorWebAssembly` project are mapping into the server's root folder.  In this instance the *Server* project doesn't have it's own `wwwroot` folder.

`app.MapControllers()` maps any controllers in the *Server* project.  In this case the `WeatherForecastController`.

`app.MapFallbackToFile("index.html")` points to the `BlazorWebAssembly` project `index.html` as the default site page.

## Running the project

Set the startup project as the *Server* project.

The solution starts, opens the site in the browser and hits the fallback file - `index.html` in the *Client* project.  This downloads and runs `_framework/blazor.webassembly.js`.  The `UseBlazorFrameworkFiles` middleware maps this correctly to the Client project.  The image below shows the *bin* folder on the Client project, which should help to turn some lights on!

![Web Assembly Bin](https://shauncurtis.github.io/posts/assets/WebAssembly/WebAssembly-bin.png)

`blazor.webassembly.js` gets `./blazor.boot.json`, the manifest file for booting the WASM codebase.  Once started the SPA replaces` <div id="app">Loading...</div>` with the `App.razor` component and we have a WASM Single Page Application in the browser.  The only comms. with the server are now API calls to get server based database information.

## Moving On

While the Blazor out-of-the-box templates are a good starting point for an initial play, you should not base your first project on them.

It's important to consider design before taking that step.  There are many resources on the Internet.  There's my quick primary on Clean Design which includes a structured Blazor project template.

[Clean Design in Blazor](https://shauncurtis.github.io/Design/Clean-Design-In-Blazor.html).

