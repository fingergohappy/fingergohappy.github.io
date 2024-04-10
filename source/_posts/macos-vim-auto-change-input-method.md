---
title: macos下,实现vim切换模式自动切换输入法
tags:
  - macos 
  - vim
categories:
  - some litttle tricks
date: 2023-09-20 10:37:19
cover: https://source.unsplash.com/m_HRfLhgABo/1200x628
abstracts: macos结合使用,vim切换模式实现自动切换输入法
feature: true
---


# macos下,实现vim切换模式自动切换输入法

作为一个重度`vim`模式使用用户,切换模式输入法不自动切换一直是一个蛋疼的问题,目前主要解决方案就是使用`im-select`,在对应使用的应用里面设置`im-select`的路径
但是这种方式有一个问题就是如果应用不支持,就不会自动切换

另一种方式就是使用`rime`输入法,但是对于我来说配置成本太高,不划算.

经过我的研究,发现`karabiner`可以实现在任意应用中使用`vim`,按下对应的模式切换键实现自动切换输入法.


## 安装karabiner

`karabiner`是mac下一个按键功能映射的软件,可以自己定义`json`文件实现许多复杂的功能的映射

`brew`不解释连招
<!--more-->


```bash
brew install karabiner
```

## 安装im-select

继续`brew`不解释连招

```bash
brew install im-select
```

## 创建映射文件

`/Users/fingerfrings/.config/karabiner/assets/complex_modifications`路径创建一个`vim-esc.json`文件

文件内容如下:

```json
{
  "title": "Vim shortcuts",
  "rules": [
     {
      "description": "Map ^+[ to esc",
      "manipulators": [
        {
          "type": "basic",
          "from": {
            "key_code": "open_bracket",
            "modifiers": {
              "mandatory": ["control"]
            }
          },
          "to": [
            {
              "key_code": "escape"
            },{
              "shell_command": "/usr/local/bin/im-select com.apple.keylayout.ABC"
            }
          ]
        }
      ]
     },
    
  ]
}
```

上面文件的映射内容就是将`Ctrl-[`映射为`Esc`,同时执行`/usr/local/bin/im-select com.apple.keylayout.ABC`命令

## 设置karabiner

在`complex Modifications`选项中启用`Map ^+[ to esc`


## 参考文档
[https://karabiner-elements.pqrs.org/docs](https://karabiner-elements.pqrs.org/docs/json/complex-modifications-manipulator-definition/to/shell-command/)
