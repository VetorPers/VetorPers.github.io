---
layout:         post
title:          Laravel中使用ajax
description:    在laravel中使用ajax遇到的token问题.
categories: laravel
keywords: laravel ,ajax
--- 
 
在使用 Laravel 使用 ajax 提交数据的时候经常遇到token提交的问题.

1.在视图文件的 head 部分包含一个 meta，用来保存 token 的值：

<pre class="html" name="colorcode">
&lt;meta name="csrf_token" id="token" content="csrf_token()"&gt;
</pre>

2.在使用 ajax 的时候就通过js获取到上面的 token 值，传给 Laravel 相对应的路由：

JQuery 的通常写法：

<pre class="js" name="colorcode">
$.ajaxSetup({
        headers: {
            'X-CSRF-TOKEN': $('meta[name="csrf_token"]').attr('content')
        }
});
</pre>

Vuejs 的通常会使用 Vue-resource，写法是这样：

<pre class="js" name="colorcode">
Vue.http.headers.common['X-CSRF-TOKEN'] = document.querySelector('#token').getAttribute('content');

new Vue({
   el: '#container'
});
</pre>
