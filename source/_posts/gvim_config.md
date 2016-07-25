---
title: gvim常用插件的安装及配置
tags: [ubuntu, linux]
---

   之前一直在windows下coding，windows下强大的IDE使用起来得心应手，一旦转到ubuntu下会渴望有一个强大的IED来增加生产效率，使用了一段时间的atom后，在崩溃了多次后触碰了我的底线，基本已经被我打入了冷宫。所以决定拾取vim，查阅一番资料打造一个适合自己的IDE，好废话不多说，let's do it.
## 语法高亮
   写代码没有语法高亮那将是一种折磨，这个只需要在vim的资源文件中进行配置即可
``` bash
syntax enable
syntax on
colorscheme desert
```
   在~/.vimrc文件中增加下面内容即可。
<!-- more -->

## 程序跳转
   程序跳转就要用到tags文件了，tags文件需要提前生成，如何生成呢，这里就要用到ctags了，首先安装这个软件
``` bash
sudo apt-get install exuberant-ctags
```
   代码根目录下生成tags文件，这时候在代码根目录下就会生成一个tags文件。
``` bash
ctags -R
```
   如何使用这个tags文件呢，vim打开一个代码文件，将光标移动到一个调用函数上，<C+]>进行函数声明，<C+t>回到调用位置。

## 文件标签列表
   插件：TagList，下载地址：http://www.vim.org/scripts/script.php?script_id=273
   ~/.vimrc中添加如下内容：
``` bash
let Tlist_Show_One_File=1
let Tlist_Exit_OnlyWindow=1
```

## 文件浏览器窗口管理窗口
   插件：WinManager，下载地址：http://www.vim.org/scripts/script.php?script_id=95
``` bash
let g:winManagerWindowLayout='FileExplorer|TagList'
nmap wm :WMToggle<cr>
```

## 全局搜索
   插件：Grep，下载：http://www.vim.org/scripts/script.php?script_id=311
``` bash
nnoremap <silent> <F3> :Grep<CR>
```
