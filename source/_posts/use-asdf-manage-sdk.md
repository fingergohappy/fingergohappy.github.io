---
title: 使用asdf管理你的sdk
tags:
  - asdf
  - asdf manage
categories:
  - 效率工具
cover: https://source.unsplash.com/1200x628
date: 2023-09-26 12:18:16
abstracts:
---

对于我这种使用多种语言开发,并且可能在给个版本之前来回切换的人来说,一直渴望一个好的`sdk`管理工具
举个具体的例子:有两个项目,一个项目使用`jdk-8`另一个项目使用`jdk-17`,在`ide`里面还好说,毕竟可选择,但是当你在命令行里面操作时,来回切换`jdk`有多难受就不必我多说了吧
而且如果要用两个`jdk`,这两个就要使用`HomeBrew`安装两个`jdk`,而且安装的路径还不一样

除此之外,还有前端的`node`版本,不同的项目使用的`NodeJs`版本不同,换来换区也是麻烦的不行

这个时候就体现了[asdf](https://asdf-vm.com)这个工具的方便之处,

如果使用来`asdf`,那么安装和切换`jdk`的版本就会如此美妙

现在假设在使用`asdf`的情况下,如果有两个项目`a` 和 `b`,分别使用`jdk-8`和`jdk-17`,那么管理的流程就是如下的

1. 安装对应的`jdk`:`asdf install java openjdk-8`, `asdf install java openjdk-17`
2. 分别进入`a`和`b`两个目录,执行`asdf local java openjdk-8`,`asdf local java openjdk-17`

这个时候,当你在命令行进入对应的项目,你执行`java -version`就会对应不同的`jdk`版本

而且安装的`jdk`统一的都在`~/.asdf.plugins/java`目录下,丝毫不会污染你其他的目录,干净又卫生了属于是



下面简单说下安装以及使用,基本上是官方文档内容

# 安装

我是使用`HomeBrew`安装

```bash
brew install asdf

echo -e "\n. $(brew --prefix asdf)/libexec/asdf.sh" >> ${ZDOTDIR:-~}/.zshrc
```


# 使用(以java为例)


## plugin

在`asdf`的语义中,`plugin`就是帮你管理对应语言`sdk`的模块,比如你需要下载是管理`java`的版本,你就需要下载对应的`java plugin`


## 搜索plugin

```bash
asdf plugin list all | grep java
```


## 安装plugin

```bash
asdf plugin add java
```


## 安装sdk

```bash 
# 搜索jdk
asdf list all java | grep jdk 

# 安装jdk
    # 安装指定版本
asdf install java openjdk-17.0
    # 安装指定版本的最新版本
asdf install java openjdk-17:latest
    # 安装最新稳定版
asdf install java latest
```

## 设置

```bash

# 设置命令行使用的版本
asdf global java openjdk-17
# 设置当前路径使用的jdk版本,会在当前目录下生成一个.tool-versions文件
asdf local java openjdk-17
```


##  其他常用命令


```bash
# 查看当前各个sdk的版本
asdf current
# 查看当前java使用的sdk版本
asdf current java
# 查看安装那些plugin
asdf plugin list
# 查看当前使用的java 命令的路径
asdf  which java
# 查看当前使用的java sdk的路径
asdf where java
# 移除对应的sdk
asdf uninstall java openjdk-17
# 移除对应的plugin
asdf plugin remove java
# 更新(所有)plugin
asdf plugin update [--all]
```

# 参考资料

`asdf`[官方文档](https://asdf-vm.com)

