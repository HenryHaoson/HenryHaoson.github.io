---
layout: post
title: flutter-序列化
date: 2021-5-11
categories: blog
tags: [flutter]
description: flutter 序列化
---

这里不会赘述序列化反序列化的概念，旨在搞明白如何在 flutter 中进行 。

官方文档给了两种序列化的手段。

- 手动序列化数据
- 使用代码生成

> flutter 没有提供类似 Gson 之类的库，因为这样的库都是通过反射去做的。运行时反射和 dart 支持的 tree shaking 冲突（本质上就是去除无用资源，无引用代码，反射调用不好检查）。

## 手动序列化数据

手动序列化就是利用的 dart:convert 。可以做到简单的序列号和反序列化操作。

convert 提供了一个 `jsonDecode()` 方法，利用这个方法,可以将 json 字符串数据转换成一个Json对象（其实类型是一个Map）

``` dart

// api 文档中的类型是 dynamic
dynamic jsonDecode (
    String source,
    {Object? reviver(
        Object? key,
        Object? value
)}
)

/**
Parses the string and returns the resulting Json object.

The optional reviver function is called once for each object or list property that has been parsed during decoding. The key argument is either the integer list index for a list property, the string map key for object properties, or null for the final result.

The default reviver (when not provided) is the identity function.

Shorthand for json.decode. Useful if a local variable shadows the global json constant.
*/

// 官方序列化的文档说明返回类型是 Map<String, dynamic>
val booksJsonStr = "{"ipp":10,"objects":[{"book_item_list_id":"putse","book_type":0,"coins":0,"description":"匹配最新四级考纲，包含所有派生词，稳妥过四级","dictionary_id":"bcpxsc","ext_example_name":"","ext_example_type":1,"icon_url":"https:\/\/wordmaster_pub_image/tvbuvh/7e2398d03c75839c7cf3a561b16b0be6.f7d523801066b1361fb30cf71dba9edc.jpg?x-oss-process=image/quality,Q_80","use_ext_example":true,"use_original_example":false}]}"
Map<String, dynamic> books = jsonDecode(booksJsonStr);

```

由于 map 的 value type 是 dynamic，所以在运行时的时候是不知道类型的。这个对于静态语言使用者就很不适应。

你并不知道 `books["ipp"]` 会是一个什么类型，在使用的时候很容易出错（其实对 dart 的动态特性还是没有很理解为什么要支持）。

所以我们一般会和模型结合去进行序列化和反序列化。

可以选择给模型实现序列化和反序列化的方法， 比如`fromJson`,`toJson`

``` dart
class Books{
    final int ipp;
    final int page;
    final List<Book> objects;

    Books.fromJson()
}

```


