---

layout: post
title: vimrc 配置
author: DyXu
permalink: /2016/1/vimrc.html
categories:
  - LINUX
  - tags:
  -   - vim

---
VIMRC
=============================================

Intro
---------------------------------------------

My personal vim configuration (.vimrc and .vim) for terminal.

Screenshot
---------------------------------------------
### Source Insight

![source insight](https://github.com/dyxu/vimrc/blob/master/screenshot/source\ insight.png)

### AutoComplete

![Autocomplete](https://github.com/dyxu/vimrc/blob/master/screenshot/YCM.png)


Installation
--------------------------------------------
### Ubuntu Linux Installation

This vim configuration includes a lot of great plugins, configurations and color schemes that
make VIM a lot better. To install it simply do following:

    cd ~
    git clone https://github.com/dyxu/vimrc.git

You'd better backup your vim configuration.

    mv .vim .vim.bak
    mv .vimrc .vimrc.bak

Then

    cp ./vim/.vim ./
    cp ./vim/.vimrc .vimrc

The vimrc is already setup, have fun!

### Errors In Installation

#### ctags or cscope

ctags and cscope should be installed for some plugins (SrcExpl and ccvext). so install them:

    sudo apt-get install ctags
    sudo apt-get install cscope

#### YouCompleteMe

you need reinstall YouCompleteMe for your local environment sometimes.(more details in [YouCompleteMe](https://github.com/Valloric/YouCompleteMe))

Plugins
--------------------------------------------

These plugins are installed in my configurations:

1. [vundle](https://github.com/gmarik/Vundle)
2. [a.vim](https://github.com/vim-scripts/a.vim)
3. [Align](https://github.com/vim-scripts/Align)
4. [auto-pairs](https://github.com/jiangmiao/auto-pairs)
5. [bufexplorer](https://github.com/vim-scripts/bufexplorer.zip)
6. [ccvext](https://github.com/vim-scripts/ccvext.vim)
7. [cSyntaxAfter](https://github.com/vim-scripts/cSyntaxAfter)
8. [indentLine](https://github.com/Yggdroot/indentLine)
9. [Mark--Karkat](https://github.com/vim-scripts/Mark--Karkat)
10. [neocomplcache](https://github.com/Shougo/neocomplcache.vim)
11. [nerdcommenter](https://github.com/scrooloose/nerdcommenter)
12. [nerdtree](https://github.com/scrooloose/nerdtree)
13. [vim-powerline](https://github.com/Lokaltog/vim-powerline)
14. [repeat](https://github.com/vim-scripts/repeat.vim)
15. [snipmate](https://github.com/msanders/snipmate.vim)
16. [SrcExpl](https://github.com/wesleyche/SrcExpl)
17. [tagbar](https://github.com/majutsushi/tagbar)
18. [taglist](https://github.com/vim-scripts/TagList.vim)
19. [TxtBrowser](https://github.com/vim-scripts/TxtBrowser)
20. [vim-fugitive](https://github.com/vim-scripts/vim-fugitive)
21. [YouCompleteMe](https://github.com/Valloric/YouCompleteMe)

more details see [.vimrc](https://github.com/dyxu/vimrc/blob/master/.vimrc)

Key Mappings
--------------------------------------------

* **a.vim** switch between .c and .h.
    * :A  (switches to the header file corresponding to the current file being edited (or vise versa))
    * :AS (splits and switches)
    * :AV (vertical splits and switches)
* **bufexplorer** quickly and easily switch between buffers.
    * \be (normal open)
    * \bs (force horizontal split open)
    * \bv (force vertical split open)
* **switch between windows**

        noremap <c-k> <c-w>k
        noremap <c-j> <c-w>j
        noremap <c-h> <c-w>h
        noremap <c-l> <c-w>l

* **Mark--Markat** highlight several words in different colors simultaneously, similar to the built-in 'hlsearch'.
    * \m (mark the word under the cursor, similar to the star command)
    * * (jump to the [count]'th next occurrence of a mark)
    * # (jump to the [count]'th previous occurrence of a mark)
* **taglist** provides an overview of the structure of source code files and allows
you to efficiently browse through source code files for different programming languages.
    * tl (open or close (toggle) the taglist window)
* **neocomplcache** provides keyword completion system by maintaining a cache of keywords in the current buffer.
    * &lt;c-n> (Toggle neocomplcache and jump to the next item)
    * &lt;c-p> (Toggle neocomplcache and jump to the previous item)
* **nerdcommenter** quickly and easily comment.
    * \ci (toggles the comment state of the selected line(s) individually)
    * \cm (comments the given lines using only one set of multipart delimiters)
    * \cu (uncomments the selected line(s))
    * \cA (adds comment delimiters to the end of line and goes into insert mode between them)
* **nerdtree** allows you to explore your filesystem and to open files and directories.
    * &lt;F2> (open or close the NERDTree)
* **SrcExpl** a source code explorer that provides context for the currently selected keyword by displaying the
function or type definition or declaration in a separate window.
    * &lt;F3> (toggles the Source Explorer window on and off)
* **mine**

        map <F9> :make<CR>
        imap <F9> <ESC>:make<CR>
        nmap <F5> :!python % <CR>
