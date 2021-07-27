---
layout: post
permalink: /:slug
categories: Blazor
title: "[Blazor]LocalStorageにデータを保存する"
---

## [Blazor]LocalStorageにデータを保存する

IJSRuntimeで、localStorage.setItemやlocalStorage.getItemを使う。

複雑なデータを扱うときは、jsonにserialize/deserializeする。  
セッションだけにしたかったら、localStorageをsessionStorageにする。


```razor
@page "/LocalStorageSample"
@using Newtonsoft.Json

<h3>LocalStorageSample</h3>

<h4>string</h4>

<div>
    <input type="text" @bind="setTokenString">
    <button type="button" @onclick="SetToken">Set</button>
</div>

<div>
    <button type="button" @onclick="GetToken">Get</button>
    @getTokenResult
</div>

<div>
    <button type="button" @onclick="RemoveToken">Remove</button>
</div>

<h4>object</h4>

<div>
    <input type="text" @bind="userId">
    <input type="text" @bind="userName">
    <button type="button" @onclick="SetUserData">Set</button>
</div>

<div>
    <button type="button" @onclick="GetUserData">Get</button>
    @userData.UserId
    @userData.UserName
</div>

<div>
    <button type="button" @onclick="RemoveUserData">Remove</button>
</div>


@code {
    [Inject]
    private IJSRuntime JSRuntime { get; set; }

    private string setTokenString = "";
    private string getTokenResult = "-";

    private UserData userData = new UserData();
    private string userId = "";
    private string userName = "";

    private async Task SetToken()
    {
        await JSRuntime.InvokeVoidAsync("localStorage.setItem", "token", setTokenString);
    }
    
    private async Task GetToken()
    {
        getTokenResult = await JSRuntime.InvokeAsync<string>("localStorage.getItem", "token");
    }

    private async Task RemoveToken()
    {
        await JSRuntime.InvokeVoidAsync("localStorage.removeItem", "token");
    }


    private async Task SetUserData()
    {
        await JSRuntime.InvokeVoidAsync(
            "localStorage.setItem", 
            "userData", 
            JsonConvert.SerializeObject(
                new UserData { 
                    UserId = userId, 
                    UserName = userName 
                }
            ));
    }
    
    private async Task GetUserData()
    {
        var json = await JSRuntime.InvokeAsync<string>("localStorage.getItem", "userData");
        userData = json is null ? new UserData() : JsonConvert.DeserializeObject<UserData>(json);
    }

    private async Task RemoveUserData()
    {
        await JSRuntime.InvokeVoidAsync("localStorage.removeItem", "userData");
    }

    public class UserData
    {
        public string UserId { get; set; }
        public string UserName { get; set; }
    }
}
```

![result](/assets/2021-07-28-blazor-local-storage/1.png)
