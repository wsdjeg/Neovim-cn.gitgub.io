```doc
*vim_diff.txt*   关于 Nvim.						{Nvim}


			    NVIM 参考指南


Nvim 和 Vim 之间的区别					       *vim-differences*

在本篇帮助文档中,Nvim和Vim之间的区别通过{Nvim}着一个标签来指名.这篇文档集中完整
列出了所有的不同点.
1. 配置			 	|nvim-configuration|
2. 默认值			|nvim-defaults|
3. 变更的特新			|nvim-features-changed|
4. 新特性New 			|nvim-features-new|
5. 缺少的特性			|nvim-features-missing|
6. 移除的特性			|nvim-features-removed|


==============================================================================
1. 配置							    *nvim-configuration*

- 使用 `$XDG_CONFIG_HOME/nvim/init.vim` 替代 `.vimrc` 存储配置信息.
- 使用 `$XDG_CONFIG_HOME/nvim` 替代 `.vim` 存储配置文件.
- 使用 `$XDG_DATA_HOME/nvim/shada/main.shada` 替代 `.viminfo` 用以存储持久session信息.

==============================================================================
2. 默认值					            *nvim-defaults*

- 默认开启语法高亮
- 默认启用文件类型相关的插件和脚本
  注意: 这些默认值可以通过`-u NONE` 命令行参数

- 'autoindent'    默 认开启
- 'autoread'      默 认开启
- 'backspace'     默 认值为"indent,eol,start"
- 'complete'      不包括 "i"
- 'display'       默 认值为"lastline"
- 'encoding'      默 认值为"utf-8"
- 'formatoptions' 默 认值为"tcqj"
- 'history'       默 认值为10000 (the maximum)
- 'hlsearch'      默 认开启
- 'incsearch'     默 认开启
- 'langnoremap'   默 认开启
- 'laststatus'    默 认值为2 (statusline is always shown)
- 'listchars'     默 认值为"tab:> ,trail:-,nbsp:+"
- 'mouse'         默 认值为"a"
- 'nocompatible'  一直开启着
- 'nrformats'     默认值为"bin,hex"
- 'sessionoptions' 不包括 "options"
- 'smarttab'      默认开启
- 'tabpagemax'    默认值为50
- 'tags'          默认值为"./tags;,tags"
- 'ttyfast'       一直开启
- 'viminfo'       包括 "!"
- 'wildmenu'      默认开启

==============================================================================
3. 变更的特性					 *nvim-features-changed*

Nvim 编译时通常包括所有的特性, 不同于Vim可以在编译时增减特性  这就像当与Vim只
用一种"HUGE"发布形式(除此之外,Nvim相比而言跟嘉小巧)

如果在你的`$PATH`环境变量中有Python解释器, |:python| 和 |:python3|
将一直可用的,并且可以被用在独立的插件当中.  如果需要在Nvim中使用Python插件,必须
安装`neovim` pip 包(详见|nvim-python|).

|mkdir()| 行为变化:
1. 假设 /tmp/foo 不存在 并且 /tmp 可以被写入 mkdir('/tmp/foo/bar', 'p', 0700) 
这将同时创建 /tmp/foo 和 /tmp/foo/bar 附带 0700 权限 Vim mkdir 将创建 /tmp/foo with 0755.
2. 如果你尝试通过`'p'`创建一个已存在的目录时,(e.g. mkdir('/', 
   'p')) mkdir() 将静默推出 在Vim中这将是一个错误.
3. mkdir() 错误信息目前包括 strerror() 文本当 mkdir 出错时.

'encoding' 不可以在Nvim启动后进行变更.

|string()| 和 |:echo| 行为改变:
1. 对于嵌套的容器结构的递归深度将不再有最大值的限制.
2. |string()| 在处理嵌套容器是立即衰退,而不是当递归限制溢出时.
2. 当 |:echo| 遇到 复制的容器 比如 >

       let l = []
       echo [l, l]
<
   它将不再使用 "[...]" (之前是: "[[], [...]]", 现在是: "[[], []]"). "..."
   仅仅被用作递归的容器.
3. |:echo| 打印 嵌套的容器在 "..."之后添加 "@level" 用以表述递归的水平: |:echo-self-refer|.  
   类似于 |string()| (尽管它使用这样的构想"{E724@level}"), 但是这并不可靠,因为 |string()| 继续错误输出.

Viminfo 文本 文件 被替换为 binary (messagepack) ShaDa files. 
附加的区别:

- |shada-c| 没有影响.
- |shada-s| 目前限制每一个项目的大小,并且不仅仅是寄存器.
- 当写入ShaDa 文件时,项目是根据时间戳来合并的. 
  |shada-merging|
- 'viminfo' 选项已被重命名为'shada'. 为了兼容性,老的选项被保留作为别名.
- |:wviminfo| 被改名为|:wshada|, |:rviminfo| 改为|:rshada|.老的命令仍然保留着.
- |:oldfiles| 支持!.
- 当写入时(|:wshada| without bang or at exit) 它将合并更多的数据, 
  并且更具时间戳来进行操作.  Vim 仅仅合并标记.
  |shada-merging|
- ShaDa file 格式设计遵循了向前和向后兼容. |shada-compatibility|
- 一些错误使得ShaDa code 暂停,并由用户决定该如何操作.  这种情况下Vim删除暂存文件. 
  |shada-error-handling|
- Vim 遵循的完全非时间戳, 不仅是viminfo 而且自身实例中也是这样.
- ShaDa 文件保持这搜索方向 (|v:searchforward|), viminfo 不支持.

==============================================================================
4. 新特性						     *nvim-features-new*

详见 |nvim-intro| 列出了Nvim的大量的新特性.

|bracketed-paste-mode| is built-in 并且默认开启.

Meta (alt) 按键的识别(包括在终端模拟器中).
  <M-1>, <M-2>, ...
  <M-BS>, <M-Del>, <M-Ins>, ...
  <M-/>, <M-\>, ...
  <M-Space>, <M-Enter>, <M-=>, <M-->, <M-?>, <M-$>, ...

  注意: Meta 按键是区分大小写的(<M-a> 区别于 <M-A>).

一些 `CTRL-SHIFT-...` 按键相当于`CTRL-...` 的变异(甚至于在终端中). 特别的, 已知一下组合可以正常工作:
  <C-Tab>, <C-S-Tab>
  <C-BS>, <C-S-BS>
  <C-Enter>, <C-S-Enter>

事件:
  |TabNew|
  |TabNewEntered|
  |TabClosed|
  |TermOpen|
  |TermClose|

高亮组:
  |hl-EndOfBuffer|
  |hl-TermCursor|
  |hl-TermCursorNC|

==============================================================================
5. 缺少的特性				 *nvim-features-missing*
				     *if_ruby* *if_lua* *if_perl* *if_mzscheme* *if_tcl*

这些遗失的Vim特性将来可能被实现,单这并不计划在近期的里程碑中.

- vim.bindeval() (Vim7.4新特性)
- |if_ruby|
- |if_lua|
- |if_perl|
- |if_mzscheme|
- |if_tcl|

==============================================================================
6. 移除的特性					 *nvim-features-removed*

Vim拥有这些特性,但是已经刻意从Nvim中移除 Nvim.

Vi-compatible 模式:
  ":set nocompatible" 将被忽略
  ":set compatible" 将产生错误

Ed-compatible mode:
  ":set noedcompatible" 被忽略的
  ":set edcompatible" 错误

'ttyfast':
  ":set ttyfast" 忽略
  ":set nottyfast" 错误

加密支持:
  'cryptmethod'
  'key'

MS-DOS 支持:
  'bioskey'
  'conskey'

高亮组:
  |hl-VisualNOS|

其他选项:
  'antialias'
  'cpoptions' ('g', 'w', 'H', '*', '-', 'j', and all POSIX flags were removed)
  'guioptions' (只有't' 标记被移除)
  'guipty'
  'imactivatefunc'
  'imactivatekey'
  'imstatusfunc'
  'macatsui'
  'restorescreen'
  'shelltype'
  'shortname'
  'swapsync'
  'term'
  'termencoding' (Vim 7.4.852 also removed this for Windows)
  'textauto'
  'textmode'
  'toolbar'
  'toolbariconsize'
  'ttybuiltin'
  'ttymouse'
  'ttyscroll'
  'ttytype'
  'weirdinvert'

其他命令:
  :Print
  :fixdel
  :helpfind
  :mode (不再接受参数)
  :open
  :shell
  :tearoff

Other compile-time features:
  EBCDIC
  Emacs tags support
  X11 integration (see |x11-selection|)

Nvim 没有内置的GUI,因此这些映射已被移除: gvim, gex, gview, rgvim, rgview

"Easy mode" (eview, evim, nvim -y)
"(g)vimdiff" (alias for "(g)nvim -d" |diff-mode|)
"Vi mode" (nvim -v)

The ability to start nvim via the following aliases has been removed in favor
of just using their command line arguments:

  ex        nvim -e
  exim      nvim -E
  view      nvim -R
  rvim      nvim -Z
  rview     nvim -RZ

==============================================================================
 vim:tw=78:ts=8:noet:ft=help:norl:
```
###[Upstream](https://github.com/neovim/neovim/blob/master/runtime/doc/vim_diff.txt)
