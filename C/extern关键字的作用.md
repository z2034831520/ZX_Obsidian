## 简介
在单文件中编写代码如果我们需要在多个函数中同时操作一个变量，我们可以在所有函数外部定义一个全局变量，这样一来所有的函数内部都可以对这个全局变量进行修改。但是如果我们想要在不同文件的不同函数之间共享同一个变量又应该怎么办呢，这个时候我们就可以尝试使用`extern`关键字，`extern`关键字主要用于跨文件共享全局变量，它的核心作用是告诉编译器：“这个变量或函数已经在其它地方被定义过了，我们此时可以直接使用它，不需要再为它重新分配内存，这就需要去先了解[定义和声明](定义和声明.md)的区别

## 应用场景

### 典型应用场景
`extern`关键字的典型应用场景就是多文件编程，假设我们现在正在编写一个包含多个源文件的项目，我们想要在`main.c`中访问`module.c`中的全局变量，此时我们就可以借助`extern`关键字来实现

文件1：`module.c`(变量的诞生地)
我们在这里定义全局变量并为其分配内存
```c
#include <stdio.h>

int global_count = 100;

void print_count() {
	printf("count = %d\n", global_count);
}
```

文件2：`main.c`使用变量
```c
#include <stdio.h>

extern int global_count;
int main()
{
	printf("global count = %d\n", global_count);
	global_count += 50;
	
}
```

当我们编写完上述文件并开始进行编译时，链接器会将`main.c`中的`global_count`引用绑定到`module.c`中定义的那个变量上。此时在两个文件中都可以对这个变量进行实时修改，修改后的结果两个文件也可以共享。

### 进阶应用场景
上面我们演示了`extern`关键字的基础用法，但是在实际的开发中我们往往不会使用上面的这种方式去使用`extern`关键字。更标准的做法是将`extern`的声明统一放在头文件中，然后让需要使用的文件去引用这个头文件

1. `globals.h`（声明）
	```c
#ifndef GLOBALS_H
#define GLOBALS_H

	// 对外暴露的接口和变量声明
	extern int shared_data;

#endif
	```
2. `globals.c`（定义）
	```c
	#include "globals.h"

	// 真正的定义，整个项目只出现一次
	int shared_data = 42;
	```
3. `main.c`（使用）
	```c
	#include <stdio.h>
	#include "globals.h" // 引入声明

	int main() 
	{
	    printf("Shared data: %d\n", shared_data);
	    return 0;
	}
	```
	这样做的好处是：一旦变量的类型需要修改，我们就只需要修改`globals.c`和`globals.h`，而不需要去所有使用了该变量的`.c`文件中去挨个修改声明，这样也可以减少代码体积。

## 注意事项
1. 带初始化的`extern`会退化成定义：
	如果我们在声明时进行了初始化操作：`extern int a = 10;`，编译器就会自动忽略`extern`关键字，直接将其视为定义并分配内存，如果其它文件里也定义了该变量，在编译链接时就会报”多重定义（`Multiple definition`）",错误。

2. 避免滥用全局变量：
	虽然`extern`可以在不同文件间进行数据共享，但是全局变量会破坏模块的封装性






