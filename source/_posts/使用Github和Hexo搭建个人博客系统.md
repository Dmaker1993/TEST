---
title: 使用Github和Hexo搭建个人博客系统
top: false
date: 2021-12-27 11:20:19
tags:
    - 博客系统
    - 经验
categories:
    - 经验分享
---
# 利用Github和Hexo搭建个人博客

作为一名开发人员，写技术博客既是一种对知识的巩固，也是一种分享精神。所以有一个自己的技术博客，对于我们技术人员来说，是很nice的一件事。

## 1、条件

​	如果要使用github搭建自己的博客系统，需要有一个 *Github账号* ，我们可以注册一个账号。其次需要一个 *node开发环境（nodejs、npm）*，因为 *Hexo* 框架需要使用node环境搭建。

- github账号
- node、npm环境（省略）
- 本地安装了git（省略）

本次我的实验环境：

- MacBook Air m1芯片
- git 2.33.1
- node v16.13.1
- npm 8.1.2

## 2、注册github账号

​	相信如果是程序员同学，那一定已经拥有一个自己的github账号了，如果是已经拥有github账号的伙伴们可以直接看 *2.2小节* 。

### 2.1 注册账号

​	github注册账号还是比较简单的，需要一个邮箱就行。但是国内访问会比较慢，所以耐心一点就行，这里略过。

### 2.2 创建项目

​	创建项目需要注意的地方是仓库的名字，命名为 *你的账号.github.io*  ，之后个人博客访问的域名就是 *https://你的账号.github.io* 。

