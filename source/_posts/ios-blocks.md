---
title: ios block
date: 2014-09-20 20:18:23
tags: iOS
---

**blocks**是C语言的扩充功能。blocks是带有自动变量（局部变量）的匿名函数。

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
		
结果：`val = 2`
		
分析：block语法的表达式使用的是它之前声明的自动变量fmt和val。block表达式截获所使用的自动变量的值为瞬间值。因为block表达式保存了自动变量的值（截获），所以在执行block语法后，即使改写了block中使用的自动变量的值也不会影响block执行的结果。

需要在block中修改一个变量的值，需要使用\_\_block说明符。
		
### block的实质
block实际上是作为极普通的C语言源代码来处理的。通过支持block的编译器，含有block语法的源代码转换为一般C语言编译器能够处理的源代码，并作为极为普通的C语言代码被编译。
			
`clang -rewrite-objc sourceFileName`

通过这个命令可以将含有block语法的源代码转换为C++代码。
通过观察，Block转换为Block的结构体类型的自动变量，\_\_block变量转换为\_\_block变量的结构体类型的自动变量（即栈上生成的该结构体的实例）。

表 1-1 Block与\_\_block变量的实质

|	名称        	 | 实质    		| 
| :-------------: | :-------------:| 
| Block  		| 栈上Block的结构体实例 | 
| \_\_block变量    | 栈上\_\_block变量的结构体实例 | 

表 1-2 Block的类

|	类        	   | 设置对象的存储域    		| 
| :-------------:  | :-------------:       | 
| \_NSConcreteStackBlock  		| 栈   | 
| \_NSConcreteGlobalBlock      | 程序的数据区域(.data区) | 
| \_NSConcreteMallocBlcok      | 堆 | 

Block为\_NSConcreteGlobalBlock类对象的情况

* 记述全局变量的地方有Block语法时
* Block语法的表达式中不使用应截获的自动变量时

除了以上两种情况block语法生成的block为\_NSConcreteStackBlock类对象，且设置在栈上。

* 将block配置在堆上的_NSConcreteMallocBlock类在何时使用呢？
* block超出变量作用域可存在的原因是？
* \_\_block变量用结构体成员变量\_\_forwarding存在的原因是？


Blocks提供了将Block和\_\_block变量从栈上复制到堆上的方法，这样，即使Block语法记述的变量作用域结束，堆上的Block还可以继续存在。

### 什么时候栈上的Block会复制到堆

* 调用Block的copy实例方法时
* Block作为函数返回值返回时
* 将Block赋值给附有\_\_strong修饰符、id类型的类或Block类型成员变量时
* 在方法名中含有usingBlock的Cocoa框架方法或Grand Central Dispatch的API中传递Block时

堆上的Block被废弃时会调用dispose函数。

只有调用\_Block\_copy函数才能持有截获的附有\_\_strong修饰符的对象类型的自动变量值。当需要在Block中使用对象类型自动变量时，除以下情形，推荐调用Block的copy方法。

* Block作为函数返回值返回时
* 将Block赋值给附有\_\_strong修饰符、id类型的类或Block类型成员变量时
* 在方法名中含有usingBlock的Cocoa框架方法或Grand Central Dispatch的API中传递Block时

 
