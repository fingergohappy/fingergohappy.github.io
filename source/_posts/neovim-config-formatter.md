---
title: neovim配置formatters
tags:
  - neovim
categories:
  - neovim奇技淫巧
cover: https://source.unsplash.com/gray-laptop-computer-turned-on-EaB4Ml7C7fE/1200x628
date: 2023-11-01 16:05:14
abstracts: neovim配置formatters
---

最近再搞`vue3`,使用了[element plus admin](https://element-plus-admin.cn/)这个模版库,没想到一直报错
后来经过排查发现是这个模板库使用了[prettier](https://prettier.io/)这位`lint`的配置项,简单来说就是如果不符合他的格式就会报错,
而我的`neovim`是使用的`volar`这个`lsp`作为格式化话的后端,想要保存自动安装`prettier`的格式格式化,需要额外加一些配置

本来是想打算用[null-ls](https://github.com/jose-elias-alvarez/null-ls.nvim),因为更符合哲学的,其实就是将一些`formatter`作为`neovim`原生的`lsp`inject进去
不过这个项目目前停更了,而且我也只是想保存的时候格式化就行了,所以就选择了另一个插件[formatter.nvim](https://github.com/mhartington/formatter.nvim)


# 安装


## 安装prettier

在`mason`中找到`prettier`安装就行
不过这个地方比较有意思,`mason`其实是安装了`prettier`的,按时我在终端中缺无法使用,后来发现`mason`安装的工具其实是在`~/.local/share/nvim/mason/bin/`里面,然后在放到`neovim`内置的环境变量中是,在`neovim`的命令模式执行`:!env`就可以看到

## 安装formatter.nvim

```lua
    {
            "mhartington/formatter.nvim",
            name = "formatter",
                event = "BufEnter",
                enabled = true,
                # 这个配置一定要写到config选项中,不然会报错
                config = function(LazyPlugin,opts)
                local opts = {
                    logging = true,
                    log_level = vim.log.levels.WARN,
                    filetype = {
                        vue = {
                            require("formatter.filetypes.vue").prettier(),
                            _
                            }
                    }
                }
                require("formatter").setup(opts)
            end
        }

```

这里需要注意的点是
1. formatter并没用给默认的格式化配置,所以没中文件都要配置
2. 配置想要写到`lazyvim`的`config`选项中,不然会提示找不到`formatter.filetypes.vue`


# 保存自动格式化

```lua
vim.api.nvim_create_autocmd({"BufWritePost"},{pattern = {"*"},command = "FormatWrite"})

```

# 原理

`formatter.nvim`相当于一个接口,然后自己配置格式化命令,当执行格式化命令时,会调用你配置的格式化命令,如果不配置就是使用默认的,想上面的`vue`执行格式化的时候其实就是在执行默认的格式化,约等于下面的伪代码:

```shell
prettier --stdin-filepath %
```
