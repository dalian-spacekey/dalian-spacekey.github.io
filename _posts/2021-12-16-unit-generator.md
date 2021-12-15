---
layout: post
permalink: /:slug
categories: C# UnitGenerator
title: "[C#]UnitGeneratorを使う"
description: "UnitGeneratorで項目を簡単にValueObjectにして間違えないようにする"
---

## [C#]UnitGeneratorを使う

UnitGeneratorで項目を簡単にValueObjectにして間違えないようにする。

[Cysharp/UnitGenerator](https://github.com/Cysharp/UnitGenerator)

[C# Advent Calendar 2021](https://qiita.com/advent-calendar/2021/csharplang)の記事になってます。

### 課題

こんな感じのメソッドがあるとします。

```csharp
public void AddUser(int companyId, int userId)
{
    ...
}
```

指定したCompanyに指定したUserを追加する何かです。
おそらく、データベースに数値型の主キーをもつテーブルに入ってる感じのヤツです。

ただ、どっちも数値型なので、間違えて引数を逆に渡してしまう過ちを起こす可能性があります。
何かの拍子に逆に渡してしまっても普通にビルドされますし、両方のキーがそれぞれ存在してたりするとそのまま正常に終了しますので、何か大事になるまで気がつかない可能性すらあります。

まあ、これぐらいであれば、ちゃんと覚えておいて注意深くプログラミングして、テストコードを工夫するとかでなんとかなります。

ところが、仕様変更があり、なんかDepartmentも意識する必要が出ましたとか、

```csharp
public void AddUser(int companyId, int departmentId, int userId)
{
    ...
}
```

Companyはコンストラクタで指定することになったから、CompanyIdいらなくなりましたとか、

```csharp
public void AddUser(int departmentId, int userId)
{
    ...
}
```

DepartmentはDefaultが100なのでoptionalになりましたとか、

```csharp
public void AddUser(int userId, int departmentId = 100)
{
    ...
}
```

まあすでに動いてるものをここまでコロコロ変えることはないとは思うんですが、開発初期段階とか、一通り作った後リファクタリングするとか言う段階で、各種情報の識別子がC#上では同じ型だと、IDEのサポートがあったとしても書き換えるのは結構気を遣う作業になります。
(今まさにコレ)

そういうときに出てくるのが、DDDで言うところのValueObjectという、意味を持って型となすやり方で、UserIdであれば中身の値はintであってもUserId型というものをこさえて定義しましょう、UserId型とCompanyId型は中身の数値が同じであっても、型が違うので別のものですよ、という風にしましょうというアレです。

ところが…

```csharp
public struct UserId
{
}
```

みたいなコードを思いつくんですがちょっと待てよ、ってなるわけです。
生成したり、比較したり、DBに書き込むときどうするんだとか、jsonとかにシリアライズするときどうするんだとか、大いに脱線がはかどるため本筋の作業が滞ってしまうんです。

### 解決

そこで登場するのが、[Cysharp/UnitGenerator](https://github.com/Cysharp/UnitGenerator)。

こうするだけで中身がintのUserId型ができあがります。  
手間のかかる細かいプログラミング一切はUnitGeneratorが見えないところでやってくれます。

```csharp
[UnitOf(typeof(int))]
public readonly partial struct UserId { }
```

そうするとこう書けます。

```csharp
public class User
{
    public UserId UserId { get; set; }
    ...
}
```

Companyはこうなってるべきです。

```csharp
[UnitOf(typeof(int))]
public readonly partial struct CompanyId { }

public class Company
{
    public CompanyId CompanyId { get; set; }
    ...
}
```

もう、AddUserの引数型が別物になりましたので、誤って入れ替えてもビルドすらできません。

```csharp
public void AddUser(CompanyId companyId, UserId userId)
{
    ...
}

...

Company company = ...;
User user = ...;

AddUser(user.UserId, company.CompanyId); // ダメ
AddUser(company.CompanyId, user.Age);    // 誤って同じ型のフィールドに間違っても引っかかる
```

Departmentが追加されましたら…

```csharp
public void AddUser(CompanyId companyId, DepartmentId departmentId, UserId userId)
{
    ...
}

AddUser(company.CompanyId, user.UserId, department.DepartmentId); // ダメ
```

というように、同じ型の引数が入れ替わり立ち替わりしても間違うことはありません。

値の生成や比較など基本的なところはもちろん、オプションをつければ加減乗除や大小比較などもできるので、READMEにあるようなヒットポイントの計算みたいな使い方もできますし、JsonConverterオプションをつけておけば、ちゃんと元々の型で出てたのと同じようになります。

何も指定しなければ、JSONシリアライズするとこうなっちゃいますが、

```csharp
[UnitOf(typeof(int))]
public readonly partial struct UserId { }

var user = new User
{
    UserId = new UserId(123)
};

Console.WriteLine(JsonSerializer.Serialize(user));
{"UserId":{}}
```

UnitGenerateOptions.JsonConverterを指定すると、ちゃんとintだったときと同じようにしてくれます。

```csharp
[UnitOf(typeof(int), UnitGenerateOptions.JsonConverter)]
public readonly partial struct UserId { }

var user = new User
{
    UserId = new UserId(123)
};

Console.WriteLine(JsonSerializer.Serialize(user));
{"UserId":123}
```

ほかにも、MessagePackFormatter、DapperTypeHandler、EntityFrameworkValueConverterなどもあります。

さすが。

### 結論

識別子の項目だけでも使っておけば楽になれるはず。

キーをGUIDやUlidみたいな型で持ってたりすると、デバッグ中に値見ても間違ってるかどうか確認するのも大変だし、コード書いたときに違うって言ってくれたほうが良いに決まってますね。

データベースを扱うコードだと、どうやっても各テーブルのキー項目を引数でとりますし、開発序盤で探りながら作っていくと、項目が増えたり減ったり、順番が変わったり…みたいなのをやりやすくしておくと、コードを新鮮に保ちやすくなるので精神衛生上もイイ。

とはいえ、だからといって何でもかんでもValueObjectにするのはよくなさそう。

ユーザー定義の型ばっかり並んでしまうと、一体それがなんなのかっていうのがわかってないと使いづらくなりそうな気がしますので、「この問題を解決したいから使う」という使い方でいいかもしれません。


### 参考文献

[UnitGenerator - C# 9.0 SourceGeneratorによるValueObjectパターンの自動実装とSourceGenerator実装Tips
](https://neue.cc/2020/12/15_597.html)

コレを作ったneueさんの解説。
