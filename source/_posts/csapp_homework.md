---
layout: post
title: Android APM
date: 2019-10-16
categories: blog
tags: [csapp]
description: 简单了解Android APM的检测点
cover: https://raw.githubusercontent.com/HenryHaoson/HenryHaoson.github.io/source/source/images/nana0.png
---

### 2.55（别人写的，为后边的题目服务）

```c
//
// Created by 朱浩 on 2019-10-16.
//

/*这一段代码的大部分来自http://csapp.cs.cmu.edu/3e/students.html*/
/* $begin show-bytes */
#include <stdio.h>
/* $end show-bytes */
#include <stdlib.h>
#include <string.h>

/* $begin show-bytes */

typedef unsigned char *byte_pointer;

void show_bytes(byte_pointer start, size_t len) {
    size_t i;
    for (i = 0; i < len; i++)
        printf(" %.2x", start[i]);    //line:data:show_bytes_printf
    printf("\n");
}

void show_int(int x) {
    show_bytes((byte_pointer) &x, sizeof(int)); //line:data:show_bytes_amp1
}

void show_float(float x) {
    show_bytes((byte_pointer) &x, sizeof(float)); //line:data:show_bytes_amp2
}

void show_pointer(void *x) {
    show_bytes((byte_pointer) &x, sizeof(void *)); //line:data:show_bytes_amp3
}
/* $end show-bytes */


/* $begin test-show-bytes */
void test_show_bytes(int val) {
    int ival = val;
    float fval = (float) ival;
    int *pval = &ival;
    show_int(ival);
    show_float(fval);
    show_pointer(pval);
}

/* $end test-show-bytes */

void simple_show_a() {
/* $begin simple-show-a */
    int val = 0x87654321;
    byte_pointer valp = (byte_pointer) &val;
    show_bytes(valp, 1); /* A. */
    show_bytes(valp, 2); /* B. */
    show_bytes(valp, 3); /* C. */
/* $end simple-show-a */
}

void simple_show_b() {
/* $begin simple-show-b */
    int val = 0x12345678;
    byte_pointer valp = (byte_pointer) &val;
    show_bytes(valp, 1); /* A. */
    show_bytes(valp, 2); /* B. */
    show_bytes(valp, 3); /* C. */
/* $end simple-show-b */
}

void float_eg() {
    int x = 3490593;
    float f = (float) x;
    printf("For x = %d\n", x);
    show_int(x);
    show_float(f);

    x = 3510593;
    f = (float) x;
    printf("For x = %d\n", x);
    show_int(x);
    show_float(f);

}

void string_ueg() {
/* $begin show-ustring */
    const char *s = "ABCDEF";
    show_bytes((byte_pointer) s, strlen(s));
/* $end show-ustring */
}

void string_leg() {
/* $begin show-lstring */
    const char *s = "abcdef";
    show_bytes((byte_pointer) s, strlen(s));
/* $end show-lstring */
}

void show_twocomp() {
/* $begin show-twocomp */
    short x = 12345;
    short mx = -x;

    show_bytes((byte_pointer) &x, sizeof(short));
    show_bytes((byte_pointer) &mx, sizeof(short));
/* $end show-twocomp */
}

int main(int argc, char *argv[]) {
    int val = 12345;

    if (argc > 1) {
        if (argc > 1) {
            val = strtol(argv[1], NULL, 0);
        }
        printf("calling test_show_bytes\n");
        test_show_bytes(val);
    } else {
        printf("calling show_twocomp\n");
        show_twocomp();
        printf("Calling simple_show_a\n");
        simple_show_a();
        printf("Calling simple_show_b\n");
        simple_show_b();
        printf("Calling float_eg\n");
        float_eg();
        printf("Calling string_ueg\n");
        string_ueg();
        printf("Calling string_leg\n");
        string_leg();
    }
    return 0;
}
```

### 2.58

```c

//
// Created by 朱浩 on 2019-10-18.
//
#include <stdio.h>

union Data {
    int a;
    char b;
};

int is_little_endian() {
    union Data d;
    d.a = 1;
    if (d.b == 1) {
        return 1;
    } else {
        return 0;
    }
}

int main(int argc, char *argv[]) {
    printf("%d", is_little_endian());
}
```

### 2.59

```c

//
// Created by 朱浩 on 2019-10-18.
//

#include <stdio.h>

int main(int argc, char *argv[]) {
    int a = 0x89ABCDEF;
    int b = 0x76543210;


    printf("0x%.8X\n", (a & 0xff) | (b & ~0xff));
}
```

### 2.60

```c
//
// Created by 朱浩 on 2019-10-21.
//
#include <stdio.h>

unsigned replace_byte(unsigned int x, int i, unsigned char b) {
    unsigned int a = ((unsigned int) 0xff) << (i << 3);
    unsigned int p = ((unsigned int) b) << (i << 3);

    return (x & ~a) | p;
}

```

### 2.61

```c
//
// Created by 朱浩 on 2019-10-21.
//
#include <stdio.h>

int a(int x) {
    return !(~x);
}

int b(int x) {
    return ~x;
}

int c(int x) {
    return !~(x | ~0xff);
}

int d(int x) {
    return !(x & (0xff << ((sizeof(int) - 1) << 3)));
}

int main() {
    printf("%8x", d(0x00112233));
    printf("%8x", d(0x01112233));
    return 0;
}
```

### 2.62

```c
//
// Created by 朱浩 on 2019-10-22.
//
#include <stdio.h>
#include <string.h>

typedef unsigned char *byte_pointer;

void show_bytes(byte_pointer start, size_t len) {
    size_t i;
    for (i = 0; i < len; i++)
        printf(" %.2x", start[i]);    //line:data:show_bytes_printf
    printf("\n");
}

void show_int(int x) {
    show_bytes((byte_pointer) &x, sizeof(int)); //line:data:show_bytes_amp1
}


int int_shifts_are_arithmetic() {
    int x = 0xffffffff;
    show_int(x >> 1);
    return !(~(x >> 1));
}


int main() {

    printf("%d", int_shifts_are_arithmetic());
    return 0;
}
```

### 2.63

```c
//
// Created by 朱浩 on 2019-10-23.
//
#include <stdio.h>

unsigned srl(unsigned x, int k) {
    unsigned xsra = (int) x >> k;
    unsigned mask = ~0 << ((sizeof(int) << 3) - k);
    return xsra & ~mask;
}

// 用减1来实现填充（左边或者右边）
unsigned sra(int x, int k) {
    int xsrl = (unsigned) x >> k;
    unsigned p = 1 << ((sizeof(int) << 3) - k - 1);
    unsigned mask = p - 1;
    unsigned right = mask & xsrl;
    unsigned left = ~mask & (~(p & xsrl) + p);
    return right | left;
}
```

### 2.64

```c

//
// Created by 朱浩 on 2019-10-24.
//
#include <stdio.h>

int any_odd_one(unsigned x) {
    return !!(x & 0x55555555);
}

int main() {
    printf("%d\n", any_odd_one(0x50000000));
    printf("%d", any_odd_one(0x00000000));
    return 0;
}
```
