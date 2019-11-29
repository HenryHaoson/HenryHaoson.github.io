---
layout: post
title: node+formidable实现上传
date: 2017-8-7
categories: blog
tags: [nodejs]
description: nodejs上传接口的编写，以及上传后图片地址的处理

---
导入formidable模块
```
let formidable=require('formidable');

```
获取formidable对象
```
var form = new formidable.IncomingForm();

```
设置formidable属性
```
     //设置编辑
    form.encoding = 'utf-8';
    //设置文件存储路径
    form.uploadDir = "./public/images/";
    //保留后缀
    form.keepExtensions = true;
    //设置单文件大小限制
    form.maxFieldsSize = 1024 * 1024 * 1024;
    //form.maxFields = 1000;  设置所以文件的大小总和
```
parse函数form.parse(req, function(err, fields, files)
files和field对象可以打印出来看看是什么，打印的时候不要+“”。
```
var token =fields.token;
        var decodeToken=jwtHelper.tokenDecode(token,config.jwt_secret);
        for(let i in files){console.log(i);console.log(files[i])}
        var filename = files.file.name;
        // 对文件名进行处理，以应对上传同名文件的情况
        var nameArray = filename.split('.');
        var type = nameArray[nameArray.length-1];
        var name = '';
        for(var i=0; i<nameArray.length-1; i++){
            name = name + nameArray[i];
        }
        var rand = Math.random()*100 + 900;
        var num = parseInt(rand, 10);

        var avatarName = name + num +  '.' + type;
        var newpath =  './public/images/'+avatarName;
    
```
传的参数通过fields来获取，files里面是存放的文件对象数组。
