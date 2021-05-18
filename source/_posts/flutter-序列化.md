---
layout: post
title: flutter-序列化
date: 2021-05-11
categories: blog
tags: [flutter, 序列化]
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
built_value 也是 google 提供的一个 json 处理库（还有其他功能，以 序列化 起家）。

主要功能：
- 全面支持面向对象设计：所有对象模型都可以被序列化，包括泛型和接口。（某些库不支持泛型）
- 他可以支持同一个数据反序列化成不同类型的对象，比如可以根据需求，选择需要的字段去序列化和反序列化。也就是 json 和模型的字段可以不是一一对应的（Gson 也可以）


#### 使用

1. 引入依赖

``` dart
dependencies:
  flutter:
    sdk: flutter
  built_value: ^8.0.6
  built_collection: ^5.0.0


  # The following adds the Cupertino Icons font to your application.
  # Use with the CupertinoIcons class for iOS style icons.
  cupertino_icons: ^1.0.2

dev_dependencies:
  flutter_test:
    sdk: flutter
  build_runner: ^2.0.3
  built_value_generator: ^8.0.6

```

built_collection 提供了不可变集合的序列化功能。built_value 内部也依赖了这个库。

build_runner 和 built_value_generator 是用来生成模版代码。


2. 编写模型

json 数据:

```json
 {
        "coins": 0,
        "icon_url": "string",
        "id": "string",
        "name": "string",
    }

```

我们要建立一个 MaterialBook 的模型，这个模型要实现 `Built` 类。

```dart
import 'package:built_value/built_value.dart';
import 'package:built_value/serializer.dart';
import 'package:built_collection/built_collection.dart';

part 'materialBook.g.dart';


abstract class MaterialBooks
    implements Built<MaterialBooks, MaterialBooksBuilder> {
  BuiltList<MaterialBook> get objects;

  // 匿名构造函数
  MaterialBooks._();

  // 提供序列化器
  static Serializer<MaterialBooks> get serializer => _$materialBooksSerializer;

  // 需要提供工厂方法
  factory MaterialBooks([void Function(MaterialBooksBuilder) updates]) =
  _$MaterialBooks;
}


abstract class MaterialBook
    implements Built<MaterialBook, MaterialBookBuilder> {
  String get id;

  int get coins;

  @BuiltValueField(wireName: "icon_url")
  String get iconUrl;

  String get name;

  // 匿名构造函数
  MaterialBook._();

  // 提供序列化器
  static Serializer<MaterialBook> get serializer => _$materialBookSerializer;

  // 需要提供工厂方法
  factory MaterialBook([void Function(MaterialBookBuilder) updates]) =
      _$MaterialBook;
}


```

其中 `materialBook.g.dart` 文件会通过插件生成，`MaterialBookBuilder` ,`MaterialBookSerializer` ,`_$MaterialBook` 这几个类会生成在文件当中。

