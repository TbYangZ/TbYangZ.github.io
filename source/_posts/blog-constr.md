---
title: 博客重建那些事
date: 2024-10-07 23:23:07
tags:
- 日常
- 博客创建
categories:
- 生活
description: 在重新创建博客的时候遇到的一些事情。
typora-root-url: blog-constr
---

# 前言

在这国庆的最后一天，鉴于没什么游戏可以玩，把这个博客重新拾起。虽然之前的源文件在原来的老电脑丢了，但俗话说人要向前看，也是时候摒弃前面的东西，重新建立一下。当时创建的时候，还对 git、hexo 这些不是很熟悉，只是跟着别人的步骤依次做下来，导致没有把源码放在一个分支上；三年过去了，这次的我可跟以前不一样了，不会再犯这个错误了。

在重新建立这个博客之前，尝试过使用 wordprees 建立博客，但是建好了后，它上面的 markdown 插件太难用了，公式、图片等各种麻烦，远远不如 Hexo 这个静态博客，遂放弃。

这篇文章也旨在记录博客中的一些问题和解决方法，以供参考。

# 平台选择

为了更加方便的进行环境的配置，这次我选择在 Windows 11 中的 WSL2 下搭建这个博客，并将这个博客放在这里。之前是在 Windows 下的，但是都知道，Windows 下环境搭建真的是很麻烦，相较于 Windows，Linux 是真的方便，敲一行代码就可以安装好整个环境。通过知乎上的一篇文章，似乎 WSL2 是直接将整个电脑改造成一个虚拟机，Windows 和 Linux 都在这个虚拟机上跑。这样虽然对性能有部分损失，但是换来了优质的体验，这两个系统几乎是无缝操作的，Linux 下文件可以直接被复制到 Windows 下，同样，反过来也可以访问和修改。这样就即拥有了 Linux 的优秀的命令行工具，又可以使用 Windows 下的窗口，可谓两全其美。

由于只剩下了原来网站部署后的文件，一开始是想有没有什么方法可以还原它。理论上应该是可行的，Hexo 有博客迁移功能，可以从 RSS 迁移，如果之前设置了 RSS 的话可能会很方便，可惜没有。然后我看到可以将 html 转成 markdown，这样的话可以尽可能的保留我之前写的文章，但是尝试了 4 篇（前 4 篇题解）后，发现这个工作不亚于重新写一遍，故再放弃。（其实是看着之前的文章，中英文不加空格，中文之间保留英文标点等不规范的行为看不下去了）

# 环境搭建 & 工具下载

