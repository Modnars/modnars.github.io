---
title: 《C语言程序设计》
date: 2020-01-05 11:16:41
abstract: 《K&R》练习题思考与答案
tags: ["C/C++"]
categories: "Notes"
---

## Prac 1-1

根据去掉内容的不同，编译器给出的错误(警告)提示不同。

## Prac 1-2

Code:

``` 
#include <stdio.h>

int main(int argc, const char *argv[]) {
    printf("Test->\c");
    return 0;
}
```

Compiler:

``` 
main.c:4:19: warning: unknown escape sequence '\c' [-Wunknown-escape-sequence]
    printf("Test->\c");
                  ^~
1 warning generated.
```

Output:

```
Test->c
```

## Prac 1-3

Code:

```
#include <stdio.h>

/**
 * 当fahr=0, 20, ..., 300时, 打印华氏温度与摄氏温度对照表;
 * 浮点数版本
 */

int main(int argc, const char *argv[]) {
    float fahr, celsius;
    int lower, upper, step;
    lower = 0;
    upper = 300;
    step = 20;
    printf("Transform Table\n");
    fahr = lower;
    while (fahr <= upper) {
        celsius = (5.0/9.0) * (fahr-32.0);
        printf("%3.0f %6.1f\n", fahr, celsius);
        fahr = fahr + step;
    }
    return 0;
}
```

## Prac 1-4

Code:

```
#include <stdio.h>

/**
 * 当celsius=0, 20, ..., 300时, 打印摄氏温度与华氏温度对照表;
 * 浮点数版本
 */

int main(int argc, const char *argv[]) {
    float fahr, celsius;
    int lower, upper, step;
    lower = 0;
    upper = 300;
    step = 20;
    printf("Transform Table\n");
    celsius = lower;
    while (celsius <= upper) {
        fahr = (9.0/5.0) * celsius + 32.0; 
        printf("%3.0f %6.1f\n", celsius, fahr);
        celsius = celsius + step;
    }
    return 0;
}
```

## Prac 1-5

Code 
```
#include <stdio.h>

/**
 * 当fahr=0, 20, ..., 300时, 打印华氏温度与摄氏温度对照表;
 * 浮点数版本
 */

int main(int argc, const char *argv[]) {
    int fahr;
    for (fahr = 300; fahr >= 0; fahr = fahr - 20)
        printf("%3d %6.1f\n", fahr, (5.0/9.0)*(fahr-32));
    return 0;
}
```
