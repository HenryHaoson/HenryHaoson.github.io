---
layout: post
title: flutter-序列化
date: 2021-05-11
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

    // 使用命名构造函数，通过json串来构造 Books 对象
    Books.fromJson(String jsonStr){
        Map<String, dynamic> books = jsonDecode(booksJsonStr);
        this.ipp = books["ipp"];
        this.page = books["page"];
        this.objects = // List Json 操作
    }

    String toJson(){
        // toString 拼接
    }
}

```

但是每个类都这样编写序列化和反序列化都操作就显得很繁琐，所以 flutter 提供了代码生成的库，不需要人为去编写这部分代码，而是根据规则自动生成。

## 使用生成库来进行序列化操作

现在有两个主流的库可以帮助我们做这件事情，一个是 json_serializable ，另一个是 built_value.

### json_serializable

json_serializable 通过注解的方式来自动生成我们的代码。和butterknife的思想很像（也是来帮助我们减少代码的编写）。

#### 依赖

```dart
dependencies:
  # Your other regular dependencies here
  json_annotation: <latest_version>

dev_dependencies:
  # Your other dev_dependencies here
  build_runner: <latest_version>
  json_serializable: <latest_version>
```

#### 使用

还是以 Books 为例，使用`@JsonSerializable()`注解来描述 Books 类。

```dart
import 'package:json_annotation/json_annotation.dart';

part 'books.g.dart';

class Books {
  Books(this.ipp, this.page, this.objects);

  int ipp;
  int page;
  List<Book> objects;

  factory Books.fromJson(Map<String, dynamic> jsonMap) => _$BooksFromJson(jsonMap);

  /// `toJson` is the convention for a class to declare support for serialization
  /// to JSON. The implementation simply calls the private, generated
  /// helper method `_$BooksToJson`.
  Map<String, dynamic> toJson() => _$BooksToJson(this);
}
```

需要提供一个工厂构造方法，这个方法把一个 map 传给由代码生成器声生成的方法 _$BooksFromJson(jsonMap) 来构造一个 Books 对象。

toJson 方法调用的也是由代码生成器生成的方法 _BooksToJson 。这个生成的方法可以序列化 Books 对象。

如果有嵌套的对象，比如说 Book，那么就可以依照 Books 的形式写就好了，但是需要注意一点。需要再父类声明的时候，给 @JsonSerializable注解方法加一个参数。像下边示例那样。

```dart
@JsonSerializable(explicitToJson: true)
class Books{
    Book book;
}

```

关于序列化还有一个就是字段映射规则。有时候我们的模型字段的命名规则可能和后端不一样，json_serializable 提供了灵活配置规则的能力。

1. 整体规则
```dart
 @JsonSerializable(fieldRename: FieldRename.snake)
 class Book{
     String coverUrl;
 }

 // json格式：
 {
     cover_url:"xxxxx"
 }

```

2. JsonKey 单独去配置
可以有很多的配置项，不仅仅是格式
```dart
// 如果 json 结构中没有这个字段或者字段的值是null，那么就赋值默认值 false
@JsonKey(defaultValue: false)
final bool isFree;

// json 中必须有这个字段，如果没有就会抛异常
@JsonKey(required: true)
final String id;

// 序列化的时候忽略这个字段。这个字段不参与序列化和反序列化。
@JsonKey(ignore: true)
final String verificationCode;

// json 字段格式
@JsonKey(name: '<snake_case>')
final String coverUrl;

// json中的word_count 匹配这个字段
@JsonKey(name: 'word_count')
final int bookWordCount;
```

接下来就是怎么去生成代码：
文档里提供了两种方式：
> 一次性代码生成
> 通过在项目根目录运行 flutter pub run build_runner build，你可以在任何需要的时候为你的模型生成 JSON 序列化数据代码。这会触发一次构建，遍历源文件，选择相关的文件，然后为它们生成必须的序列化数据代码。
> 虽然这样很方便，但是如果你不需要在每次修改了你的模型类后都要手动构建那将会很棒。

> 持续生成代码
> 监听器 让我们的源代码生成过程更加方便。它会监听我们项目中的文件变化，并且会在需要的时候自动构建必要的文件。你可以在项目根目录运行 flutter pub run build_runner watch 启动监听。
> 启动监听并让它留在后台运行是安全的。

### built_value
// todo



