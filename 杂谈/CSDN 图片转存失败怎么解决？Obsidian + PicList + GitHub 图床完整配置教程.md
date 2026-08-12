![](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/file-20260812101124110.png)## 引言
不知道有没有像我一样经常写博客的同学：当我们花费一段时间写出了一篇还算满意的文章，准备把它发布到网页平台时，却发现本地文档中的图片全部无法正常显示，这里以 `CSDN` 为例。我们打开文章编写界面，切换到 Markdown 编辑器，然后把在本地写好的文章复制进去，图片位置却出现了“图片转存失败”，如下图所示：
![](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/file-20260811202852156.png)
为了让图片恢复正常，我们往往只能删除失败的图片链接，再把本地图片一张张复制到网页编辑器中。如果一篇文章中只有两三张插图那还可以接收，但是如果文章中有十几张插图那就让人比较红温了。显然，这并不是一个明智的解决方案。本篇文章将介绍如何更高效的解决这个问题

借助 `GitHub + PicList + jsDelivr + Obsidian`，把本地图片自动上传为网页可以访问的图片，并让 Markdown 中的本地路径自动替换为公网链接。

## 问题成因
首先需要搞明白一件事：为什么同一张图片在本地可以正常显示，复制到网页之后却无法显示？关键不在图片本身，而在 Markdown 中保存的图片地址。同一张图片在本地和网页中可能分别使用下面两种写法：
```markdown
<!-- 本地相对路径 -->
![](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/file-20260811202852156.png)

<!-- 公网图片链接 -->
![](https://example.com/images/file.png)
```
第一种写法中的 `assets/...`是相对于当前笔记的本地路径。`Obsidian`可以根据这个路径在我们的电脑中找到图片，因此能够正常显示；但是把 `Markdown`复制到`CSDN`后，`CSDN`的服务器无法访问我们电脑上的文件夹，自然也就找不到这张图片。第二种写法使用的是公网链接，只要这个地址不需要登录、没有过期，并且能够在普通浏览器中直接打开，网页平台才能够读取并显示图片

所以我们解决问题的思路为：
1. 把本地图片上传到能够公开访问的存储位置，也就是通常所说的“图床”
2. 把 Markdown 中的本地相对路径替换成图床返回的公网链接

## 方案概览
本篇文章使用的完整链路如下：
```text
本地图片
   ↓
Obsidian 图片自动上传插件
   ↓
PicList 本地上传 API
   ↓
GitHub 公开仓库
   ↓
jsDelivr 公网访问地址
   ↓
Markdown 公网图片链接
   ↓
CSDN 等网页平台
```
其中各部分的职责如下：

- `GitHub`：保存图片文件
- `PicList`：接收图片、上传到`GitHub`，并生成公网链接
- `jsDelivr`：把`GitHub`文件转换为更适合网页访问的`CDN`链接
- `Obsidian` 图片自动上传插件：调用`PicList`，并自动替换笔记中的图片路径

## 实操

### 一、创建 GitHub 图床仓库
首先登录 GitHub，新建一个专门存放图片的仓库，仓库名随意，下面是一个参考：
```text
obsidian-images-bed
```
仓库需要设置为 `Public`。这是因为网页平台无法读取私有仓库中的图片。默认分支建议保持为 `main`，后续 `PicList`中填写的分支名必须与这里一致，参考配置如下图所示
![](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/file-20260811224559378.png)
建议让图片统一存放在仓库的 `images/`目录中，避免所有文件堆放在仓库根目录，那样会显得比较凌乱
### 二、创建 GitHub 访问令牌
`PicList`需要通过 `GitHub API`上传图片，因此需要创建一个密钥来授予相应的访问权限，创建的过程也很简单

1. 单击右上角的个人头像，在随后弹出的窗口中选择`Settings`：
	![](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/file-20260811224848596.png)
2. 在新界面中下拉选择`Developer settings`：
	![](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/file-20260811225031087.png)
3. 在新界面中选择`Personal access tokens`然后选择下方的`Tokens classic`，在新界面中点击生成新秘钥，然后选择下方的经典密钥：
	![](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/file-20260811225144865.png)
4. 然后进行一下验证
	![](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/file-20260811225407282.png)
