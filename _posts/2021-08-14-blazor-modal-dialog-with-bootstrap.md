---
layout: post
permalink: /:slug
categories: Blazor Bootstrap
title: "[Blazor]BootstrapのModalを使って確認ダイアログを出す"
description: "Bootstrapのmodalをラップしたコンポーネントを作る"
---

## [Blazor]BootstrapのModalを使って確認ダイアログを出す

Bootstrapのmodalをラップしたコンポーネントを作る。

使っているBootstrapはv5.1。  
Hide時のDelayは、ダイアログのfadeが終わるのを待ってるだけなので、fadeしないなら不要。  

### Shared/ConfirmDialog.razor

```razor
<div class="modal fade @classShow" tabindex="-1" style="display: @display;">
    <div class="modal-dialog">
        <div class="modal-content">
            <div class="modal-header">
                <h5 class="modal-title">@Title</h5>
            </div>
            <div class="modal-body">
                @Message
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-secondary" @onclick="() => SelectAction(false)">@CancelButtonCaption</button>
                <button type="button" class="btn btn-primary" @onclick="() => SelectAction(true)">@OKButtonCaption</button>
            </div>
        </div>
    </div>
</div>

<div class="modal-backdrop fade @classShow" style="display: @display;"></div>

@code {
    [Parameter]
    public string Title { get; set; } = "Dialog title";

    [Parameter]
    public string Message { get; set; } = "Message";

    [Parameter]
    public string OKButtonCaption { get; set;  }

    [Parameter]
    public string CancelButtonCaption { get; set; }

    [Parameter]
    public EventCallback<bool> ActionSelected { get; set; }

    private string display = "none;";
    private string classShow = "";

    public async Task ShowDialog()
    {
        display = "block";
        await Task.Delay(50);
        classShow = "show";
        StateHasChanged();
    }

    private async Task HideDialog()
    {
        classShow = "";
        await Task.Delay(200);
        display = "none";
        StateHasChanged();
    }

    private async Task SelectAction(bool value)
    {
        await HideDialog();
        await ActionSelected.InvokeAsync(value);
    }
}
```

### ConfirmDialogSample.razor

```razor
@page "/ConfirmDialogSample"

<h3>ConfirmDialogSample</h3>

<div>
    <button 
        type="button" 
        class="btn btn-primary" 
        @onclick="() => confirmation.ShowDialog()">ShowDialog</button>
    Result : @result;
</div>

<ConfirmDialog 
    @ref="confirmation" 
    Title = "Confirm"
    Message = "Are you OK?"
    OKButtonCaption="Yes"
    CancelButtonCaption="No"
    ActionSelected="ActionSelected">
</ConfirmDialog>

@code {
    private bool result;
    private ConfirmDialog confirmation;

    private void ActionSelected(bool dialogResult)
    {
        result = dialogResult;
    }
}
```

![result](/assets/2021-08-14-blazor-modal-dialog-with-bootstrap/1.png)