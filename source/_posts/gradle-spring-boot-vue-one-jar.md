---
title: gradle将spring-boot和vue打入一个jar包
tags:
  - gradle
  - spring-boot
categories:
  - gradle
date: 2023-11-11 15:23:43
abstracts: 使用gradle将spring-boot和vue打入一个jar包
toc: true
---

之前用`maven`搞过将`spring-boot`和`vue`打成一个jar包,现在的项目,我使用了`gralde`,又是同样的需求,使用`gradle`的`composite`似乎更优雅一些.


# 项目结构


```bash
.
├── build.gradle.kts
├── settings.gradle.kts
├── app #spring-boot 目录
│   ├── build.gradle.kts
│   ├── settings.gradle.kts
├── web #vue目录
│   ├── build.gradle.kts
│   ├── settings.gradle.kts

```
<!--more-->

# 结构说明

`app`和`web`分别为`spring-boot`和`vue`项目,就正常写就行,唯一不同的就是,`vue`项目里面添加一个`gradle`配置文件

# 配置文件说明

需要改动的地方就只有`web`和最外面的项目的`gradle`的配置文件需要改动

## web

其中`web`的`build.gradle.kts`内容如下:


```gradle
plugins{
        base
}

tasks{
        register<Exec>("build-dev"){
            workingDir("./")
            commandLine("pnpm","run","build:dev")
        }
}
```
就是添加了一个打包任务


## 根目录

根目录的`gradle`配置文件需要调整的地方比较多


`setting.gradle.kts`:


```kotlin
rootProject.name = "example"

// 这里很重要,使用includeBuild完成组合构建
includeBuild("./app")
includeBuild("./web")
```

`build.gradle.kts`:


```kotlin

plugins {
    base
}

tasks{
    register("build-all"){
        dependsOn(gradle.includedBuild("app").task(":classes"))
        dependsOn(gradle.includedBuild("web").task(":build-dev"))
    }
    register<Copy>("copy-static"){
        dependsOn("build-all")
        // gradle没有提供通过includeBuild直接获取build目录的api,只能这样写死
        val webDistPath = gradle.includedBuild("web").projectDir.getPath() + "/dist-dev"
        val appClassesPath = gradle.includedBuild("app").projectDir.getPath() + "/build/resources/main/static"
        from(webDistPath){
                include("*")
        }
        into(appClassesPath)
    }
    register("build-jar"){
        dependsOn("copy-static")
        dependsOn(gradle.includedBuild("app").task(":jar"))
    }
}

```

这样执行`gradle :build-jar`,就会生成一个带有`vue`打包后的`dist`目录的`jar`包
