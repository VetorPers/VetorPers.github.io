---
layout: post
title: larvel 当中预加载常用的关联模型
categories: laravel
description: laravel 小技巧
keywords: laravel
---

在 laravel 中，我们经常使用预加载来避免 N + 1 的问题。
```opp
Post::with('comments')->get();

$post = Post::first();
$post->load('comments');
```
但是，我们还有一种方法可以实现。
```opp
// App\Post.php

/**
 * The relationships to eager load with the model.
 * 
 * @var array
 */
protected $with = [
    'comments',
];
```
这样我们在每次查询 Post 模型的时候，都会对 'comments' 进行预加载。
我们还可以在模型中使用 withcount() 方法，来获得关联模型的数量。
像这样：
```opp
// App\Post.php

/**
 * The relationships to eager load with the model count.
 * 
 * @var array
 */
protected $withCount = [
    'comments',
];
```
