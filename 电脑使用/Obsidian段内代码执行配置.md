## 引言
我们在进行日常笔记的编写或者查看`AI`以及他人分享的`markdown`笔记时，其中往往会出现代码块，代码块中的代码往往会有对应的高亮语法显示。我们在编写自己的笔记时可能会想验证一下自己当前的代码是否正确，运行结果是什么，观看别人分享的笔记时同样会有对应的想法。对于一些动辄几百行的复杂代码，我们可以直接将代码拷贝到IDE中直接运行并查看结果，但是有些简单的代码，50行以内的代码，我们自己虽然能够看懂代码在干什么，但是却无法百分百确保代码能够正常运行时，可能还是需要将代码放入IDE中观察输出结果，但是这样就会导致效率降低。如果我们的代码可以直接在笔记内执行呢？

接下来我就要介绍一下如何在`obsidian`中实现段内代码的运行，提高你编写笔记时的效率，减少观看他人笔记时的困惑

## 插件安装
想要实现段内代码运行我们需要安装`Execute Code`插件
![Pasted image 20261010195028](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/Pasted%20image%2020261010195028.png)插件安装完成之后我们进入插件配置界面
![Pasted image 20261010195129](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/Pasted%20image%2020261010195129.png)
在插件配置界面中我们可以看到的几个通用配置作用为：
- Timeout：超时时间，默认配置为10s，如果代码在运行后10s内没有输出结果，那么就判定代码运行失败，防止卡死`obsidian`对于我们轻量级代码的运行场景来说10s的运行时间已经足够了
- `Allow input`：开启后运行代码时会出现输入框，当我们运行的代码中需要用户输入参数时可以开启此选项，例如当`C++`代码中出现`cin`时
- `WSL Mode`：这里需要介绍的是`WSL`是`windows`的`Linux`子系统，如果你安装并配置过`WSL`那么可以开启此选项，如果没有安装或配置过那么这个选项保持关闭即可
- `[Experimental] Persistent Output`：持续输出功能，开启此功能后，代码运行结果会直接以文本的形式保存在当前的`Markdown`笔记文件中而不是临时显示
上面介绍的都是一些通用设置，接下来我会针对不同的编程语言来介绍一下对应的详细配置

## C/C++
我们在插件配置界面中观察到`C`语言设置中推荐你我们使用`gcc/Cling`，但是在`C++`中就推荐我们使用`Cling`了，所以为了能够同时满足两种语言的需求，我们这里选择`Cling`作为编译器
这里先简单介绍一下`Cling`，它和我们熟知的`GCC`编译器的主要区别是，`GCC`属于传统的静态编译器，而`Cling`是动态的交互式解释器，因此`Cling`的灵活度更高
如果你没有在自己的电脑里安装过`Cling`，那么接下来我先带着你安装一下`Cling`

### 安装Cling
这里我们选择通过**Miniforge**来安装`Cling`，该项目的`Github`链接为[项目下载界面](https://github.com/conda-forge/miniforge/releases)在项目下载界面中我们选择适配自己当前操作系统的版本安装包，随后双击安装包即可进行安装
![Pasted image 20261010210957](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/Pasted%20image%2020261010210957.png)
在下载界面中对应的程序安装路径我们可以自定义，只要后续进行配置时能找到程序的安装路径即可
![Pasted image 20261010211101](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/Pasted%20image%2020261010211101.png)
在图示配置界面中我们可以保持图中所示配置，第一个和第四个选项勾选与否影响不大，第一个选项是创建启动图标，最后一个选项是减少空间占用

安装完成之后我们在开始界面中搜索`Miniforge`
![Pasted image 20261010213314](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/Pasted%20image%2020261010213314.png)打开之后我们在其中执行以下四条指令
```bash
conda create -n cling-lab -c conda-forge cling
conda activate cling-lab
where cling
cling --help
```
指令执行完毕之后我们复制以下`where cling`指令的运行结果，该指令的输出结果是`cling.exe`程序的安装路径

随后我们在插件的配置界面中将对应的路径输入`Cling Path`对应的参数框中，此时我们在obsidian中随便打开一个笔记，在其中创建一段C语言代码，随后按下键盘上的`ctrl+E`快捷键，即可进入视图模式，在该模式下，对应的代码段下面会出现`run`选项，我们点击`run`即可观察到对应的运行结果
![Pasted image 20261010214102](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/Pasted%20image%2020261010214102.png)
`C++`的配置与C语言保持一致即可
![Pasted image 20261010214703](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/Pasted%20image%2020261010214703.png)
这里还需要介绍一个配置就是`Use main function`选项，该选项的作用是控制运行`C++`时是否需要以`main()`函数作为程序入口
由于该插件在底层使用的是`Cling`，它支持类似`Python`中断那样的交互式运行方式，该选项的开关决定了解释器如何解析你的代码

当我们关闭该选项时，对应的解释器就会像运行`python`代码一样一句一句的翻译你的代码，然后在给出对应的运行结果，我们可以在代码段中直接编写裸句并执行，这比较契合我们日常的快速笔记或者临时验证某个代码片段是否正确，代码片段可以像下面这样：
```cpp
#include <iostream>

int a = 10;
int b = 20;
std::cout << "Sum: " << (a + b) << "\n";
```
在此配置下运行标准普通代码就会出现报错
![Pasted image 20261010220109](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/Pasted%20image%2020261010220109.png)

当我们开启了对应的选项之后插件就会将代码段中的代码视作一个完整的程序，回去寻找`main`函数，如果没有`main`函数那么就会直接报错，因此代码块中的代码就必须严格遵守原版的代码格式，代码风格如下：
```cpp
#include <iostream>

int main() 
{
    std::cout << "Hello, World!\n";
    return 0;
}
```
在此配置下运行上面的段代码就会出现报错
![Pasted image 20261010220217](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/Pasted%20image%2020261010220217.png)
## python
如果之前你在电脑中安装`python`解释器时将其安装在了默认目录下那么你就不需要修改对应的插件配置，直接点击对应的`run`选项就可以看到对应的运行结果了。否则就需要像配置C语言解释器那样将你电脑中真正的`python`解释器路径更新一下，更新之后就能正常使用了
![Pasted image 20261010214421](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/Pasted%20image%2020261010214421.png)
## Java
对于`Java`代码，只要对应的Java解释器版本满足高于Java11即可直接在软件中运行段内代码块，无需进行二次配置，像下面的代码直接点击运行就能看到结果
```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```


## JavaScript
对于JavaScript代码也是同理，如果你的电脑中已经安装了对应的Node.js环境，那么插件在运行代码的时候就会自动寻找到对应的运行环境，让代码直接运行，示例代码如下，点击run选项即可直接运行
```javascript
function main() {
    console.log("Hello, World!");
}

main();
```

## 总结
至此就介绍完了如何在obsidian软件中运行段内代码，希望这个功能能够改善你的笔记阅读和编写体验