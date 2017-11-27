---
title: Typeof()
date: 2014-07-27 20:56:50
tags: C
---
最近看到项目中，有人这样写 `__weak __typeof__(self) wself = self;`。我查了一下资料，总结一下。

typeof关键字是C语言中的一个新扩展,在linux内核中应用非常广泛。

### 说明
typeof()的参数可以是表达式或一种类型。返回的结果是一种类型。

* 表达式

`typeof(x[0](1));`

这里假设 x 是一个函数指针数组，这样就可以得到这个函数返回值的类型了。如果将 typeof 用于表达式，则该表达式不会执行。只会得到该表达式的类型。以下示例声明了 int 类型的 var 变量，因为表达式 foo() 是 int 类型的。由于表达式不会被执行，所以不会调用 foo 函数。

```
extern int foo();
typeof(foo()) var;
```

* 一种类型

`typeof(int *) a,b;` 等价于：`int *a,*b;`	

### 例子

1. 把 y 定义成 x 指向的数据类型：
           
   `typeof(*x) y;`
           
2. 把 y 定义成 x 指向数据类型的数组：
   
   `typeof(*x) y[4];`
   
3. 把 y 定义成一个字符指针数组：
   
   `typeof(typeof(char *)[4]) y;`
   
   这与下面的定义等价：
   `char *y[4];`

4. `typeof(int *) p1,p2; `等价于

   `int *p1, *p2;`

5. `typeof(int) *p3,p4;`等价于
   
   `int *p3, p4;`

6. `typeof(int [10]) a1, a2;`等价于
    
   `int a1[10], a2[10];`

### 特殊情况

typeof 构造中的类型名不能包含存储类说明符，如 extern 或 static。不过允许包含类型限定符，如 const 或 volatile。例如，下列代码是无效的，因为它在 typeof 构造中声明了 extern：

`typeof(extern int) a;`


    	
以上是typeof()总结。