```dart
// GENERATED CODE - DO NOT MODIFY BY HAND

part of 'materialBook.dart';

// **************************************************************************
// BuiltValueGenerator
// **************************************************************************

Serializer<MaterialBooks> _$materialBooksSerializer =
    new _$MaterialBooksSerializer();
Serializer<MaterialBook> _$materialBookSerializer =
    new _$MaterialBookSerializer();

class _$MaterialBooksSerializer implements StructuredSerializer<MaterialBooks> {
  @override
  final Iterable<Type> types = const [MaterialBooks, _$MaterialBooks];
  @override
  final String wireName = 'MaterialBooks';

  @override
  Iterable<Object> serialize(Serializers serializers, MaterialBooks object,
      {FullType specifiedType = FullType.unspecified}) {
    final result = <Object>[
      'objects',
      serializers.serialize(object.objects,
          specifiedType:
              const FullType(BuiltList, const [const FullType(MaterialBook)])),
    ];

    return result;
  }

  @override
  MaterialBooks deserialize(
      Serializers serializers, Iterable<Object> serialized,
      {FullType specifiedType = FullType.unspecified}) {
    final result = new MaterialBooksBuilder();

    final iterator = serialized.iterator;
    while (iterator.moveNext()) {
      final key = iterator.current as String;
      iterator.moveNext();
      final Object value = iterator.current;
      switch (key) {
        case 'objects':
          result.objects.replace(serializers.deserialize(value,
                  specifiedType: const FullType(
                      BuiltList, const [const FullType(MaterialBook)]))
              as BuiltList<Object>);
          break;
      }
    }

    return result.build();
  }
}

class _$MaterialBookSerializer implements StructuredSerializer<MaterialBook> {
  @override
  final Iterable<Type> types = const [MaterialBook, _$MaterialBook];
  @override
  final String wireName = 'MaterialBook';

  @override
  Iterable<Object> serialize(Serializers serializers, MaterialBook object,
      {FullType specifiedType = FullType.unspecified}) {
    final result = <Object>[
      'id',
      serializers.serialize(object.id, specifiedType: const FullType(String)),
      'coins',
      serializers.serialize(object.coins, specifiedType: const FullType(int)),
      'icon_url',
      serializers.serialize(object.iconUrl,
          specifiedType: const FullType(String)),
      'name',
      serializers.serialize(object.name, specifiedType: const FullType(String)),
    ];

    return result;
  }

  @override
  MaterialBook deserialize(Serializers serializers, Iterable<Object> serialized,
      {FullType specifiedType = FullType.unspecified}) {
    final result = new MaterialBookBuilder();

    final iterator = serialized.iterator;
    while (iterator.moveNext()) {
      final key = iterator.current as String;
      iterator.moveNext();
      final Object value = iterator.current;
      switch (key) {
        case 'id':
          result.id = serializers.deserialize(value,
              specifiedType: const FullType(String)) as String;
          break;
        case 'coins':
          result.coins = serializers.deserialize(value,
              specifiedType: const FullType(int)) as int;
          break;
        case 'icon_url':
          result.iconUrl = serializers.deserialize(value,
              specifiedType: const FullType(String)) as String;
          break;
        case 'name':
          result.name = serializers.deserialize(value,
              specifiedType: const FullType(String)) as String;
          break;
      }
    }

    return result.build();
  }
}

class _$MaterialBooks extends MaterialBooks {
  @override
  final BuiltList<MaterialBook> objects;

  factory _$MaterialBooks([void Function(MaterialBooksBuilder) updates]) =>
      (new MaterialBooksBuilder()..update(updates)).build();

  _$MaterialBooks._({this.objects}) : super._() {
    BuiltValueNullFieldError.checkNotNull(objects, 'MaterialBooks', 'objects');
  }

  @override
  MaterialBooks rebuild(void Function(MaterialBooksBuilder) updates) =>
      (toBuilder()..update(updates)).build();

  @override
  MaterialBooksBuilder toBuilder() => new MaterialBooksBuilder()..replace(this);

  @override
  bool operator ==(Object other) {
    if (identical(other, this)) return true;
    return other is MaterialBooks && objects == other.objects;
  }

  @override
  int get hashCode {
    return $jf($jc(0, objects.hashCode));
  }

  @override
  String toString() {
    return (newBuiltValueToStringHelper('MaterialBooks')
          ..add('objects', objects))
        .toString();
  }
}

class MaterialBooksBuilder
    implements Builder<MaterialBooks, MaterialBooksBuilder> {
  _$MaterialBooks _$v;

  ListBuilder<MaterialBook> _objects;
  ListBuilder<MaterialBook> get objects =>
      _$this._objects ??= new ListBuilder<MaterialBook>();
  set objects(ListBuilder<MaterialBook> objects) => _$this._objects = objects;

  MaterialBooksBuilder();

  MaterialBooksBuilder get _$this {
    final $v = _$v;
    if ($v != null) {
      _objects = $v.objects.toBuilder();
      _$v = null;
    }
    return this;
  }

  @override
  void replace(MaterialBooks other) {
    ArgumentError.checkNotNull(other, 'other');
    _$v = other as _$MaterialBooks;
  }

  @override
  void update(void Function(MaterialBooksBuilder) updates) {
    if (updates != null) updates(this);
  }

  @override
  _$MaterialBooks build() {
    _$MaterialBooks _$result;
    try {
      _$result = _$v ?? new _$MaterialBooks._(objects: objects.build());
    } catch (_) {
      String _$failedField;
      try {
        _$failedField = 'objects';
        objects.build();
      } catch (e) {
        throw new BuiltValueNestedFieldError(
            'MaterialBooks', _$failedField, e.toString());
      }
      rethrow;
    }
    replace(_$result);
    return _$result;
  }
}

class _$MaterialBook extends MaterialBook {
  @override
  final String id;
  @override
  final int coins;
  @override
  final String iconUrl;
  @override
  final String name;

  factory _$MaterialBook([void Function(MaterialBookBuilder) updates]) =>
      (new MaterialBookBuilder()..update(updates)).build();

  _$MaterialBook._({this.id, this.coins, this.iconUrl, this.name}) : super._() {
    BuiltValueNullFieldError.checkNotNull(id, 'MaterialBook', 'id');
    BuiltValueNullFieldError.checkNotNull(coins, 'MaterialBook', 'coins');
    BuiltValueNullFieldError.checkNotNull(iconUrl, 'MaterialBook', 'iconUrl');
    BuiltValueNullFieldError.checkNotNull(name, 'MaterialBook', 'name');
  }

  @override
  MaterialBook rebuild(void Function(MaterialBookBuilder) updates) =>
      (toBuilder()..update(updates)).build();

  @override
  MaterialBookBuilder toBuilder() => new MaterialBookBuilder()..replace(this);

  @override
  bool operator ==(Object other) {
    if (identical(other, this)) return true;
    return other is MaterialBook &&
        id == other.id &&
        coins == other.coins &&
        iconUrl == other.iconUrl &&
        name == other.name;
  }

  @override
  int get hashCode {
    return $jf($jc(
        $jc($jc($jc(0, id.hashCode), coins.hashCode), iconUrl.hashCode),
        name.hashCode));
  }

  @override
  String toString() {
    return (newBuiltValueToStringHelper('MaterialBook')
          ..add('id', id)
          ..add('coins', coins)
          ..add('iconUrl', iconUrl)
          ..add('name', name))
        .toString();
  }
}

class MaterialBookBuilder
    implements Builder<MaterialBook, MaterialBookBuilder> {
  _$MaterialBook _$v;

  String _id;
  String get id => _$this._id;
  set id(String id) => _$this._id = id;

  int _coins;
  int get coins => _$this._coins;
  set coins(int coins) => _$this._coins = coins;

  String _iconUrl;
  String get iconUrl => _$this._iconUrl;
  set iconUrl(String iconUrl) => _$this._iconUrl = iconUrl;

  String _name;
  String get name => _$this._name;
  set name(String name) => _$this._name = name;

  MaterialBookBuilder();

  MaterialBookBuilder get _$this {
    final $v = _$v;
    if ($v != null) {
      _id = $v.id;
      _coins = $v.coins;
      _iconUrl = $v.iconUrl;
      _name = $v.name;
      _$v = null;
    }
    return this;
  }

  @override
  void replace(MaterialBook other) {
    ArgumentError.checkNotNull(other, 'other');
    _$v = other as _$MaterialBook;
  }

  @override
  void update(void Function(MaterialBookBuilder) updates) {
    if (updates != null) updates(this);
  }

  @override
  _$MaterialBook build() {
    final _$result = _$v ??
        new _$MaterialBook._(
            id: BuiltValueNullFieldError.checkNotNull(id, 'MaterialBook', 'id'),
            coins: BuiltValueNullFieldError.checkNotNull(
                coins, 'MaterialBook', 'coins'),
            iconUrl: BuiltValueNullFieldError.checkNotNull(
                iconUrl, 'MaterialBook', 'iconUrl'),
            name: BuiltValueNullFieldError.checkNotNull(
                name, 'MaterialBook', 'name'));
    replace(_$result);
    return _$result;
  }
}

// ignore_for_file: always_put_control_body_on_new_line,always_specify_types,annotate_overrides,avoid_annotating_with_dynamic,avoid_as,avoid_catches_without_on_clauses,avoid_returning_this,lines_longer_than_80_chars,omit_local_variable_types,prefer_expression_function_bodies,sort_constructors_first,test_types_in_equals,unnecessary_const,unnecessary_new

```