5. 在`Token`配置界面中我们只需呀修改一下名称和权限，名称随意，权限只需要勾选红色方框中的`repo`即可
	![](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/file-20260811225541630.png)
6. 然后点击底部的生成密钥即可
	![](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/file-20260811225642713.png)
7. 接下来会弹出`Token`，复制之后保存下来，接下来要用

### 三、安装 PicList
`PicList`是一个开源的项目，目前托管在`Github`上，项目链接为：[项目链接，点击直达](https://github.com/Kuingsmile/PicList)在项目主页中我们可以看到多种下载方式，如果你想省事一点可以直接执行对应的安装指令进行一键下载。如果你想更改文件的下载路径那么你需要先下载对应的`.exe`安装程序，然后再进行下载，在下载的引导程序中我们可以选择软件的安装位置
![](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/file-20260812092025845.png)

### 四、配置 GitHub 图床
下载完成之后接下来我们需要对图床进行配置,打开软件侧边栏中的图床选项，然后在下面选择`Github`，如果是第一次打开可能会有一个默认配置，我们直接点击默认配置右上角的铅笔图标修改配置即可。或者你也可以新建一个配置
![](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/file-20260812092238496.png)
在修改界面中我们需要注意的几点是仓库名我们可以到仓库主页中进行寻找
![](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/file-20260812092854455.png)
如图所示仓库名就是**用户名/仓库名**的格式
分支我们选择`main`分支，设定`Token`我们填写**创建 GitHub 访问令牌**这一步中生成的`Token`，存储路径我们可以填写一个自己喜欢的名字，默认情况下该名称是在根目录下创建的，当前我们填写的是`images/`，那么我们后续上传的照片都会被保存在`Github`项目的`images`目录下
![](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/file-20260812092736705.png)
还有就是设置自定义域名，自定义域名的作用是`CDN`加速，可以让国内的一些网站加速访问，如`CSDN`等，否则的话配置之后可能还是会显示报错

如果`PicList`没有配置自定义域名，`GitHub`图床通常会返回 `raw.githubusercontent.com` 地址。这个地址在部分网络环境中访问不稳定，`CSDN`的服务器也可能无法抓取，从而出现：
```text
外链图片转存失败，源站可能有防盗链机制
```
因此我们需要在“自定义域名”中填写：
```text
https://cdn.jsdelivr.net/gh/GitHub用户名/仓库名@分支名/
```
例如：
```text
https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/
```

### 六、开启 PicList 上传 API
`Obsidian`插件需要通过本地接口把图片交给 `PicList`，打开软件的设置界面，下拉打开上传`API`服务设置：
![](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/file-20260812101126902.png)
开启上传 API，然后进行如下配置：
![](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/file-20260812101234491.png)
这里注意`Web`服务最好不要开启，否则两个服务之间可能会出现端口占用的问题
这里需要注意的一点是后续上传图片时`PicList`必须一直在后台运行

### 七、配置 Obsidian 自动上传
打开 Obsidian 的第三方插件市场，搜索并安装图片自动上传插件，例如 `Image auto upload`
![](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/file-20260812101506491.png)
安装完成后启用插件，并在插件设置中填写`PicList`上传接口：
![](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/file-20260812101536657.png)

```text
http://127.0.0.1:36677/upload
```
如果你勾选了最上方的剪切板自动上传，那么当你把图片粘贴到笔记中后就会自动向云端进行上传，我建议先不要勾选这个选项，因为上传比较耗时，建议后续手动上传哪些需要上传的照片，默认情况下还是使用本地路径
上传的方法也很简答，右键图片，然后点击`upload`即可
![](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/file-20260812102642968.png)
## 总结
操作完成之后想必你就能否解决图片上传的问题了，在本次教程中我们使用的是`Github`作为图床，这对网络环境要求比较高，且私密性较差。如果你有大批量的图片需要处理或者有隐私相关的需求这里推荐使用一些国内的存储服务，如腾讯云 COS、阿里云 OSS 等

## 参考资料
- [PicList 官方网站](https://piclist.cn/)
- [PicList GitHub 图床配置说明](https://piclist.cn/configure#github图床)
- [GitHub Personal Access Token 官方文档](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)