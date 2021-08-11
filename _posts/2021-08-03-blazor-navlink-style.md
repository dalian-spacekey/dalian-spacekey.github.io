---
layout: post
permalink: /:slug
categories: Blazor
title: "[Blazor]NavLinkにスタイルを適用する"
description: "cssクラスに::deepをつける"
---

## [Blazor]NavLinkにスタイルを適用する

cssクラスに::deepをつける。

bootstrapのnavとかでリンクにスタイルを入れたくても、NavLinkだと効いてくれないのに対処。

### NavLinkStyle.razor

```razor
@page "/NavLinkStyle"

<h3>NavLinkStyle</h3>

<h4>::deepなし</h4>

<h5>a</h5>

<ul class="navbar-nav nested-items">
    <li class="nav-item">
        <a href="#">Menu1</a>
    </li>
    <li class="nav-item">
        <a href="#">Menu2</a>
    </li>
    <li class="nav-item">
        <a href="#">Menu3</a>
    </li>
</ul>

<h5>NavLink</h5>

<ul class="navbar-nav nested-items">
    <li class="nav-item">
        <NavLink href="#">Menu1</NavLink>
    </li>
    <li class="nav-item">
        <NavLink href="#">Menu2</NavLink>
    </li>
    <li class="nav-item">
        <NavLink href="#">Menu3</NavLink>
    </li>
</ul>

<hr />

<h4>::deepあり</h4>

<h5>a</h5>

<ul class="navbar-nav nested-items-with-deep">
    <li class="nav-item">
        <a href="#">Menu1</a>
    </li>
    <li class="nav-item">
        <a href="#">Menu2</a>
    </li>
    <li class="nav-item">
        <a href="#">Menu3</a>
    </li>
</ul>

<h5>NavLink</h5>

<ul class="navbar-nav nested-items-with-deep">
    <li class="nav-item">
        <NavLink href="#">Menu1</NavLink>
    </li>
    <li class="nav-item">
        <NavLink href="#">Menu2</NavLink>
    </li>
    <li class="nav-item">
        <NavLink href="#">Menu3</NavLink>
    </li>
</ul>


@code {}
```

### NavLinkStyle.razor.css

```css
.nested-items a {
    margin-left: 2rem;
    text-decoration: none;
}

.nested-items-with-deep ::deep a {
    margin-left: 2rem;
    text-decoration: none;
}
```

![result](/assets/2021-08-03-blazor-navlink-style/1.png)