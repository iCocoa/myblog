---
title: iOS block
date: 2014-09-20 20:18:23
tags: iOS
---

`blocks` 是 C 语言的扩充功能。blocks 是带有自动变量（局部变量）的匿名函数。

### 截获自动变量

```		
int main()
{
	int dmy = 256;
	int val = 10;
	const char *fmt = "val = %d\n";
	void (^blk)(void) = ^{
		printf(fmt,val);
	};
	
	val = 2;
	fmt = "These value were changed. val = %d\n";
	
	blk();
	
	return 0;
}
```
		
结果：`val = 10`
		
分析：block 语法的表达式使用的是它之前声明的自动变量 fmt 和 val。block 表达式截获所使用的自动变量的值为瞬间值。因为 block 表达式保存了自动变量的值（截获），所以在执行 block 语法后，即使改写了 block 中使用的自动变量的值也不会影响 block 执行的结果。

需要在 block 中修改一个变量的值，需要使用 \_\_block 说明符。
		
### block 的实质
block 实际上是作为极普通的 C 语言源代码来处理的。通过支持 block 的编译器，含有 block 语法的源代码转换为一般 C 语言编译器能够处理的源代码，并作为极为普通的 C 语言代码被编译。
			
`clang -rewrite-objc sourceFileName`

通过这个命令可以将含有 block 语法的源代码转换为 C++ 代码。
通过观察，Block 转换为 Block 的结构体类型的自动变量，\_\_block 变量转换为 \_\_block 变量的结构体类型的自动变量（即栈上生成的该结构体的实例）。

表 1-1 Block 与 \_\_block 变量的实质

|	名称        	 | 实质    		| 
| :-------------: | :-------------:| 
| Block  		| 栈上 Block 的结构体实例 | 
| \_\_block 变量    | 栈上 \_\_block 变量的结构体实例 | 

表 1-2 Block 的类

|	类        	   | 设置对象的存储域    		| 
| :-------------:  | :-------------:       | 
| \_NSConcreteStackBlock  		| 栈   | 
| \_NSConcreteGlobalBlock      | 程序的数据区域(.data区) | 
| \_NSConcreteMallocBlcok      | 堆 | 

Block 为 \_NSConcreteGlobalBlock 类对象的情况

* 记述全局变量的地方有 Block 语法时
* Block 语法的表达式中不使用应截获的自动变量时

除了以上两种情况 block 语法生成的 block 为 \_NSConcreteStackBlock 类对象，且设置在栈上。

* 将 block 配置在堆上的 \_NSConcreteMallocBlock 类在何时使用呢？
* block 超出变量作用域可存在的原因是？
* \_\_block 变量用结构体成员变量 \_\_forwarding 存在的原因是？


Blocks 提供了将 Block 和 \_\_block 变量从栈上复制到堆上的方法，这样，即使 Block 语法记述的变量作用域结束，堆上的 Block 还可以继续存在。

### 什么时候栈上的 Block 会复制到堆

* 调用 Block 的 copy 实例方法时
* Block 作为函数返回值返回时
* 将 Block 赋值给附有 \_\_strong 修饰符、id 类型的类或 Block 类型成员变量时
* 在方法名中含有 usingBlock 的 Cocoa 框架方法或 Grand Central Dispatch 的 API 中传递 Block 时

堆上的 Block 被废弃时会调用 dispose 函数。

只有调用 \_Block\_copy 函数才能持有截获的附有 \_\_strong 修饰符的对象类型的自动变量值。当需要在 Block 中使用对象类型自动变量时，除以下情形，推荐调用 Block 的 copy 方法。

* Block 作为函数返回值返回时
* 将 Block 赋值给附有 \_\_strong 修饰符、id 类型的类或 Block 类型成员变量时
* 在方法名中含有 usingBlock 的 Cocoa 框架方法或 Grand Central Dispatch 的 API 中传递 Block 时

 
