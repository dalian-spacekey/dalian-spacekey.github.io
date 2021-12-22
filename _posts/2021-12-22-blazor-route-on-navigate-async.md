---
layout: post
permalink: /:slug
categories: Blazor 
title: "[Blazor]Pageに遷移する前になにかしたい"
description: "RouterのOnNavigateAsyncをつかう。"
---

## [Blazor]Pageに遷移する前になにかしたい

RouterのOnNavigateAsyncをつかう。

App.razor

```razor
<Router AppAssembly="@typeof(App).Assembly"
        OnNavigateAsync="OnNavigateAsync"> 
    <Found Context="routeData">
        <RouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)" />
    </Found>
    <NotFound>
        <PageTitle>Not found</PageTitle>
        <LayoutView Layout="@typeof(MainLayout)">
            <p role="alert">Sorry, there's nothing at this address.</p>
        </LayoutView>
    </NotFound>
</Router>

@code
{
    private void OnNavigateAsync(NavigationContext navigationContext)
    {
        // ここでなにかする
        // どこに行こうとしてるかはnavigationContext.Pathに入ってる
    }
}
```

App.OnNavigateAsyncは、呼び出し先ページのレイアウトのOnInitializedよりも先に動きます。
なので、初回表示時とかリロード時などの場合、App, Layout, PageそれぞれのOnInitialized, OnParameterSet, OnAfterRenderとOnNavigateAsyncが動く順番は下記の通りになります(Blazor serverでプリレンダリングの時)。

* App.OnInitialized
* App.OnParametersSet
* App.OnNavigateAsync
* MainLayout.OnInitialized
* MainLayout.OnParametersSet
* MainLayout.OnParametersSet
* MainLayout.OnParametersSet
* Index.OnInitialized
* Index.OnParametersSet
* --- 2回目
* App.OnInitialized
* App.OnParametersSet
* App.OnNavigateAsync
* MainLayout.OnInitialized
* MainLayout.OnParametersSet
* MainLayout.OnParametersSet
* MainLayout.OnParametersSet
* Index.OnInitialized
* Index.OnParametersSet
* --- AfterRender
* App.OnAfterRender:firstRender=True
* App.OnAfterRender:firstRender=False
* MainLayout.OnAfterRender:firstRender=True
* Index.OnAfterRender:firstRender=True

この後、NavLink等でPage1に遷移すると、

* App.OnNavigateAsync
* MainLayout.OnParametersSet
* Page1.OnInitialized
* Page1.OnParametersSet
* App.OnAfterRender:firstRender=False
* MainLayout.OnAfterRender:firstRender=False
* Page1.OnAfterRender:firstRender=True

こういう順番でイベントが発生します。

なので、ページ遷移するたびに何かチェックするとか、各ページでDIで何か差し込まれる前にやっとかないといけない処理があるような場合に使えると思います。
ただし、場合によっては2回走るので、動くたびにバックエンドの状態を変えてしまうような処理は不具合の元でしょう。

