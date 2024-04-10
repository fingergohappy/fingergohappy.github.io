---
title: gradle spring boot 打包无依赖jar
tags:
  - java
  - gradle
categories:
  - java
cover: https://source.unsplash.com/1200x628
date: 2023-12-22 20:21:39
abstracts: gradle将spring boot项目打包成无依赖jar包的轻量级包
toc: true
---


# 背景

希望`spring-boot`的`jar`包不包含项目的依赖,打包成一个轻量`jar`包,方便部署和快速打包


# 方法一:

修改`build.gradle.kts`:

```kotlin
tasks.withType<Jar>{
    manifest {
        attributes["Main-Class"] = "com.io.alyze.AlyzeApplicationKt"
        //添加所有依赖的
        attributes["Class-Path"] =  configurations.runtimeClasspath.get().files.joinToString(" ") { "./libs/${it.name}" }
    }
}
tasks.register<Copy>("jar-dependency") {
    dependsOn("jar")
    from(configurations.runtimeClasspath)
    into(layout.buildDirectory.dir("libs/libs/"))
}
```

<!--more-->

这样执行:`./gradlew :jar-dependency` 就会在`build/lib`目录生成一个`xxx-palin.jar`和一个`libs`文件夹

将这两个文件拷贝在一起,以使用`java -jar xxx-plain.jar`直接运行


# 方法二


修改`build.gradle.kts`:

```kotlin
tasks.register<Copy>("jar-dependency") {
    dependsOn("jar")
    from(configurations.runtimeClasspath)
    into(layout.buildDirectory.dir("libs/libs/"))
}
```


这样执行:`./gradlew :jar-dependency` 就会在`build/lib`目录生成一个`xxx-palin.jar`和一个`libs`文件夹

将这两个文件拷贝在一起,可以使用`java -cp "alyze-1.0-plain.jar:libs/*" your_main_class_name`直接运行
