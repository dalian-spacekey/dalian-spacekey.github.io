---
layout: post
permalink: /:slug
categories: Blazor
title: "[Blazor]テキストボックスの変更イベントを入力中に起こす"
---

## [Blazor]テキストボックスの変更イベントを入力中に起こす

`@bind-value`でbindして、`@bind-value:event`をoninputにする。


`@bind`のみで記述すると、内部的には「valueプロパティにbindして、onChangeでイベント発火」という処理が行われるので、入力後フォーカスが外れるタイミングでイベントが起きる。
3番目の書き方と同じ結果になる。

4番目は@bindのイベントをonChangeで差し替える例。


```razor
@page "/InputSample"

<h3>InputSample</h3>

<div>
    <h4>OnInput</h4>
    <input type="text" @bind-value="TextValue1" @bind-value:event="oninput">
    <button type="button" disabled="@ButtonDisabled1">button</button>
</div>

<div>
    <h4>Bind</h4>
    <input type="text" @bind="TextValue2">
    <button type="button" disabled="@ButtonDisabled2">button</button>
</div>

<div>
    <h4>OnChange</h4>
    <input type="text" @bind-value="TextValue3" @bind-value:event="onchange">
    <button type="button" disabled="@ButtonDisabled3">button</button>
</div>

<div>
    <h4>Bind-OnChange</h4>
    <input type="text" @bind="TextValue4" @bind:event="oninput">
    <button type="button" disabled="@ButtonDisabled4">button</button>
</div>

@code {
    private string TextValue1 { get; set; }
    private string TextValue2 { get; set; }
    private string TextValue3 { get; set; }
    private string TextValue4 { get; set; }

    private bool ButtonDisabled1 => string.IsNullOrEmpty(TextValue1);
    private bool ButtonDisabled2 => string.IsNullOrEmpty(TextValue2);
    private bool ButtonDisabled3 => string.IsNullOrEmpty(TextValue3);
    private bool ButtonDisabled4 => string.IsNullOrEmpty(TextValue4);
}
```
