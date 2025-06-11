---
title: vscode更好用的vim插件
tags:
  - vim
  - vscode
categories:
  - 万物皆可vim
cover: https://source.unsplash.com/oA7MMRxTVzo/1200x628
date: 2023-09-23 19:10:04
abstracts: vscode更好用的vim插件:Neovim
toc: true
---



# 更好用的vim插件

之前在 `vscode`中使用的 `vim`插件一直是[vim](https://marketplace.visualstudio.com/items?itemName=vscodevim.vim),但是这个插件有一个让我非常苦恼的问题,
就是这个插件没有`neovim`中的`flash`插件,它默认的快速移动插件是`easymotion`,个人觉得,`easymotion`这个插件的已送快捷键方式实在是太不方便,举个简单的例子:
他的`prefix`是要按两下`leader`按键,而`flash`插件就十分的方便,按下`s`然后输入词就可以快速搜索,这点就是很舒服的地方,下面两个图
<!--more-->

`flash`

![flash](flash-demo.gif)

`easy-motion`

![easymotion](easy-motion.gif)

另外我也没有在`vim`插件的文档中找到如何更改`easymotion`的快捷键的地方,如果可以改的话也可以进行配置(我的`intellij idea`就是通过这个更改实现类`flash`模式)

而`vscode`中的[neovim](https://marketplace.visualstudio.com/items?itemName=asvetliakov.vscode-neovim)插件可以使用`neovim`的原生插件,这一点就非常舒服了,
这个插件的基本原理大概就是:使用`vscode`去连接一个`neovim`实例,然后在`command`和`normal`模式下,将用户输入的命令直接传给`neovim`



# 配置

安装就不多说了,有手就行

有一些需要注意的地方是:

1. 需要指定`neovim`的安装位置
2. 如果需要个性化配置,可以制定一个`init.lua`/`init.vim`文件


具体配置选项可以看[这里](https://github.com/vscode-neovim/vscode-neovim)


# 我的配置


```lua
--                              _                   _
--                      _ __   | |  _   _    __ _  (_)  _ __
--                     | '_ \  | | | | | |  / _` | | | | '_ \
--                     | |_) | | | | |_| | | (_| | | | | | | |
--                     | .__/  |_|  \__,_|  \__, | |_| |_| |_|
--                     |_|                  |___/
--

local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  -- bootstrap lazy.nvim
  -- stylua: ignore
  vim.fn.system({ "git", "clone", "--filter=blob:none", "https://github.com/folke/lazy.nvim.git", "--branch=stable", lazypath })
end
vim.opt.rtp:prepend(vim.env.LAZY or lazypath)


require('lazy').setup({

    {
        "kylechui/nvim-surround",
        version = "*", -- Use for stability; omit to use `main` branch for the latest features
        event = "VeryLazy",
        config = function()
            require("nvim-surround").setup({
                -- Configuration here, or leave empty to use defaults
            })
        end
    },
    {
        "folke/flash.nvim",
        event = "VeryLazy",
        ---@type Flash.Config
        opts = {},
        -- stylua: ignore
        keys = {
            { "s", mode = { "n", "o", "x" }, function() require("flash").jump() end, desc = "Flash" },
            { "S", mode = { "n", "o", "x" }, function() require("flash").treesitter() end, desc = "Flash Treesitter" },
            -- { "r", mode = "o", function() require("flash").remote() end, desc = "Remote Flash" },
            { "R", mode = { "o", "x" }, function() require("flash").treesitter_search() end, desc = "Treesitter Search" },
            { "<c-s>", mode = { "c" }, function() require("flash").toggle() end, desc = "Toggle Flash Search" },
        },
    }
})

--                                                     _
--                   __   __  ___    ___    ___     __| |   ___
--                   \ \ / / / __|  / __|  / _ \   / _` |  / _ \
--                    \ V /  \__ \ | (__  | (_) | | (_| | |  __/
--                     \_/   |___/  \___|  \___/   \__,_|  \___|
--
--                                             __   _
--                      ___    ___    _ __    / _| (_)   __ _
--                     / __|  / _ \  | '_ \  | |_  | |  / _` |
--                    | (__  | (_) | | | | | |  _| | | | (_| |
--                     \___|  \___/  |_| |_| |_|   |_|  \__, |
--                                                      |___/
--
local map = vim.keymap.set
local expr_options = { expr = true, silent = true }

-- 这里很重要,防止光标到折叠为止,折叠自动展开
map("n","k","gk",{remap = true})
map("n","j","gj",{remap = true})

map('n','zc',"<cmd>call VSCodeCall('editor.fold')<cr>")
map('n','zo',"<cmd>call VSCodeCall('editor.unfold')<cr>")
map('n','zR',"<cmd>call VSCodeCall('editor.unfoldAll')<cr>")
map('n','zM',"<cmd>call VSCodeCall('editor.foldAll')<cr>")

map('n','<leader>e',"<cmd>call VSCodeCall('workbench.action.quickOpenNavigateNextInFilePicker')<cr>",{remap=true})

```

**注意**

```lua

-- 这里很重要,防止光标到折叠为止,折叠自动展开
map("n","k","gk",{remap = true})
map("n","j","gj",{remap = true})
```

这两句配置很是很重要的,如果不配置,在`neovim`模式下,光标移动到折叠为止,折叠会自动展开




