---
title: linux下安装多版本jdk并进行切换
tags:
  - linux
categories:
  - linux
date: 2024-01-05 15:24:32
abstracts: 如何优雅的linux下安装多版本的jdk并进行切换
toc: true
---


# 背景

进行程序升级的时候,可能要在服务器上安装多个版本的`jdk`并且进行切换,我在本机使用的是`asdf`,这个东西用来管理多版本还是挺香的,不过由于服务器无法访问国际互联网,只能寻找其他替代方案.



# alternatives 

`alternatives`是`linux`系统中一个十分强大的命令,
主要功能就是为了解决,系统中有类似的命令,给用户一个选择的方式,在多个`jdk`的切换就很有用


<!--more-->

# 配置

## 删除原来的内容

如果你原来配置`jdk`方式是通过修改环境变量来实现的,要现去除掉这个配置,一般在`/etc/profile`或者`/etc/environment`或者`~/.bash_profile`这几个文件中,将原来的配置删除掉

```bash
# 全部注释掉
#export JAVA_HOME=/var/local/jdk1.8.0_202
#export JRE_HOME=${JAVA_HOME}/jre
#export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
#export PATH=${JAVA_HOME}/bin:$PATH
```
然后执行`source /etc/profile`生效,如果不生效就重新登录下.
这部完成后,再验证下,`exho $PATH`,看下`java`命令是不是在`PATH`中

## 使用laternatives配置jdk

删除掉原来的配置后,就可以使用`alternatives`来配置`java`命令了,
找到你两个`jdk`的位置,然后执行一下命令:

```bash
sudo alternatives --install /usr/bin/java java /opt/jdk1.8.0_301/bin/java 1
sudo alternatives --install /usr/bin/javac javac /opt/jdk1.8.0_301/bin/javac 1

sudo alternatives --install /usr/bin/java java /opt/jdk-11.0.12/bin/java 2
sudo alternatives --install /usr/bin/javac javac /opt/jdk-11.0.12/bin/javac 2
```


## 切换jdk

后面如果想切换`java`版本,就可以使用命令:

```bash
$ alternatives --config java

There are 2 programs which provide 'java'.

  Selection    Command
-----------------------------------------------
   1           /var/local/jdk1.8.0_202/bin/java
*+ 2           /usr/local/jdk-11/bin/java

Enter to keep the current selection[+], or type selection number:
```
输入一个序号就完成`java`版本的切换了


