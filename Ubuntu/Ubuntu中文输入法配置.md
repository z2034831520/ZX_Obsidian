## 问题背景
在`ubuntu24.04`中系统默认给我们配置的输入法是`ibus`(`Intelligent Input Bus`)，这套输入法和`GNOME`桌面的集成度很高，，且和`Wayland`的兼容性也比较好，极少出现候选框偏移等问题。如果我们在安装`ubunut`时将系统的默认语言选择为中文，那么系统就会为我们配置好`ibus`输入法

尽管`ibus`是系统默认且稳定性极强的框架，但是在中文输入下的体验感却不是很好，使用起来会觉得有些不跟手。接下来我就会分享一下如何解决这个问题

## 解决方案
搜狗输入法想必大家都不陌生，作为国内知名度较高的一款输入法在很多中国用户会选择在`Windows`使用搜狗输入法。其实搜狗输入法也是有`Linux`版本的，但是有点古早，对于较新版本的`ubunut`系统兼容性可能不是很好
![](assets/Ubuntu中文输入法配置/file-20260809174657136.png)
官网上看从`ubuntu16.04`支持到`ubunut20.10`，对于我们现在使用的`24.04`版本来说有点古老。因此我们在这里会更倾向于用另一款较新的输入法框架：`Fcitx`(`Free Chinese input Toy for X`)，这是`Linux`下最著名的输入法框架之一，在`Ubuntu24.04`中如果我们在系统安装时选择了中文作为系统语言，并且勾选了下载默认集合等选项，系统同样会为我们自动下载`Fcitx5`输入法框架
![](assets/Ubuntu中文输入法配置/file-20260809175345436.png)
该输入法框架的中文输入体验就比`ibus`要好很多了，且可扩展性也比较强，我们可以配置它的外观界面以及词库等，正是由于这些可定制化的特性使得该输入法的上限变的很高，在本期内容我就会向你展示如何在`Ubuntu24.04`下配置一套操作体验极好的中文输入法

## 实操

### 基础配置
由于有些人的系统中默认没有安装`Fcitx5`输入法框架，所以在这里提供一下安装该输入法的命令
```bash
sudo apt update
sudo apt install fcitx5 fcitx5-chinese-addons fcitx5-frontend-gtk3 fcitx5-frontend-gtk4 fcitx5-frontend-qt5 fcitx5-config-qt
```
安装完成之后我们需要将系统默认的输入法更改为`Fcitx5`，对应的命令如下：
```bash
im-config -n fcitx5
```
配置玩默认输入法之后我们需要先进行最底层的配置，我们在终端中输入如下命令可以直接唤起`Fcitx5`的配置界面
```bash
fcitx5-configtool
```
在配置界面中我们需要将左侧面板中原先默认的几个选项都删除，删除方法很简单双击目标选项栏即可删除，将默认选项全部删除完毕之后我们接下来还需要添加图中红色方框内的选项
![](assets/Ubuntu中文输入法配置/file-20260809195953106.png)
在选择之前我们需要取消勾选底部的仅显示当前语言选项，否则我们就只能看到部分选项
![](assets/Ubuntu中文输入法配置/file-20260809201537464.png)
我们需要选择的选项为以下两个，其中第一个是键盘布局，第二个是输入法
![](assets/Ubuntu中文输入法配置/file-20260809201826230.png)
配置完成之后我们应该就可以实现中英文切换了，全局选项中的选项大多都不需要二次设置，除非你自己有特殊的习惯或需求
![](assets/Ubuntu中文输入法配置/file-20260809202422132.png)
如果不出意外的话现在你已经能够通过`shift`按键来切换中英文了

### UI配置
这部分内容大家可以选看，如果觉得系统默认的输入法`UI`界面不是很美观那么接下来我们可以更改输入法的外观，更改的方式是借助`Github`上的开源项目，项目地址[项目链接，点击直达](https://github.com/travisbikkle/fcitx5-themes)，我们打开项目主页可以看到作者给我们写好的`readme`文件，我们按照上面的操作一步步进行即可，首先是将该项目拷贝到本地
```bash
git clone https://github.com/thep0y/fcitx5-themes.git
```
通过`Git`将项目拉下来之后我们需要创建存放`UI`主题配置文件的文件夹，
```bash
mkdir -p ~/.local/share/fcitx5/themes
```
创建完成对应的文件夹之后我们先切换到拷贝项目的文件夹下，通过`ls`命令观察拉取结果：
```bash
❯ ls
autumn  install.sh  macOS-dark-png   README.en.md  README.md  transparent-green
green   LICENSE     macOS-light      README.jp.md  spring     winter
images  macOS-dark  macOS-light-png  README.kr.md  summer

fcitx5-themes on git:main 
❯ 

```
如果我们在文件夹下看到了上述内容就说明文件目录切换成功，接下来我们需要把我们想要的`UI`主题的文件夹直接拷贝到我们刚才创建的`UI`主题配置文件的文件夹下，这里我以`macOS-dark`主题作为演示，先通过
```bash
cp -a macOS-dark ~/.local/share/fcitx5/themes/
```
命令拷贝文件夹，拷贝结束之后我们通过
```bash
ls ~/.local/share/fcitx5/themes/macOS-light
```
命令检查拷贝结果是否完整，输出结果中应该包含
```bash
theme.conf
panel.svg
highlight.svg
```
接下来我们需要修改配置文件的内容
```bash
vim ~/.config/fcitx5/conf/classicui.conf
```
如果没有生效我们可以执行
```bash
fcitx5-remote -r
```
命令来刷新输入法状态