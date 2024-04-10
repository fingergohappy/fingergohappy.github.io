---
title: 管道操作符和xargs的区别
tags:
  - shell
categories:
  - shell
cover: https://source.unsplash.com/a-computer-screen-with-a-program-running-on-it-NLSXFjl_nhc/1200x628
date: 2023-10-31 17:38:16
abstracts: shell中管道操作符和xargs的区别
---


之前一直没有理解shell中管道操作符和xargs的区别,为什么有一些命令要用可以直接用管道操作符链接,为什么一些需要加个`xargs`,最近翻了一些文档之后,终于理解


# 官方定义

最好的解释永远来自于官方文档,正所谓真传一句话,假传万卷书.

## 管道操作符

A pipeline is a sequence of one or more commands separated by one of the control operators ‘|’ or ‘|&’.
The output of each command in the pipeline is connected via a pipe to the input of the next command. That is, each command reads the previous command’s output. This connection is performed before any redirections specified by command1.


简单来说就是,管道操作符连接一系列命令,前面的标准输出作为后面的标准输入

<!--more-->


## xargs

The xargs utility reads space, tab, newline and end-of-file delimited strings from the standard input and executes utility with the strings
     as arguments.
Any arguments specified on the command line are given to utility upon each invocation, followed by some number of the arguments read from
the standard input of xargs.  This is repeated until standard input is exhausted.

Spaces, tabs and newlines may be embedded in arguments using single (`` ' '') or double (``"'') quotes or backslashes (``\'').  Single
quotes escape all non-single quote characters, excluding newlines, up to the matching single quote.  Double quotes escape all non-double
quote characters, excluding newlines, up to the matching double quote.  Any single character, including newlines, may be escaped by a
backslash.



简单来说,就是xargs是从标准输入读取内容,作为`xrags`命令后面跟的命令的参数


## 举个例子

### 管道操作符

```shell
echo "hello \n world" | grep hello
```

`echo "hello world"` 会将`hello world`字符串输出到**标准输出**,然后通过管道操作符作为 `grep hello`这条命令的**标准输入**

上面的命令相当于下面的命令

```shell
grep hello      # 执行完按回车
hello       # 输入hello world
hello      # 打印hello
world   # 输入world 什么也不打印
```

### xargs

```shell
xargs ls #按回车
/usr     #按ctrl-d 会打印/usr目录下的所有文件
```

`ctrl-d`相当于向`shell`输入了一个`eof`,表示输入完成

### 组合

这两个命令一般是组合使用

比如:

```shell
cat a.txt | xargs mkdir -p 
```

这条命令的含义是,将a.ext文件中的内容作为文件夹名,新建一个文件夹
其中,`cat a.txt`命令是将`a.txt`中的内容输出到**标准输出**,然后通过管道操作符作为`xargs mkdir -p`这个命令的标准输入,`xargs`将前面传过来的标准输入作为`mkdir -p` 这条命令的**参数**

# 标准输入/标准输出/参数

在linux中,默认情况下

标准输入是指键盘
标准输出是指屏幕
参数就是命令后面跟着的内容


## 举个例子

```shell
$ cat hello-world.txt #  hello-world.txt 是参数

$ cat   #执行cat命令后回车
$ hello  #输入hello   这里的hello就是标准输入
$ hello  # 命令行打印hello  这里打印的hello就是找女友输入
```




