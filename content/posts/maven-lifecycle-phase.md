---
title: "浅析Maven的lifecycle和phase"
date: 2022-07-03T21:21:01+08:00
draft: false
author: finger
tags:
    - 'build tool'
    - 'maven'
categories:
    - maven
---

## 啥是maven
`maven`是一个构建工具,啥又是构建工具?简单来说,构建工具就是自动化从源代码执行的工具.包括但不限于,自动编译,打包,部署,创建文档等

## Maven怎么进行自动构建
`maven`通过项目读取项目根目录中的`pom.xml`文件最终构建成一个`pom(proejct object model)`对象,来获取项目的构建信息,比如,父级项目,项目依赖那些`jar`包,最终的打包结果是什么,已经`jdk`的版本.

## lifecycle 和 phase
要研究maven是怎么构建的,要先了解`maven`的`lifecycle`和`phase`.
`maven` 的整个构建过程是基于`lifecycle`,而`lifecycle`是有`phase`构成的.
`maven`内置3个`lifecyle`:
- default
- clean
- site

如果需要自己添加`lifecycle`可以自己更多`${MAVEN_HOME}/libexec/lib/maven-core-3.8.5/META-INF/plexus/components.xml`配置文件
每个`lifecycle`由不同的`phase`组成,`lifecycle`就是用来配置phase执行的.例如`clean`由以下`phase`组成:
- `pre-clean`
- `clean`
- `post-clean`

需要注意两点:
- `phase`是有顺序的,先执行后面的`phase`会把前面的也执行
- 但是需要注意的是,`maven`只是定义了这些`lifecycle`和`phase`,里面并没有做任何事情,`maven`只是定义了一个架子

###  plugin goal
上面我们说到,`maven`只是定义了一个架子,那我们日常执行的`complie`,`package`是怎么执行的呢?
这一切都来源于`maven`的精髓`plugin goal`,虽然每个`lifecycle`和`phase`并没有做任何事,但是`maven`可以将`plug goal`绑定到各个`phase`,从而执行想要的操作.
`phase`是由`plugin goal`组成的,`plug goal`是一系列特定的任务,是比`phase`更细粒度的任务.
一个`plug goal`可以绑定到`0-你`个`phase`,一个`phase`也可以绑定`0-n`个`plug goal`
如果一个`plug goal`没有绑定任何一个`phase`,可以脱离`lifecycle`直接调用
如果一个`phase`没有绑定任何`plug goal`,`phase`不会执行
现在回到上面的问题;
> 上面我们说到,`maven`只是定义了一个架子,那我们日常执行的`complie`,`package`是怎么执行的呢?

这其中的奥妙就在于maven提供了一系列默认的`plug goal`来绑定到对应的`phase`,从而就有了开箱即用的感觉.

### 给`phase`添加`plug goal`
那么问题来了,如何给`phase`添加`plug goal`呢,官方文档介绍了两种方式:

####  Packaging
设置`packaging`元素不同的值,就是将不同的`plug goal`绑定的`default lifecycle`的`phase`中,例如,当`packaging`是`jar`类型的时候,会有如下绑定:
|Phase|plugin:goal|
|-------|--------|
|`process-resources`|`resources:resources`|
|`compile`|`compiler:compile`|
|`process-test-resources`|`resources:testResources`|
|`test-compile`|`compiler:testCompile`|
|`test`|`surefire:test`|
|`package`|`jar:jar`|
|`install`|`install:install`|
|`deploy`|`deploy:deploy`|

#### Plugins
第二种方式就是在项目中给`phase`设置`plugin`,一个插件可以有多个`goal`,所以`plugin`可以绑定到多个`phase`,`plugin`会有默认绑定的`phase`
下面是一个设置`plugin`的例子
```xml
 <plugin>
   <groupId>org.codehaus.modello</groupId>
   <artifactId>modello-maven-plugin</artifactId>
   <version>1.8.1</version>
   <executions>
     <execution>
       <configuration>
         <models>
           <model>src/main/mdo/maven.mdo</model>
         </models>
         <version>4.0.0</version>
       </configuration>
       <goals>
         <goal>java</goal>
       </goals>
     </execution>
   </executions>
 </plugin>
```
当然我们也可以自己开发`plugin`,不过这就是另外一个故事了
## 结语
以上就是我对于`maven`的`lifecycle`和`phase`的粗浅理解,如有不同意见,还请指正.

## 参考资料
> https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html
