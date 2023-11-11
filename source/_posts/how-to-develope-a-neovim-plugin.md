
title: 手把手教你开发一个neovim插件
tags:
  - neovim
categories:
  - neovim奇技淫巧
cover: https://source.unsplash.com/iMwiPZNX3SI/1200x628
date: 2023-10-12 17:10:19
abstracts: 手把手胸贴胸教你开发一个neovim插件
---


# 开篇废话

使用`neovim`久了,使用了众多插件,不自己开发一个插件,怎么好标榜自己是一个`vimer`?
今天老夫就手把手教大家开发一个自己的`neovim`插件

# 项目结构与插件路径

## 插件路径

这里差不多也是废话,理论上来说,`vim`的插件可以在任何位置,插件的方式也可以是各种形式,不过为了这里面还是有一些潜规则的.
首先就是插件的路径,一般来说,`neovim`会默认加载一些路径,可以在`neovim`中执行`echo &rtp`(rtp:runtime path)来查看默认的加载路径有哪些,不过由于我们是
自己开发的插件,就暂时不要放在这些路径里面了,避免启动时报错,假设我现在放在了`~/temp/neovim/lua/lowb`这个目录下进行开发`lowb`插件,然后在启动`neovim`的时候使用`nvim --cmd "set rtp+=~/temp/neovim"`启动就可以加载`lowb`插件了,
插件名就是`lua`路径下的文件夹名,如果你的插件比较简单,可以不用新建一个文件夹,直接搞一个`lowb.lua`就可以加载了,不过按照约定一般是有一个文件夹的,方便发布,供其他人使用

**这里需要注意,一定放在`lua`的子目录下,否则`neovim`是不会进行加载这个插件的,[参考](https://neovim.io/doc/user/lua-guide.html#lua-guide-modules)**

## 项目结构


### init.lua

`neovim`会自动加载目录下的`init.lua`作为`neovim`的默认加载文件


# 开始开发

## 加载

确定了插件路径和项目结构,就可以开发`lowb`插件了,在`~/temp/neovim/lua/lowb`,先新建一个`init.lua`文件,文件内容如下:
```lua
print("Hello lowb")

```

然后使用`nvim -c "set rtp+=~/temp/neovim"`启动`neovim` 

进入`neovim`的命令模式执行:`lua require('lowb')`
可以看到会打印`Hello lowb`
OK,现在插件成功加载了


## 导出函数

插件肯定不能简单的加载,还要导出一些函数供用户使用,更新`init.lua`

```lua
local M = {}
function M.say_lowb()
    print("hello lowb")
end

return M
```


保存插件,重新使用命令`nvim -c "set rtp+=~/temp/neovim"`启动`neovim`
进入`neovim`的命令模式执行:`lua require('lowb').say_lowb()`

现在`lowb`插件基本完成,还有一项功能就是给用户设置的能力


## setup函数

`neovim`插件一般是通过向外暴露一个`setup`函数来给用户设定,这倒不是强制规定,只是一个默认约定,像一些插件管理工具,比如[lazy.vim](https://github.com/folke/lazy.nvim)一般就是默认使用`setup`来进行初始化插件,

现在再改一下`init.lua`文件

```lua
local default_config = {
    name = "狗蛋"
}

local M = {
    config = default_config
}
function M.say_lowb()
    print("hello lowb "..M.config.name)
end



function M.setup(config)
    M.config = config
    vim.print(M)
end

return M

```

保存退出,重新使用命令:`nvim -c "set rtp+=~/temp/neovim"` 启动`neovim`,

进入命令模式执行以下命令

```vim
lua require('lowb').setup({name=finger})
lua require('lowb').say_lowb()

```
就会看到打印的信息从"狗蛋"更新为"finger"



现在一个简单的`neovim`插件就开发完成了,至于能做什么

**想象力越大,能力越大**
