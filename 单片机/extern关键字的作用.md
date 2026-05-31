## 简介
在单文件中编写代码如果我们需要在多个函数中同时操作一个变量，我们可以在所有函数外部定义一个全局变量，这样一来所有的函数内部都可以对这个全局变量进行修改。但是如果我们想要在不同文件的不同函数之间共享同一个变量又应该怎么办呢，这个时候我们就可以尝试使用`extern`关键字，`extern`关键字主要用于跨文件共享全局变量，它的核心作用是告诉编译器：“这个变量或函数已经在其它地方被定义过了，我们此时可以直接使用它，不需要再为它重新分配内存

## 应用场景
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