---
layout: post
permalink: /:slug
categories: Blazor
title: "[Blazor]イベントのメソッド呼び出しに引数を渡す"
---

## [Blazor]イベントのメソッド呼び出しに引数を渡す

lambda式で呼出しコード自体を渡す。

```razor
@page "/WithArgument"

<h3>WithArgument</h3>

<h4>foreach</h4>

<ul>
    @foreach(var sample in samples)
    {
        <li>
            <button @onclick="()=> Click1(sample.Id)">Button-@sample.Id</button>
        </li>    
    }
</ul>

<div>
    @message1
</div>

<h4>for</h4>

<ul>
    @for(var i = 0; i < 5; i++)
    {
        var j = i;
        <li>
            <button @onclick="()=> Click2(j)">Button-@i</button>
        </li>    
    }
</ul>

<div>
    @message2
</div>

@code {
    private Sample[] samples = new Sample[] {
        new Sample { Id = 1, Name = "Sample1" },
        new Sample { Id = 2, Name = "Sample2" },
        new Sample { Id = 3, Name = "Sample3" },
        new Sample { Id = 4, Name = "Sample4" },
        new Sample { Id = 5, Name = "Sample5" }
    };

    private string message1;
    private string message2;

    private void Click1(int id)
    {
        message1 = $"Button-{id} clicked";
    }

    private void Click2(int i)
    {
        message2 = $"Button-{i} clicked";
    }

    private class Sample
    {
        public int Id { get; set; }
        public string Name { get; set; }
    }
}
```

ループ内で指定する場合、オブジェクトを参照する場合はちゃんと区別してくれるが、値渡しの場合は一度変数の詰め替えを行わないと、ループの最後の値ですべてスクリプト化されてしまうので注意。
(forの例で、iをそのまま使うと全部「Button-4 clicked」になってしまう)