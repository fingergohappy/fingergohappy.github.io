---
title: idea下配置vim模式
tags:
  - idea
  - vim
categories:
  - 万物皆可vim
cover: https://source.unsplash.com/Bss5nhYnLKU/1200x628
date: 2023-09-22 17:04:46
abstracts: 如何在idea中配置vim,让写代码更加舒服
---


# vim搓一切?

有一说一,vim虽然可以配置的很舒服,但是对于`java`程序员来说,用vim去写`java`是真的折磨,除非你想进行自我折磨,变成代码仙人.
所以最省心的方案就是在`idea`中下一个vim插件



#  安装


首先你要有一个 **java第一** IDE:`intellij idea`
然后打开插件市场,下一个idea vimrc 安装上


# 配置`.ideavimrc`

创建一个`~/.ideavimrc`

加入一些配置:

```vim
"                   ____       _      ____    ___    ____
"                  | __ )     / \    / ___|  |_ _|  / ___|
"                  |  _ \    / _ \   \___ \   | |  | |
"                  | |_) |  / ___ \   ___) |  | |  | |___
"                  |____/  /_/   \_\ |____/  |___|  \____|


" 上下预留5行
set so=5
" 不折叠 set nowrap
" 去掉哔哔声
set belloff=all
set noerrorbells
set t_vb=
set backspace=indent,eol,start whichwrap+=<,>,[,]
" Vim 的默认寄存器和系统剪贴板共享
set clipboard+=unnamedplus
set keep-english-in-normal
" 开启相对行
set relativenumber
set nu
set ignorecase
set smartcase


"                   ____    _       _   _    ____   ___   _   _
"                  |  _ \  | |     | | | |  / ___| |_ _| | \ | |
"                  | |_) | | |     | | | | | |  _   | |  |  \| |
"                  |  __/  | |___  | |_| | | |_| |  | |  | |\  |
"                  |_|     |_____|  \___/   \____| |___| |_| \_|
"
"

"开启多光标支持
Plug 'machakann/vim-highlightedyank'
Plug 'vim-scripts/argtextobj.vim'
Plug 'justinmk/vim-sneak'
Plug 'easymotion/vim-easymotion'
Plug 'unblevable/quick-scope'
Plug 'preservim/nerdtree'
Plug 'tpope/vim-commentary'


"USAGE 
" add surround
" ys{textobject motion}{indeifier}
" eg:
"   ysiw'
"
" remove surround 
" ds'
"
" change surround 
" cs[(

Plug "tpope/vim-surround"
"USAGE 
" [count]["x]gr{motion}   Replace {motion} text with the contents of register x.
"                         Especially when using the unnamed register, this is
"                         quicker than "_d{motion}P or "_c{motion}<C-R>"
" [count]["x]grr          Replace [count] lines with the contents of register x.
"                         To replace from the cursor position to the end of the
"                         line use ["x]gr$
" {Visual}["x]gr          Replace the selection with the contents of register x."

Plug 'vim-scripts/ReplaceWithRegister'
" USAGE 
" cx
" On the first use, define the first {motion} to exchange. On the second use, define the second {motion} and perform the exchange.
"
" cxx
" Like cx, but use the current line.
"
" X
" Like cx, but for Visual mode.
"
" cxc
" Clear any {motion} pending for exchange.
"

Plug 'tommcdo/vim-exchange'
Plug 'terryma/vim-multiple-cursors'
" quick-scop cofnig
let g:qs_highlight_on_keys = ['f', 'F', 't', 'T']
let g:qs_accepted_chars = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z'] 




" https://github.com/JetBrains/ideavim/wiki/ideajoin-examples
set ideajoin
set sneak


"                   _  __  _____  __   __  __  __      _      ____
"                  | |/ / | ____| \ \ / / |  \/  |    / \    |  _ \
"                  | ' /  |  _|    \ V /  | |\/| |   / _ \   | |_) |
"                  | . \  | |___    | |   | |  | |  / ___ \  |  __/
"                  |_|\_\ |_____|   |_|   |_|  |_| /_/   \_\ |_|

" basic key map 
inoremap <C-e> <END>
inoremap <C-a> <HOME>
inoremap <C-f> <Right>
inoremap <C-b> <Left>
inoremap <C-n> <Down>
inoremap <C-p> <Up>
inoremap <M-f> <S-Right>
inoremap <M-b> <S-Left>
inoremap <C-k> <ESC>d$i
inoremap <C-d> <Del>


let mapleader=" "

" idea action config
nnoremap gd <Action>(GotoDeclaration)
nnoremap gi <Action>(GotoImplementation)
nnoremap gs <Action>(GotoSuperMethod)
nnoremap gt <Action>(GotoTypeDeclaration)

map <leader>re  <Action>(RenameElement)

map <leader>gf <Action>(GotoFile)
map <leader>gt <Action>(GotoSymbol)


map <leader>f: <Action>(GotoAction)
map <leader>f. <Action>(FindInPath)
map <leader>fl <Action>(RecentLocations)
map <leader>ff  <Action>(RecentFiles)


map <leader>et :NERDTreeToggle<CR>
map <leader>ef :NERDTreeFocus<CR>
map <leader>es <Action>(FileStructurePopup)


nmap <C-W>q <Action>(CloseAllEditors)


" plugin config 

let g:multi_cursor_use_default_mapping=0

" Default mapping
let g:multi_cursor_start_word_key      = '<C-n>'
let g:multi_cursor_select_all_word_key = '<A-n>'
let g:multi_cursor_start_key           = 'g<C-n>'
let g:multi_cursor_select_all_key      = 'g<A-n>'
let g:multi_cursor_next_key            = '<C-n>'
let g:multi_cursor_prev_key            = '<C-p>'
let g:multi_cursor_skip_key            = '<C-x>'
let g:multi_cursor_quit_key            = '<Esc>'



```

# 让其生效

打开idea,source一下

```bash
source ~/.ideavimrc
```

# FBI WARNING


1. `.ideavimrc`里面有一些插件是要在idea中也要下载对应的插件才能使用的,具体参考[这里](https://github.com/JetBrains/ideavim/wiki/IdeaVim-Plugins)
2. `map <leader>f: <Action>(GotoAction)` 这种形式是在调用`idea`的原生功能,具体有哪些功能,可以参考[这里](https://github.com/JetBrains/ideavim),可以打开`ideavim`的插件,查看对应操作的`<ActionId>`

![track_action_id](track_action_dark.gif)