编写好之后就给 序列化器添加 插件

```dart
import 'package:built_value/serializer.dart';
import 'package:built_value/standard_json_plugin.dart';
import 'materialBook.dart';
import 'package:built_collection/built_collection.dart';


part 'serializers.g.dart';

@SerializersFor(const[
  MaterialBooks,
  MaterialBook,
])
final Serializers serializers  =
(_$serializers.toBuilder()..addPlugin(StandardJsonPlugin())).build();
```

这里用的是`StandardJsonPlugin` 。默认的也可以使用

> 因为built value的json格式不是标准的, 而是所有字段逗号分隔的. 用了`StandardJsonPlugin`之后就转换成了标准的JSON格式.


调用 serializers.

```dart
import 'dart:async';
import 'dart:convert';
import 'package:shanbay_book_demo/materialBook.dart';

import 'serializers.dart';
import 'package:http/http.dart' as http;

const BOOK_API =
    "https://apiv3.shanbay.com/wordsapp/material_books?tag_names=%E5%9B%9B%E7%BA%A7";

Future<MaterialBooks> getMaterialBooks() async {
  final res = await http.get(Uri.parse(BOOK_API));
  MaterialBooks materialBooks = serializers.deserializeWith(
      MaterialBooks.serializer, json.decode(res.body));
  print(materialBooks);
  return materialBooks;
}


// 打印结果
flutter: MaterialBooks {
  objects=[MaterialBook {
    id=odcxi,
    coins=0,
    iconUrl=https://media-image1.baydn.com/wordmaster_pub_image/tvbuvh/7e2398d03c75839c7cf3a561b16b0be6.f7d523801066b1361fb30cf71dba9edc.jpg?x-oss-process=image/quality,Q_80,
    name=四级考纲词汇(完全版),
  }, MaterialBook {
    id=odihq,
    coins=0,
    iconUrl=https://media-image1.baydn.com/wordmaster_pub_image/verskb/dd78d6e102df08ad84436581c0987a87.f69ccc840779614f1073f38f530b7aa5.jpg?x-oss-process=image/quality,Q_80,
    name=四级考纲词汇(精简版),
  }, MaterialBook {
    id=fxlze,
    coins=0,
    iconUrl=https://media-image1.baydn.com/wordmaster_pub_image/qayvad/607135b0de61e5607820c7b9b8dbd197.85c6515e085351bdff02d6fa8534c956.png?x-oss-process=image/quality,Q_80,
    name=四级真题核心词汇书,
  }],
}
```




