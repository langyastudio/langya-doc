> 本文来自JavaGuide，郎涯进行简单排版与补充



## github 访问速度慢

> 安装代理软件，提供翻墙功能

### HTTP 代理

使用 IP 127.0.0.1 + 端口8080进行代理

```bash
git config --global http.https://github.com.proxy http://127.0.0.1:8080
git config --global https.https://github.com.proxy http://127.0.0.1:8080
```

第一个是设置 Git在采用HTTP连接时的代理地址

第二个是设置 Git 在采用HTTPS连接时的代理地址



### SOCKS 5 代理

使用 IP 127.0.0.1 + 端口4781进行代理

```bash
git config --global http.https://github.com.proxy socks5://127.0.0.1:4781
git config --global https.https://github.com.proxy socks5://127.0.0.1:4781

git config --global http.https://raw.githubusercontent.com.proxy socks5://127.0.0.1:4781
git config --global https.https://raw.githubusercontent.com.proxy socks5://127.0.0.1:4781
```



### Git 取消代理

```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```

> 建议使用Git安装包安装Git客户端



### push 数据失败

可以通过如下步骤解决：

- 升级 Git 版本

- github 现在只能使用 personal access token 登录

  把原本的 `https://github.com/<USERNAME>/<REPO>.git` 修改成 `https://<your_token>@github.com/<USERNAME>/<REPO>.git`

  ![image-20220616212625319](https://img-note.langyastudio.com/202206162126397.png?x-oss-process=style/watermark)

  

## Github 简历

通过 [https://resume.github.io/](https://resume.github.io/) 这个网站你可以一键生成一个在线的 Github 简历。

当时我参加的校招的时候，个人信息那里就放了一个在线的 Github 简历。我觉得这样会让面试官感觉你是一个内行，会提高一些印象分。

但是，如果你的 Github 没有什么项目的话还是不要放在简历里面了。生成后的效果如下图所示。

![Github简历](https://img-note.langyastudio.com/202111171122172.png?x-oss-process=style/watermark)



## 个性化 Github 首页

Github 目前支持在个人主页自定义展示一些内容。展示效果如下图所示。

![个性化首页展示效果](https://img-note.langyastudio.com/202111171122282.png?x-oss-process=style/watermark)

想要做到这样非常简单，你只需要创建一个和你的 Github 账户同名的仓库，然后自定义`README.md`的内容即可。

展示在你主页的自定义内容就是`README.md`的内容（_不会 Markdown 语法的小伙伴自行面壁 5 分钟_）。

![创建一个和你的Github账户同名的仓库](https://img-note.langyastudio.com/202111171122067.png?x-oss-process=style/watermark)



这个也是可以玩出花来的！比如说：通过 [github-readme-stats](https://hellogithub.com/periodical/statistics/click/?target=https://github.com/anuraghazra/github-readme-stats) 这个开源项目，你可以 README 中展示动态生成的 GitHub 统计信息。展示效果如下图所示。

![通过github-readme-stats动态生成GitHub统计信息 ](https://img-note.langyastudio.com/202111171122883.png?x-oss-process=style/watermark)

关于个性化首页这个就不多提了，感兴趣的小伙伴自行研究一下。



## 自定义项目徽章

你在 Github 上看到的项目徽章都是通过 [https://shields.io/](https://shields.io/) 这个网站生成的。我的 JavaGuide 这个项目的徽章如下图所示。

![项目徽章](https://img-note.langyastudio.com/202111171122340.png?x-oss-process=style/watermark)

并且，你不光可以生成静态徽章，shield.io 还可以动态读取你项目的状态并生成对应的徽章。

![自定义项目徽章](https://img-note.langyastudio.com/202111171118270.png?x-oss-process=style/watermark)

生成的描述项目状态的徽章效果如下图所示。

![描述项目状态的徽章](https://img-note.langyastudio.com/202111171118675.png?x-oss-process=style/watermark)



## Github 表情

![Github表情](https://img-note.langyastudio.com/202111171118674.png?x-oss-process=style/watermark)

如果你想要在 Github 使用表情的话，可以在这里找找 ：[www.webfx.com/tools/emoji-cheat-sheet/ ](www.webfx.com/tools/emoji-cheat-sheet/)。

![在线Github表情](https://img-note.langyastudio.com/202111171118668.png?x-oss-process=style/watermark)



## 高效阅读 Github 源代码

Github 前段时间推出的 Codespaces 可以提供类似 VS Code 的在线 IDE，不过目前还没有完全开发使用。

简单介绍几种我最常用的阅读 Github 项目源代码的方式。

### Chrome 插件 Octotree

这个已经老生常谈了，是我最喜欢的一种方式。使用了 Octotree 之后网页侧边栏会按照树形结构展示项目，为我们带来 IDE 般的阅读源代码的感受。

![Chrome插件Octotree](https://img-note.langyastudio.com/202111171119550.png?x-oss-process=style/watermark)



### Chrome 插件 SourceGraph

我不想将项目 clone 到本地的时候一般就会使用这种方式来阅读项目源代码。SourceGraph 不仅可以让我们在 Github 优雅的查看代码，它还支持一些骚操作，比如：类之间的跳转、代码搜索等功能。

当你下载了这个插件之后，你的项目主页会多出一个小图标如下图所示。点击这个小图标即可在线阅读项目源代码。

![](https://img-note.langyastudio.com/202111171119919.png?x-oss-process=style/watermark)

使用 SourceGraph 阅读代码的就像下面这样，同样是树形结构展示代码，但是我个人感觉没有 Octotree 的手感舒服。不过，SourceGraph 内置了很多插件，而且还支持类之间的跳转！

![](https://img-note.langyastudio.com/202111171119238.png?x-oss-process=style/watermark)



### 克隆项目到本地

先把项目克隆到本地，然后使用自己喜欢的 IDE 来阅读。可以说是最酸爽的方式了！

如果你想要深入了解某个项目的话，首选这种方式。一个`git clone` 就完事了。



### 其他

如果你要看的是前端项目的话，还可以考虑使用 [https://stackblitz.com/](https://stackblitz.com/) 这个网站。

这个网站会提供一个类似 VS Code 的在线 IDE。



## 扩展 Github 的功能

**Enhanced GitHub** 可以让你的 Github 更好用。这个 Chrome 插件可以可视化你的 Github 仓库大小，每个文件的大小并且可以让你快速下载单个文件。

![](https://img-note.langyastudio.com/202111171119343.png?x-oss-process=style/watermark)



## Markdown 文件生成目录

如果你想为 Github 上的 Markdown 文件生成目录的话，通过 VS Code 的 **Markdown Preview Enhanced** 这个插件就可以了。

生成的目录效果如下图所示。你直接点击目录中的链接即可跳转到文章对应的位置，可以优化阅读体验。

![](<https://img-note.langyastudio.com/202111171119227.png?x-oss-process=style/watermark>)

