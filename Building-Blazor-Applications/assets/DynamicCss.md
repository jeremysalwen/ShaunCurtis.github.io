Some code snippets to change out CSS dynamically in Blazor.

Set up the default link in the page as:

```html
    <link id="dynamicCssLink" rel="stylesheet" href="/css/site.css" />
```

In this case the `elementId` is *dynamicCssLink*. 

JS code to go into site.js.
```js
window.DynamicCss = {
   setCss: function (elementId, url) {
        var link = document.getElementById(elementId);
        if (link === undefined) {
            link = document.createElement(elementId);
            link.id = elementId;
            document.head.insertBefore(link, document.head.firstChild);
            link.type = 'text/css';
            link.rel = 'stylesheet';
        }
        link.href = url;
        return true;
    }
}
```

Interop libary code.

```csharp
public class InteropLibrary
{
    protected IJSRuntime JSRuntime { get; }

    public InteropLibrary(IJSRuntime jsRuntime)
        => JSRuntime = jsRuntime;

    public ValueTask<bool> SetDynamicCss(string elementId, string url)
        => JSRuntime.InvokeAsync<bool>("DynamicCss.setCss", elementId, url);
}
```
Register `InteropLibrary` as a scoped service.

```csharp
builder.Services.AddScoped<InteropLibrary>();
```
Add an alternative Css file - site-black.css

```css
@import url('open-iconic/font/css/open-iconic-bootstrap.min.css');

html, body {
    font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
    background-color:black !important;
    color: white !important;
}
....
```

Test index page to demo how to dynamically change out the stylesheet.

```csharp
@page "/"

<PageTitle>Index</PageTitle>

<h1>Hello, world!</h1>

Welcome to your new app.

<SurveyPrompt Title="How is Blazor working for you?" />

<div class="m-2">
    <button class="btn @this.buttonColour" @onclick="SwitchCSS">@this.buttonLabel</button>
</div>

@code {
    [Inject] private InteropLibrary? interopLibrary { get; set; }


    private bool isDark;
    private string css => isDark ? "/css/site-black.css" : "/css/site.css";
    private string buttonLabel => isDark ? "Switch To Light" : "Switch To Dark" ;
    private string buttonColour => isDark ? "btn-light" : "btn-dark" ;

    private async Task SwitchCSS()
    {
        this.isDark = !this.isDark;
        await interopLibrary!.SetDynamicCss("dynamiccsslink", this.css);
    }
}
```

[Source/Idea Material](https://github.com/chanan/BlazorStrap) 
