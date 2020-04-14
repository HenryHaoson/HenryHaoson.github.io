---
layout: post
title: FileProvider 学习
date: 2018-02-05
categories: blog
tags: [Android]
description: 在使用Provider的是时候发现并不能通过Uri获取到data信息
---

## FileProvider 是什么

FileProvider 是 ConetentProvider 的子类,在安卓 5.1 中添加到库中，这个是用来作为文件共享的组件。

我们通常在在 Android 7.0 之后通过`getContentProvider.query`根据 Uri 来查询文件信息。

可以看到下面代码，在查询的时候并没有给出`_data`的值，所以拿不到 file 的绝对路径。

```java

    @Override
    public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs,
            String sortOrder) {
        // ContentProvider has already checked granted permissions
        final File file = mStrategy.getFileForUri(uri);

        if (projection == null) {
            projection = COLUMNS;
        }

        String[] cols = new String[projection.length];
        Object[] values = new Object[projection.length];
        int i = 0;
        for (String col : projection) {
            if (OpenableColumns.DISPLAY_NAME.equals(col)) {
                cols[i] = OpenableColumns.DISPLAY_NAME;
                values[i++] = file.getName();
            } else if (OpenableColumns.SIZE.equals(col)) {
                cols[i] = OpenableColumns.SIZE;
                values[i++] = file.length();
            }
        }

        cols = copyOf(cols, i);
        values = copyOf(values, i);

        final MatrixCursor cursor = new MatrixCursor(cols, 1);
        cursor.addRow(values);
        return cursor;
    }

```