![创建github项目](https://img-blog.csdnimg.cn/9e3a87a1040a47038bcf0676a4c1d69c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

> 这里我已经创建过，所以提示已经存在。

​	项目创建好之后，需要本地把我们的项目拉下来，因为博客内容一般都是在本地写好在上传到github的，而且本地还可以启动服务预览博客，以便我们在正式发布之前，调整内容和样式。这里我们使用 `git` 协议去拉取仓库，就需要配置 `ssh key` ，因为直接使用账号密码不太安全，相关账号密码需要配置在配置文件。

### 2.3 配置ssh key

​	首先连接过的ssh key会在  `~/.ssh` 目录下有 *xxx.pub*文件，就是公钥。但是是首次连接的话，那么就需要配置一下了，如下：

```bash
# 1.查看.ssh文件夹，是否有配置过密钥
ls ~/.ssh
# 2.如果不存在密钥，则需要创建，如下命令直接三个回车就可以
ssh-keygen -t rsa-C "邮箱地址"
# 3.执行完以上步骤之后，会生成两个文件
id_rsa 和 id_rsa.pub
# 4.使用vim打开id_ras.pub，复制内容到github的公钥配置中
# 5.测试是否已经连通
ssh -T git@github.com
看到结果如下则表示连通：Hi 用户名! You've successfully authenticated, but GitHub does not provide shell access.
# 6.还需要配置下公共的用户名和邮箱
git config --global user.name "你的用户名"
git config --global user.email "你的邮箱"
```

### 2.4 设置gitbhu pages

​	找到项目的 `Settings -> Pages -> Source  ` ，设置branch和root。

![设置Github Pages](https://img-blog.csdnimg.cn/7282fbfc184f4245997fc0367c5caf73.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

## 3、搭建Hexo环境

​	Hexo是一个快速、简介且高效的博客框架，官网：[https://hexo.io/zh-cn/](https://hexo.io/zh-cn/) ，可以在网站下学习他的API、文档、插件、主题等。

### 3.1 搭建环境

**注意：** 由于mac系统新版本对 `/usr/*` 做了权限限制，无法写入，所以我们需要修改下默认的npm全局依赖包的位置。

```bash
# 0.修改npm全局依赖仓库位置，并设置到环境变量
mkdir ~/.npm-global
# 1.将新位置加入到环境变量，在./bash_profile中加入以下内容
export PATH=~/.npm-global/bin:$PATH
# 2.修改 .zshrc （因为我用的zsh），加入以下内容
source ~/.bash_profile
```

​	接下来我们搭建hexo环境：

```bash
# 1.安装hexo-cli工具（全局）
npm install -g hexo-cli
# 2.安装hexo（可局部安装，我就是安装在了我的博客项目中）
cd ~/projects/dmaker1993@github.io
npm install hexo
```

### 3.2 初始化Hexo

​	我们要使用 `hexo` 命令初始化一个hexo博客写作环境，相当于是hexo的基本框架：

```bash
# 1.将项目拉到本地某个文件夹，我的是 /Users/adu/IdeaProjects/dmaker1993.github.io
cd /Users/adu/IdeaProjects/
git clone git@dmaker1993@dmaker1993.github.io.git
# 2.进入项目中，初始化hexo框架
cd dmaker1993.github.io
hexo init
# 3.构建demo博客，生成静态文件
hexo generate
# 4.启动本地环境，查看demo
hexo server
# 5.访问localhost:4000
```

​	初始化之后的目录结构如下：

![hexo目录结构](https://img-blog.csdnimg.cn/b6f93034c3764ead8f64bf928ea85144.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_14,color_FFFFFF,t_70,g_se,x_16#pic_center)
```tex
.
├── _config.yml # 网站的配置信息，可以配置网站的绝大部分信息
├── package.json # 应用程序的信息，可以不关注
├── scaffolds # 模版文件夹，我们在创建新文章时，可以指定某一个模板来生成新文章（.md文档）
├── source # 存放用户资源文件的地方，同时这里也会存放我们的博客、标签、目录等内容，这里的.md和html文件会被编译到public文件夹
|   ├── _drafts # 草稿文件
|   └── _posts # 博客文件，这里的博客文件会被编译到public文件夹中
└── themes # 主题文件夹，hexo会根据主题生成博客静态页面
```

#### 3.3 选择主题

​	经过 *[3.2步骤](#3.2)* 之后，我们就已经搭建好了一个hexo环境，然后就可以写博客了。但是默认的信息比如 *博客名称*、*所有者* 等等信息，需要都替换成我们自己的，也就是做一些配置的修改，我们直接修改项目根目录下的 `_config.yml` 文件, 以下是一些hexo配置信息的修改：

```yaml
title: 网站标题
subtitle: 网站副标题
description: 网站描述
keywords: 网站的关键词。支持多个关键词
author: 作者姓名
language: 语言，对于简体中文用户来说，使用不同的主题可能需要设置成不同的值，请参考你的主题的文档自行设置，常见的有 zh-Hans和 zh-CN。
timezone: 网站时区。Hexo 默认使用您电脑的时区。请参考 时区列表 进行设置，如 America/New_York, Japan, 和 UTC 。一般的，对于中国大陆地区可以使用 Asia/Shanghai。
url: 网址, 必须以 http:// 或 https:// 开头
theme: 主题，这个是关键，我们可以在网上找一些好看主题
deploy:
  type: git
  repo: 仓库地址
  branch: 分支，一般是master
```

​	上述的 `theme` 就是主题的名字，我们可以到hexo官方提供的列表中选择一个自己喜欢的：[https://hexo.io/themes/](https://hexo.io/themes/)。我用的是 *[hexo-theme-melody](https://molunerfinn.com/hexo-theme-melody-doc/zh-Hans/)* 这个主题，他的文档也比较全，下面是一些展示：

![dmaker1993博客首页](https://img-blog.csdnimg.cn/5b038cb8bc9b4fe9b99cfcba1e9bef77.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)
![damper993博客内容详情页](https://img-blog.csdnimg.cn/bc1d723e296d4913b5c0e0ab625d9b9e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

​	更多的可以去看看我的博客：[https://dmaker1993.github.io/](https://dmaker1993.github.io/) 。

#### 3.4 主题下载和配置

​	在搭建的过程中也是直接看的文档，比较简单，如果有什么问题可以给我留言哦。首先下载主题，因为我下载的hexo是最新的，所以我直接使用npm安装。

1）主题下载

```bash
npm install hexo-theme-melody
```

​	下载之后，我们就可以在 `node_modules` 目录下找到这个主题的依赖：

![hexo主题目录结构](https://img-blog.csdnimg.cn/04df98ca7c494ea28618cf314d0633ab.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_14,color_FFFFFF,t_70,g_se,x_16#pic_center)


2）设置主题

​	找到项目跟目录下的 `_config.yml` ，修改其中的 `theme` 值为 `melody`。

```yaml
theme: melody # 将主题设置成melody
```

3）配置

​	先安装两个渲染插件：

```bash
npm install hexo-renderer-pug hexo-renderer-stylus --save
```

​	然后将 `node_modules/hexo-theme-melody/_config.yml` 文件复制到项目根目录下，并改名为 `_config.melody.yml` ，修改其中的一些配置信息，比如我的配置（`_config.melody.yml`）是这样的：

```yaml
# 配置语言的
language: zh-Hans
# 配置菜单的（ ':'之前的是展示的名字，之后的是博客中关联的）
menu:
  首页: /
  归档: /archives
  标签: /tags
  分类: /categories
  #XXX: /xxx
# 社交信息，可以挂一写自己的github，微博什么的
social:
#icon_name fa: url
#icon_name fab: url
  github fa: https://github.com/Dmaker1993
  weibo fa: https://blog.csdn.net/DMaker1993
# footer设置
since: 2021
```

全部的主题配置在这里：[https://molunerfinn.com/hexo-theme-melody-doc/zh-Hans/theme-config.html](https://molunerfinn.com/hexo-theme-melody-doc/zh-Hans/theme-config.html) 。

4）测试

​	都配置好之后，我们就可以看一下效果了，执行如下命令：

```bash
# 清理
hexo clear
# 构建。hexo generate 简写为：hexo g
hexo g
# 启动服务。hexo server 简写：hexo s
hexo s
```

​	正常是一片 *Hello World* 的demo文章，这里我们就都配置好了，下面我们就可以开始写博客了。

## 4、写博客

​	这个过程大概就是：生成博客文档、填充博客内容、构建博客页面、发布。等待几分钟之后，我们在自己的 *github.io* 页面中就能够看到了。下面我们看一下具体步骤：

```bash
# 1.生成博客文档，主要是生成一些hexo必要的配置
hexo new [layout] "博客名称" 
	## 这个命令会生成 ./source/_posts/博客名称.md 文档
	## layout表示模板名称，默认使用post，在./scaffolds中可以看到所有的模版
# 2.写博客，我一般会用 typora 写mk文档
# 3.将写好的mk文档复制到刚才生成的文档中，注意不要覆盖掉之前的配置，比如 title、categories等
# 4.构建
hexo g
# 5.本地预览
hexo s
# 6.发布，就是将本地的代码上传到github，这个过程是完全覆盖现在github上的内容的，所以一定要先拉下来。
hexo d
```

![创建新博客](https://img-blog.csdnimg.cn/bb8da1e5e6f94dadbab5387fc9b75758.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)
![创建结果](https://img-blog.csdnimg.cn/57fd27ae974643b49be6155b4e4bc5fe.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)
![创建博客默认内容](https://img-blog.csdnimg.cn/ec5e8acbb2264485b4a4494ec9cb5fc9.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

​	这里需要看一下最后一步 **发布** 过程，第一次如果发布失败，可能需要安装插件 `hexo-deployer-git` ：

```bash
npm install hexo-deployer-git --save
```

​	然后确认下 `_config.yml` 中git配置是否正确：

```yaml
deploy:
  type: git
  repo: <repository url> #https://bitbucket.org/JohnSmith/johnsmith.bitbucket.io
  branch: [branch] 默认maste
```

​	以上配置之后，在使用 `hexo d` 命令发布，等待几分钟，在 *您的账号.github.io* 中便可以看到内容。

​	**注意：因为发布过程是直接覆盖github内容，所以在写博客之前，先将github内容拉到本地，然后在发布！！！**


## 5、总结

​	综合以上内容，安装nodejs、npm、hexo，再有一个github账号，我们就可以免费拥有一个自己的博客系统啦～其实搭建的过程中还是会遇到很多困难，但是跟着看文档就都可以解决掉。而且 `hexo` 和 `hexo-theme-melody` 文档都支持中文，更方便了，赶紧搞起～



**参考：**

1. [https://hexo.io/zh-cn/docs](https://hexo.io/zh-cn/docs)
2. [https://www.cnblogs.com/liuxianan/p/build-blog-website-by-hexo-github.html](https://www.cnblogs.com/liuxianan/p/build-blog-website-by-hexo-github.html)
3. [https://molunerfinn.com/hexo-theme-melody-doc/zh-Hans/quick-start.html](https://molunerfinn.com/hexo-theme-melody-doc/zh-Hans/quick-start.html)
4. [https://dmaker1993.github.io/](https://dmaker1993.github.io/)

---

欢迎浏览我的更多文章：[一名不愿透露姓名的程序员](https://dmaker1993.github.io/) 或者  [https://dmaker1993.github.io](https://dmaker1993.github.io)

最后：因小的才疏学浅，如有问题，请不吝指出，感谢感谢～