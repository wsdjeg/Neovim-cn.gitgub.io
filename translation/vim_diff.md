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
  注意: 这些默认值可以通过`-u NONE` 命令行参数关闭

- 'autoindent'    默认开启
- 'autoread'      默认开启
- 'backspace'     默认值为"indent,eol,start"
- 'complete'      默认不包括 "i"
- 'display'       默认值为"lastline"
- 'encoding'      默认值为"utf-8"
- 'formatoptions' 默认值为"tcqj"
- 'history'       默认值为10000 (the maximum)
- 'hlsearch'      默认开启
- 'incsearch'     默认开启
- 'langnoremap'   默认开启
- 'laststatus'    默认值为2 (statusline is always shown)
- 'listchars'     默认值为"tab:> ,trail:-,nbsp:+"
- 'mouse'         默认值为"a"
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

不同于Vim可以在编译时增减特性，Nvim在编译时总是包括所有的特性。这就像Vim只
用一种"HUGE"发布形式一样(除了Nvim相比而言体积更小)

如果在你的`$PATH`环境变量中有Python解释器, |:python| 和 |:python3|
将一直可用的,并且可以同时被用在独立的插件当中.  如果需要在Nvim中使用Python插件,必须
安装`neovim` pip 包(详见|nvim-python|).

|mkdir()| 行为变化:
1. 如果 /tmp/foo 不存在，并且 /tmp 可以被写入，mkdir('/tmp/foo/bar', 'p', 0700) 
将同时创建附带 0700 权限的 /tmp/foo 和 /tmp/foo/bar。而Vim的 mkdir 将创建 0755权限的/tmp/foo .
2. 如果你尝试通过`'p'`创建一个已存在的目录时,(e.g. mkdir('/', 
   'p')) mkdir() 将静默退出，而在Vim中这是一个错误.
3. 现在，当 mkdir 出错时，mkdir() 错误信息将包括 strerror() 的内容.

在Nvim启动后，'encoding' 不再可被修改.

|string()| 和 |:echo| 行为改变:
1. 对于嵌套的容器结构的递归深度将不再有最大值的限制.
2. |string()| 在处理嵌套容器时将直接失败,而不是等到达到递归的最大深度限制时再报错.
2. 当 |:echo| 遇到重复的容器时 比如 >

       let l = []
       echo [l, l]
<
   它将不再使用 "[...]"作为输出结果的一部分 (之前是: "[[], [...]]", 现在是: "[[], []]"). 
   "..."仅仅被用作递归的容器的输出结果.
3. |:echo| 在打印嵌套的容器时，会在 "..."之后添加 "@level" 用以表示递归的深度: |:echo-self-refer|.  
   类似于 |string()| (尽管它使用"{E724@level}"来格式化输出), 但是这并不可靠,因为 |string()| 接下来会报错退出.
4. 已经被转成字符串的infinite和NaN值，现在可以被用作|str2float()|的参数了，而且可以被转换回之前的值.
5. (内部实现) 在Vim中，输出或者把 VAR_UNKNOWN 转成字符串不会有任何影响， |E908|, 在NeoVim中这会产生一个内部错误.

|json_decode()| 的行为发生了改变:
1. 可能会返回 |msgpack-special-dict|.
2. 即使在有重复键的情况下，也会返回|msgpack-special-dict|而不是像Vim中一样报错退出.
3. 只接受合法的JSON格式，末尾的,不被接受

|json_encode()| 的行为有了小的变动：现在它可以接受|msgpack-special-dict|作为输入，但|v:none|不再被接受.


*v:none* 不存在于Nvim中. 在Vim中，它表示"[,]"由|js_decode()|返回的JS空值。

Viminfo 文本文件 被替换为 binary (messagepack) ShaDa files. 
附加的区别:

- |shada-c| 没有影响.
- |shada-s| 目前限制每一个项目的大小,并且不仅仅是寄存器.
- 当写入ShaDa 文件时,项目是根据时间戳来合并的. 
  |shada-merging|
- 'viminfo' 选项已被重命名为'shada'. 为了兼容性,老的选项作为别名被保留.
- |:wviminfo| 被改名为|:wshada|, |:rviminfo| 改为|:rshada|.老的命令仍然保留着.
- |:oldfiles| 支持 !.
- 当写入时(|:wshada| without bang or at exit) 它将根据时间戳合并更多的数据, 
  而Vim 只能合并标记.
  |shada-merging|
- ShaDa file 格式设计遵循了向前和向后兼容. |shada-compatibility|
- 在发生错误时，ShaDa相关的代码将在原地保留暂存文件,并由用户决定该如何操作. 而这种情况下Vim将删除暂存文件. 
  |shada-error-handling|
- Vim 在所有时候都不会保存时间戳：在viminfo中与自身实例中都不会.
- ShaDa 文件保持搜索的方向 (|v:searchforward|), viminfo 不支持.

==============================================================================
4. 新特性						     *nvim-features-new*

详见 |nvim-intro| 列出了Nvim的大量的新特性.

|bracketed-paste-mode| is built-in 并且默认开启.

Meta (alt) 按键的识别(即使在终端中也能被识别).
  <M-1>, <M-2>, ...
  <M-BS>, <M-Del>, <M-Ins>, ...
  <M-/>, <M-\>, ...
  <M-Space>, <M-Enter>, <M-=>, <M-->, <M-?>, <M-$>, ...

  注意: Meta 按键是区分大小写的(<M-a> 不同于 <M-A>).

一些 `CTRL-SHIFT-...` 按键相当于`CTRL-...` 的变异(甚至于在终端中). 特别的, 已知以下组合可以正常工作:
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

这些遗失的Vim特性将来可能被实现,但这些并不在近期的发布计划中.

- vim.bindeval() (Vim7.4新特性)
- |if_ruby|
- |if_lua|
- |if_perl|
- |if_mzscheme|
- |if_tcl|

==============================================================================
6. 移除的特性					 *nvim-features-removed*

Vim拥有这些特性,但是Nvim已经决定移除它们。

Vi-compatible 模式:
  ":set nocompatible" 将被忽略
  ":set compatible" 将产生错误

Ed-compatible 模式:
  ":set noedcompatible" 将被忽略
  ":set edcompatible" 将产生错误

'ttyfast':
  ":set ttyfast" 将被忽略
  ":set nottyfast" 将产生错误

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
  'cpoptions' ('g', 'w', 'H', '*', '-', 'j', 和全部的 POSIX 标记均已移除)
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
  'termencoding' (Vim 7.4.852 也在Windows版本中移除了这个选项)
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

其它编译期特性:
  EBCDIC
  Emacs tags文件的支持
  X11的集成 (参见 |x11-selection|)

Nvim 没有内置的GUI,因此这些命令映射已被移除: gvim, gex, gview, rgvim, rgview

"Easy mode" (eview, evim, nvim -y)
"(g)vimdiff" (是 "(g)nvim -d"的别名 |diff-mode|)
"Vi mode" (nvim -v)

通过别名启动nvim的方法已经被去除了，现在应该利用命令行选项启动:

  ex        nvim -e
  exim      nvim -E
  view      nvim -R
  rvim      nvim -Z
  rview     nvim -RZ

==============================================================================
 vim:tw=78:ts=8:noet:ft=help:norl:
```
###[Upstream](https://github.com/neovim/neovim/blob/master/runtime/doc/vim_diff.txt)