由于 hexo 是基于 Node.js 的，所以需要先下载 Node.js，这个在[官网](https://nodejs.org/en)下载就好了，都有说明。然后需要下载 Hexo，同样按照[Hexo 官网](https://hexo.io/zh-cn/)上的方法下载即可。虽然我参考的教程改了镜像源[1]，但是似乎我不用也能够正常下载？

Linux 下安装到个人目录可能更好[1]，但是我当时直接使用 `sudo` 莽过去了(:

到这里工具下载完了，但是后面还有几个东西需要下载。

# Hexo 本体

## 初始化

新建一个文件夹，然后执行指令初始化博客文件：

```bash
mkdir blog && cd blog
hexo init
```

## 主题设置

Hexo 自带了几个主题，但是都不是很好看，打算还是沿用原来的 [NexT 主题](https://theme-next.js.org/)。

官网上指出，有两种方式安装：

1. 使用 `npm` 安装：

```bash
cd blog
npm install hexo-theme-next
```

2. 直接 `clone` 本体：

```bash
cd blog
git clone https://github.com/next-theme/hexo-theme-next themes/next
```

这里推荐第二种方法。第一种方法的不知道安装到哪里去了（应该是 node_modules 文件夹里），之后配置什么不是很方便。

之后打开 `blog/_config.yml` 找到 `theme` 一栏，修改为 `next`。

```yml
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next
```

# Hexo 配置

## 数学公式

作为一名即将退役的 ACMer / 已经退役的 OIer，数学公式可谓是家常便饭，所以这一点对我来说非常重要。第一次装的时候公式始终渲染不起来，一气之下全部重新删了装了一边。

为了能够正确的渲染公式，需要将自带的渲染器（hexo-renderer-marked）卸载掉，装上能够渲染公式的渲染器。这里有几个选择：(1) hexo-renderer-kramed (2) hexo-renderer-pandoc (3) hexo-renderer-markdown-it (4) hexo-renderer-markdown-it-plus；具体区别可以参考[2]。

在一开始是想用 makrdown-it-plus，因为功能更多，但是渲染出问题了被迫抛弃。pandoc 功能要相对多一点，所以选择了使用 pandoc。

这里先将原来的渲染器卸载掉，再根据[官网](https://github.com/hexojs/hexo-renderer-pandoc)指导安装：

```bash
npm uninstall hexo-renderer-marked --save
npm install hexo-renderer-pandoc --save
```

之后还需要安装 [pandoc](https://pandoc.org/) 这个软件（就是 Typora 转化其他格式所需的那个），Linux 下可以直接安装（使用 `apt-get`），但直接安装的不一定是最新版[3]，但是最新的功能的也用不到就是了（目前就我而言）。

```bash
sudo apt-get install pandoc
```

之后在 `blog/themes/next/_config.yml` 里找到 `math` 设置。注意这里自动生成的部分有可能不一样，可以自行阅读英文注释理解。由于 pandoc 是使用 mathjax 的，所以只开启 mathjax 即可，mhchem 是一个辅助写化学反应式的插件。

```yml
# Math Formulas Render Support
math:
  # Default (true) will load mathjax / katex script on demand.
  # That is it only render those page which has `mathjax: true` in Front-matter.
  # If you set it to false, it will load mathjax / katex srcipt EVERY PAGE.
  per_page: false

  # hexo-renderer-pandoc (or hexo-renderer-kramed) required for full MathJax support.
  mathjax:
    enable: true
    # See: https://mhchem.github.io/MathJax-mhchem/
    mhchem: false

  # hexo-renderer-markdown-it-plus (or hexo-renderer-markdown-it with markdown-it-katex plugin) required for full Katex support.
  katex:
    enable: false
    # See: https://github.com/KaTeX/KaTeX/tree/master/contrib/copy-tex
    copy_tex: false
```

## 博客配置

### 图片相关

有两种方法，第一种是在 `blog/` 下新建一个 `images` 文件夹存放图片，这样文章中使用图片时候使用地址 `/images/image.png`。

第二种是对每个文章生成与文件同名的资源文件夹，将图片放在这里面，这样上传上去的时候文章也会被放在这个文件夹里，可以直接使用这个文件夹里的图片。在 `blog/_config.yml` 里设置后使用 `hexo new` 的时候会直接新建这样的文件夹[4]。

```yml
post_asset_folder: true # 启用文章资源文件夹
```

这两种方法各有优缺点，前者对多文章使用同一张图片节省空间，而后者图片等资源管理方便。但是这两种方法在本地都没有办法直接预览。由于本地和部署上去的地址不一样，本地是 `blog/images/image.png`，而在云端则是 `/images/image.png`，导致本地显示不了图片。虽然可以使用 `hexo s` 进行调试，但还是很麻烦。

只能对本地的编辑器进行修改了。如果使用的是 Typora，可以在前置信息（Front-matter）中添加 `typora-root-url: `，这样的话 Typora 就会在指定位置下寻找图片内容。

如果使用第二种存储图片的方法的话，可以在 `blog/scaffolds/post.md` 里添加 `typora-root-url: {{ title }}`，这样新建的时候会自动加上，就不需要手动添加[1]。

更新：Typora 中可以将图片复制到指定位置：

![](image.png)

可以选择复制到指定位置，然后规定指定位置为 `./${filename}`，同时选择优先使用相对位置即可，同时配合上面设置 `typora-root-url` 这样可以直接将图片复制到文件中。

如果有能力的话，也可以将图片上传至自己的服务器或者其他可以管理图片的图床等。

### 博客信息

这部分比较简单，在 `blog/_config.yml` 里进行寻找。常见的比如博客名字、副标题、描述等。

```yml
# Site
title: owack's blog
subtitle: 'A website to record life and study'
description: 'Think twice, code once'
keywords:
author: owack
language: zh-CN
timezone: ''
```

博客的图片、个人的头像等在 `blog/themes/next/_config.yml`，即主题的配置文件中进行修改：

```yml
favicon:
  small: /images/favicon-16x16-next.png
  medium: /images/favicon-32x32-next.png
  apple_touch_icon: /images/apple-touch-icon-next.png
  # safari_pinned_tab: /images/logo.svg
  #android_manifest: /images/manifest.json
  #ms_browserconfig: /images/browserconfig.xml

# Sidebar Avatar
avatar:
  # Replace the default image and set the url here.
  url: /images/avatar.jpg
  # If true, the avatar will be dispalyed in circle.
  rounded: false
  # If true, the avatar will be rotated with the cursor.
  rotated: false
```

这些图片是存放在 `blog/themes/next/source/image/`。

### 额外页面

如果想要添加额外的页面，如标签页、关于、分类页等，可以使用 

```shell
> hexo new page about
> hexo new page tags
> hexo new page categories
```

之后，hexo 会帮我们创建 `about`、`categories` 和 `tags` 这三个文件夹，并且里面默认有一个文件。然后我们在里面的导言区加上 `type: "..."`，其中 `...` 为每个文件的功能，如作为标签则填写 tags。然后在每篇文章的导言区里就可以加入分类和标签啦。

具体如下：

```yml
title: xxx
time: xxx
tags:
- tag1
- tag2
categories:
- category
description: xxx
```

需要注意的是标签（tags）可以有多个，而分类（category）只能有一个。另外如果想要在主页侧边栏中开关，可以在 theme 里的 `_config.yml` 里进行修改：

```yml
menu:
  home: / || fa fa-home
  about: /about/ || fa fa-user
  tags: /tags/ || fa fa-tags
  categories: /categories/ || fa fa-th
  archives: /archives/ || fa fa-archive
  #schedule: /schedule/ || fa fa-calendar
  #sitemap: /sitemap.xml || fa fa-sitemap
  #commonweal: /404/ || fa fa-heartbeat
```

### 背景图片和虚化

首先需要在 `blog/themes/next/_config.yml` 里将自定义格式打开。

```yml
# Define custom file paths.
# Create your custom files in site directory `source/_data` and uncomment needed files below.
custom_file_path:
  #head: source/_data/head.swig
  #header: source/_data/header.swig
  #sidebar: source/_data/sidebar.swig
  #postMeta: source/_data/post-meta.swig
  #postBodyEnd: source/_data/post-body-end.swig
  #footer: source/_data/footer.swig
  #bodyEnd: source/_data/body-end.swig
  #variable: source/_data/variables.styl
  #mixin: source/_data/mixins.styl
  style: source/_data/styles.styl   # 取消这一行的注释
```

然后在 `blog/source/` 下新建 `_data/styles.styl` 文件（文件夹也要新建），里面填充以下内容[5]：

```styl
// 博客背景设置
body {
    background:url(/images/bg.png);
    background-repeat: no-repeat;
    background-attachment:fixed; // 不重复
    background-size: cover;      // 填充
    background-position:50% 50%;
}
// 博客内容透明化
// 文章内容的透明度设置
.content-wrap {
  opacity: 0.8;
}

// 侧边框的透明度设置
.sidebar {
  opacity: 0.8;
}

// 菜单栏的透明度设置
.header-inner {
  background: rgba(255,255,255,0.8);
}

// 搜索框（local-search）的透明度设置
.popup {
  opacity: 0.9;
}
```

### 代码相关

目前遇到过的困难是复制按钮和缩进。

复制按钮在 `blog/themes/next/_config.yml` 里设置。这里还可以调整高亮模式。

```yml
codeblock:
  # Code Highlight theme
  # Available values: normal | night | night eighties | night blue | night bright | solarized | solarized dark | galactic
  # See: https://github.com/chriskempson/tomorrow-theme
  highlight_theme: normal
  # Add copy button on codeblock
  copy_button:
    enable: true
    # Show text copy result.
    show_result: false
    # Available values: default | flat | mac
    style:
```

默认似乎是一个 tab 等于 8 个空格，太长了，在 `blog/_config.yml` 里设置，把 `tab_replace` 换成 4 个空格，代码观感好很多。

```yml
highlight:
  line_number: true
  auto_detect: false
  tab_replace: '    '
  wrap: true
  hljs: false
```

# 部署到 github page

参考[6]。

为了防止我之前的悲剧再度发生，这次得把源码好好存着。

在源文件的文件夹下依次执行

```bash
git init
git remote add origin git@github.com:username/username.github.io.git
git checkout -b source
git fetch origin
git push origin source
```

之后可以再到 Github 上检查一下是否添加成功。


---

**References**

[1] http://home.ustc.edu.cn/~liujunyan/blog/hexo-next-theme-config/

[2] https://bugwz.com/2019/09/17/hexo-markdown-renderer/

[3] https://blog.csdn.net/weixin_44375591/article/details/104005570

[4] https://blog.csdn.net/2301_77285173/article/details/130189857

[5] https://blog.csdn.net/TomAndersen/article/details/104872852

[6] https://oceanwang.top/personal-website-7/