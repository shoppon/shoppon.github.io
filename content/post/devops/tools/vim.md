---
title: "vim相关"
categories: ["开发工具", "效率"]
tags: ["vim"]
date: 2020-02-28T17:05:23+08:00
---

# 插件

# 快捷键

# 技巧

- 跳到指定行`:linenum`
- 复制多行`:start, end copy dest`
- 保存退出`ZZ`
- 重复操作`.`
- 大量删除`:start, end delete/d/del`
- 格式化`:G`

# 配置
```shell
autocmd FileType text setlocal textwidth=120
filetype indent on
filetype on
filetype plugin on
imap <c-h> <Left>
imap <c-j> <Down>
imap <c-k> <Up>
imap <c-l> <Right>
let mapleader=","
map <a-0> 10gt
map <a-1> 1gt
map <a-2> 2gt
map <a-3> 3gt
map <a-4> 4gt
map <a-5> 5gt
map <a-6> 6gt
map <a-7> 7gt
map <a-8> 8gt
map <a-9> 9gt
map tc :tabclose <CR>
map te :tabnew <CR>
map tn :tabnext <CR>
map tp :tabprevious <CR>
nmap <leader>e :e $VIM/_vimrc<cr>
nnoremap <c-h> <c-w>h
nnoremap <c-j> <c-w>j
nnoremap <c-k> <c-w>k
nnoremap <c-l> <c-w>l
nnoremap <space> @=((foldclosed(line('.')) < 0) ? 'zc' : 'zo')<CR>
set autochdir
set autoindent
set backspace=2
set encoding=utf-8
set fdl=5
set fileencodings=utf-8,gb2312,gbk,latin-1
set foldenable
set foldmethod=indent
set formatoptions=tcrqn
set guifont=Lucida_Console:h12
set guioptions-=m
set guioptions-=T
set hlsearch
set ignorecase smartcase
set incsearch
set laststatus=2
set mouse=a
set nobackup
set nocompatible
set nofen
set noswapfile
set number
set ruler
set shiftwidth=4
set showcmd
set showmatch
set showmode
set smartindent
set smarttab
set softtabstop=4
set statusline=%F%m%r%h%w\ [FORMAT=%{&ff}]\ [TYPE=%Y]\ [ASCII=\%03.3b]\ [HEX=\%02.2B]\ [POS=%04l,%04v][%p%%]\ [LEN=%L]
set tabstop=4
set wildignore+=.hg,.git,.svn
set wildignore+=.metadata
source $VIMRUNTIME/delmenu.vim
source $VIMRUNTIME/menu.vim
syntax on
```
