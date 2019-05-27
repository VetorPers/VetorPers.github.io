---
layout: post
title: 使用 larvel-collect 包来实现收藏功能
categories: laravel
description: laravel collect
keywords: laravel
---

[Laravel Collect](https://github.com/VetorPers/laravel-collect) 是我开发的一个收藏文章的扩展，借鉴于 [cybercog/laravel-love](https://github.com/cybercog/laravel-love) ，我也有幸参加了社区对该扩展的外文翻译文章 [为你的 Eloquent 模型添加喜欢和讨厌功能](https://learnku.com/laravel/t/15898?#SectionIndex_4)。我的初衷是学习怎么开发 Laravel 扩展包，所以实现的功能可能比较简单，请大神勿喷。但是对于想学习开发 Laravel 扩展包的同学还是不错的。望大家点赞支持，感谢。


## 安装

通过 composer 安装，命令如下：

```sh
$ composer require vetor/laravel-collect
```

我们需要执行模型迁移命令，将 `Collections` 表发布到我们的数据库：

```sh
$ php artisan migrate
```

## 使用

在我们的收藏者表，即 `User` 表里需要实现 `CollectorContract` 接口，并引用 `Collector trait`:

```
use Illuminate\Foundation\Auth\User as Authenticatable;
use Vetor\Laravel\Collect\Collector\Models\Traits\Collector;
use Vetor\Contracts\Collect\Collector\Models\Collector as CollectorContract;

class User extends Authenticatable implements CollectorContract
{
    use Collector;
}
```

如果用户需要收藏文章，在 `Article` 表里实现 `CollectableContract` 接口并引用 `Collectable trait` 即可：

```
use Vetor\Laravel\Collect\Collectable\Models\Traits\Collectable;
use Vetor\Contracts\Collect\Collectable\Models\Collectable as CollectableContract;

class Article extends Model implements CollectableContract
{
    use Collectable;
}
```


## 可用的方法

对于用户来说,可用的方法有：

```
// 收藏
$user->collect($article);

// 取消收藏
$user->cancelCollect($article);

// 用户的所有收藏记录
$user->collections;

// 用户收藏的文章记录
$user->collectionsWhereCollectable(Article::class);
```

文章可用的方法有：

```
// 收藏
$article->collect();

// 取消收藏(默认为当前用户，可以把用户实例作为参数传入)
$article->cancelCollect();

//  获取文章的收藏情况
$article->collections();

// 获取文章收藏数
$article->collections_count;

// 根据收藏数排序(升序 'asc'；降序 'desc'；默认为升序)
Article::orderByCollectionsCount()->get();
```

我们可以通过下面的方法来获取收藏表里所有文章：
```
Collection::whereCollectable(Article::class)->get();
```

## 更多

代码参见 Github 仓库 [vetor/laravel-collect](https://github.com/VetorPers/laravel-collect)，欢迎大家提出自己的想法，指出不足，我们一起学习进步。再次感谢 [cybercog/laravel-love](https://github.com/cybercog/laravel-love) 。
