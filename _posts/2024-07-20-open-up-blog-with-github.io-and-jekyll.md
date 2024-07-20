---
layout: post
title: "使用 github.io 和 jekyll 建立博客"
date: 2024-07-20 22:03:11 +0800
categories: 操作记录
---

自己做的很多东西如果不记录就很难再回忆了，于是打算重启博客。

之前的博客依附于服务器，在移植的时候相当麻烦，并且因此一度不想再做。最近又展开了很多学习和工作的项目，又激发了我建立博客的欲望。不同于当初，这次我开始寻找一种更简洁的方式，用 github.io（及其使用的 jekyll）建立博客。它在一个 repo 里就能实现需求，看起来是一个相当不错的方案。

## 一、github.io 仓库的建立

一个以 *username*.github.io 命名的 repo，会自动作为 jekyll 文件仓库生成 Github Pages 页面（默认网址 https://*username*.github.io/）。参考 <https://pages.github.com/>。

## 二、Jekyll 本地测试

参考 [Testing your GitHub Pages site locally with Jekyll](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll)

### 2.1 安装 Jekyll

Windows 安装 ruby（我使用的是 msys2，用 pacman 管理；不过直接用 winget 之类的应该更舒服）

```shell
pacman -S mingw-w64-x86_64-ruby
```

然后安装 jekyll 和 bundler

```shell
gem install jekyll bundler
```

#### 问题：补丁还没发正式版

在 gem install jekyll 中，google-protobuf 库的安装出现问题，

```shell
compiling defs.c
defs.c: In function 'MethodDescriptor_initialize':
defs.c:1513:19: error: assignment to 'const upb_MethodDef *' from incompatible
pointer type 'const upb_ServiceDef *' [-Wincompatible-pointer-types]
 1513 |   self->methoddef = (const upb_ServiceDef*)NUM2ULL(ptr);
      |                   ^
make: *** [Makefile:250: defs.o] Error 1
```

这个问题 protobuf 已经有 commit [[Ruby] Fix mismatched pointer type](https://github.com/protocolbuffers/protobuf/commit/0aa74497527c6656e073f2635019ebc2a946268c) 解决了（2024.7.3 两周前），但是还没有发正式版（latest v27.2 是2024.6.26 一个月前），只有一个 pre-release v28.0-rc1 修了这个问题。

似乎可以从源码下载，谷歌找到了 stackoverflow 的问题 [How to install gem from GitHub source?](https://stackoverflow.com/questions/2577346/how-to-install-gem-from-github-source)，下面有一种可行的解决方案。例如这样，

```shell
gem 'google-protobuf', :git => 'git@github.com:protocolbuffers/protobuf.git', :tag => 'v28.0-rc1'
```

但是没有学过 ruby，并不知道这里 Gemfile 相关机制是否安全/将来怎么维护，但是摸索中知道了这个 28.0-rc1 的 gem 版本格式是 4.28.0.rc.1，所以尝试直接按照版本安装，

```shell
gem install google-protobuf -v 4.28.0.rc.1
```

成功。然后安装 jekyll，成功。

### 2.2 Jekyll 其他依赖

先检查版本，

```shell
CMD> jekyll -v
jekyll 4.3.3
```

完工后，踩了坑发现要先安装依赖，需要添加 Gemfile 文件（windows 下），

```gemfile
# in `Gemfile`
source 'https://rubygems.org'
gem 'tzinfo', platforms: [:mingw, :mswin, :x64_mingw]
gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw]
gem 'jekyll-watch', platforms: [:mingw, :mswin, :x64_mingw]
gem 'webrick', platforms: [:mingw, :mswin, :x64_mingw]
gem 'wdm', '>= 0.1.0' if Gem.win_platform?
```

然后先用 bundler 安装依赖再运行，

```shell
bundler install
```

完成依赖安装。

如果 wdm 因为 cflags 的 warning as error 安装失败，试试单独处理（参考 <https://github.com/ffi/ffi/issues/840>），

```shell
gem install wdm -- --with-cflags=-Wno-implicit-function-declaration
```

### 2.3 Jekyll 生成与运行

```shell
jekyll build
jekyll serve
```

进 <http://localhost:4000/> 看一眼，成功。

## 三、博客所需文件与内容

部分参考[这篇博客](https://jokinkuang.github.io/2016/09/03/how-to-create-the-jekyll-theme.html)

jekyll 除了下划线（以及 `.` `#`、其他一些特殊文件夹，参考 [About GitHub Pages and Jekyll](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/about-github-pages-and-jekyll#configuring-jekyll-in-your-github-pages-site)）开头的 `_xxx` 文件夹不被跟踪，其他文件和文件夹都会视为普通的静态资源。

先生成默认主题，文件内容会放在 \<any-name\> 下（实际上这一步应该一开始做，就不用再复制粘贴出来了）

```shell
jekyll new <any-name>
```

接着复制到 github.io 文件夹。或者直接搜索一个好用的库然后 clone（这里由于 favicon 的问题，我选择直接使用 minima 原仓库）

当然，google-protobuf 的 Gemfile 也要改成这样

```gemfile
source 'https://rubygems.org'
gem 'google-protobuf', '>= 4.28.0.rc.1', platforms: [:mingw, :mswin, :x64_mingw]
gem 'minima', platforms: [:mingw, :mswin, :x64_mingw]
gem 'tzinfo', platforms: [:mingw, :mswin, :x64_mingw]
gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw]
gem 'jekyll-watch', platforms: [:mingw, :mswin, :x64_mingw]
gem 'webrick', platforms: [:mingw, :mswin, :x64_mingw]
gem 'wdm', '>= 0.1.0' if Gem.win_platform?
```

当然，这里你可以采用其他主题。

## 四、撰写博客

`_posts` 内以 `年-月-日-内容` 命名的 `.md` 文件就是博客了，但是开头还要加上诸如这类标记。

```yaml
---
layout: post
title: "使用 github.io 和 jekyll 建立博客"
date: 2024-07-20 22:03:11 +0800
categories: 操作记录
---
```

其他分类与导航可以参考[这篇博客](https://zoharandroid.github.io/2019-08-02-Jekyll%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2%E6%B7%BB%E5%8A%A0%E5%88%86%E7%B1%BBCategory%E5%8A%9F%E8%83%BD/)。
