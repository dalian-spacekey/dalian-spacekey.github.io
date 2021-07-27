---
layout: post
permalink: /:slug
categories: Blazor
title: "[Blazor]JavaScriptコードを呼び出す"
---

## [Blazor]JavaScriptコードを呼び出す

IJSRuntimeをinjectして、InvokeVoidAsyncやInvokeAsyncで実行する。

Local storageや、IndexedDBなどの機能を使うときは必須になってくるし、UIを実装するときはこなれたライブラリを使ったほうが効率が良いと思う。
ただ、JavaScript呼出しを無計画に散りばめてしまうとコードの見通しも悪くなりそう。


### 1.JavaScriptファイルを作って、index.htmlで読み込まれるようにする

wwwroot\js\functions.js

```javascript
function showAlert(message, name) {
    alert(`${message}, ${name}`)
}

function getMessage(message, name) {
    return `${message}, ${name}`;
}
```

wwwroot\index.html

```html
...
<body>
    <div id="app">Loading...</div>

    <div>
        An unhandled error has occurred.
        <a href="" class="reload">Reload</a>
        <a class="dismiss">🗙</a>
    </div>
    <script src="_framework/blazor.webassembly.js"></script>
    <script src="js/functions.js"></script>
</body>
...
```

### 2.IJSRuntimeをinjectして呼び出す

JavaScriptSample.razor

```razor
@page "/JavaScriptSample"

<h3>JavaScriptSample</h3>

<div>
    <button type="button" @onclick="Button1Click">InvokeVoidAsync</button>
</div>

<div>
    <button type="button" @onclick="Button2Click">InvokeVoidAsync</button>
</div>

<div>
    <button type="button" @onclick="Button3Click">InvokeAsync</button>
    @button3Message
</div>


@code {
    [Inject]
    private IJSRuntime JSRuntime { get; set; }

    private string button3Message { get; set; } = "-";

    public async Task Button1Click()
    {
        await JSRuntime.InvokeVoidAsync("showAlert", new[] { "Hello", "Spacekey" });
    }

    public async Task Button2Click()
    {
        await JSRuntime.InvokeVoidAsync("window.alert", new[] { "Hello, Spacekey" });
    }

    public async Task Button3Click()
    {
        button3Message = await JSRuntime.InvokeAsync<string>("getMessage", new[] { "Hello", "Spacekey" });
    }
}
```

2番目の例は、中途半端なラッピングをするぐらいなら、直接書いたほうが意図が伝わるだろうという例。
