Title: 我的vim配置
Date: 2015-10-14
Category: Pages
Tags: python, vim

>『工欲善其事，必先利其器。』

## 自己家的配置才是最好的配置

vim作为一个高度定制的文本编辑工具，怎么配置好vim决定了vim的功能是否强大，但是同样作为文本编辑工具本身，过多不必要的配置不仅会让使用者无所适从，而且还拖慢了编辑器的速度，如何在这之间找到平衡完全取决于使用者的需求。

所以，和*别人家的孩子都是好孩子*不同，**自己家的配置才是最好的配置**。所以并不推荐直接使用github上其他人的配置，但是他人的推荐却可以帮助了解更多可能有效的配置。我现在使用的配置是参考[**k-vim**](https://github.com/wklken/k-vim)的配置，并且修改不适合自己的内容后（比如使用方向键等）最终感觉最顺手的配置，如果有需要，可以参考。

## 搜索、折叠、缩进、注释

这些操作都是比较刚需的操作，并且vim已经自带这些功能。下面是一个针对python的设置示例，以及每条功能的注解：

### 1. 搜索

在普通模式下输入`/`即可搜索，以下配置可以更加方便地搜索：

```vim
set showcmd		" 在右下角显示输入的命令
set showmode		" 在左下角显示当前的模式
set hlsearch		" 高亮搜索结果
set ignorecase		" 忽略大小写
set smartcase		" 当输入的内容有一个以上大写字母时，不忽略大小写
set incsearch		" 设置该参数后，输入的同时会即使显示搜索结果，个人感觉看着晕
```

### 2. 折叠

```vim
" 折叠相关命令
" zM 折叠所有
" zR 打开所有
" zc 折叠当前
" zo 打开当前
set foldenable		" 开启折叠
set foldmethod=indent		" 使用缩进折叠
set foldlevel=10		" 打开文件时只折叠超过10层以上的
```

更加详细的设置方法，比如其他折叠类型等，很容易在网上可以查到。

### 3. 缩进

Python的缩进非常重要，所以下面的设置也非常重要：

```vim
set smartindent		" 智能缩进
set autoindent		" 自动缩进
set smarttab
set expandtab		" 将tab转为空格，重要！建议看下文说明
set tabstop=4		" 设置tab键宽度为4
set softtabstop=4		" 设置退格键删除4个空格
set shiftwidth=4		" 设置每次缩进对应4个空格
set shiftround		" 缩进时取整
```

在Python中，使用4个空格比使用tab更加可靠（见PEP8），但是在编辑其他文件，特别是csv文件时，将tab自动转成空格将会导致更多错误，所以，更加可靠的做法是，只在编辑Python文件时，才启用`expandtab`，如：`autocmd FileType python set expandtab`，默认配置下，vim检查文件类型只会用默认后缀来识别，如果希望添加更多的后缀或者其他正则等，可以使用如下配置：`autocmd BufRead,BufNew *.cpy set filetype=python`。

在其他编辑器中，往往有比较好用的块缩进功能，在vim中也一样，按一下`v`键进入*visual*模式，然后选中你想要选中的行，按`>`即可块缩进，按`<`则取消一层块缩进，但是默认设置中，每次操作都会退出*visual*模式，设置下面两项，保持*visual*模式：

```vim
vnoremap < <gv
vnoremap > >gv
```

### 4. 注释

无论什么代码，方便的注释功能都是非常重要的，vim中使用块注释的方法为：

1. 块注释 `Ctrl+v`，选中多行后，按`I`，在第一行输入`#`等内容，然后按`Esc`，即可看到所有行都进行了注释。
2. 取消注释，和注释的原理一样，不过这次不是按`I`输入内容，而是选中多行的注释符号后，按`d`统一删除。

## 插件管理

除了vim自带的设置之外，vim社区提供了非常丰富的插件，选择对自己有用的插件安装，插件多了之后，插件管理则显得非常重要。Vundle是比较常用的插件管理工具，其本身也是插件。

从github上clone并且安装或者升级：

```bash
if [ ! -e ~/bundle/vundle ];then
	git clone https://github.com/gmarik/vundle.git ~/bundle/vundle
else
	cd "~/bundle/vundle" && git pull
fi
```
设置以下内容，建议和vimrc分离，单独弄一个文件管理插件：

```vim
filetype off		" 暂时关闭文件类型，必须设置
set rtp+=~/.vim/bundle/vundle
call vundle#rc()
Bundle 'gmarik/vundle'		" 类似于这样，在Bundle 后面跟需要安装的插件名称
" 在用vim编辑该文件时，使用下面的相关命令管理插件
" :BundleInstall		安装配置中的插件
" :BundleInstall!	跟新配置中的插件
" :BundleClean		删除本地无用的插件
```

## 语法高亮、语法检查、格式化

**hdima/python-syntax**插件提供了更加完善的语法高亮功能，**kevinw/pyflakes-vim**插件则提供了完善地Python语法检查，不过语法检查插件更加建议选择**scrooloose/syntastic**，不过不论选择哪一个，都必须安装一个python模块：`pip install pyflakes`，另外，**mindroit101/vim-yapf**插件可以对Python代码进行快速格式化，一键满足PEP8要求，同样需要安装一个python模块：`pip install yapf`。这三个模块的配置如下：

```vim
" 安装使用python-syntax
Bundle 'hdima/python-syntax'
let python_highlight_all = 1		" 所有关键词高亮
```

```vim
" 安装使用syntastic
Bundle 'scrooloose/syntastic'
let g:syntastic_error_symbol='>>'		" 错误行使用'>>' 标记
let g:syntastic_warning_symbol='>'		" 警告行使用'>' 标记
let g:syntastic_check_on_open=1		" 打开文件时即开启语法检查
let g:syntastic_check_on_wq=0		" 保存时进行语法检查
let g:syntastic_enable_highlighting=1	" 提示内容高亮显示
" 设置Python检查规则为pyflakes和pep8
let g:syntastic_python_checkers=['pyflakes', 'pep8']
" 提示内容显示相关
let g:syntastic_always_populate_loc_list = 0
let g:syntastic_auto_loc_list = 0
let g:syntastic_loc_list_height = 5
```

```vim
" 安装使用vim-yapf
Bundle 'mindriot101/vim-yapf'
scriptencoding utf-8		" 设置编码为utf-8
let g:yapf_style = "pep8"		" 设置主题为pep8，或者"google"
nnoremap <leader>y :call Yapf()<cr>		" 设置格式化快捷键为：',y'
```

其中，

`nnoremap` 指在普通模式下的键位映射，这里将`:call Yapf()`映射为`<leader>y`,

<leader>表示在普通模式下的命令头，如设置`let mapleader=','`将其设置为`,`，则，在普通模式下输入`,y`即可将Python文件按照PEP8的建议格式化。

## 导航

使用**scrooloose/nerdtree**插件，可以在vim中方便地使用导航功能，快速打开其他的文件等：

```vim
Bundle 'scrooloose/nerdtree'
" 一些常用的键位
" <leader>n : 打开导航
" Ctrl + ww : 窗口键切换
" <leader>o : 打开选中的文件
" <leader>m : 根据提示新建或者删除文件
map <leader>n :NERDTreeToggle<CR>
let NERDTreeHighlightCursorline=1
let NERDTreeIgnore=['\.pyc$','\.pyo$','\.o$','\.so$','\.egg$'] " 设置需要忽略的文件
```

## 自动补全

自动补全是一项比较复杂的工作，所以需要设置的东西也相对比较多。

1. vim编译时，必须添加python或者相关语言库；
2. 编译安装**YouCompleteMe**
3. 在vim中配置**Valloric/YouCompleteMe**插件

以下是配置内容：

```vim
Bundle 'Valloric/YouCompleteMe'
let g:ycm_key_list_select_completion = ['<Down>']
let g:ycm_key_list_previous_completion = ['<Up>'] "使用上下方向键来选择补全列表中的内容
let g:ycm_complete_in_comments = 1	"在注释中使用补全
let g:ycm_complete_in_strings = 1		"在字符串使用补全
let g:ycm_use_ultisnips_completer = 1	" 使用ultisnips提示
let g:ycm_collect_identifiers_from_comments_and_strings = 1 " 将注释和字符串中的内容收入到补全
let g:ycm_collect_identifiers_from_tags_files = 1 " 从tags文件中导入补全
let g:ycm_seed_identifiers_with_syntax = 1 " 打开关键字补全
" 在新窗口跳转到定义，设置快捷键为 <leader>jd
let g:ycm_goto_buffer_command = 'horizontal-split'
nnoremap <leader>jd :YcmCompleter GoToDefinitionElseDeclaration<CR>
let g:ycm_global_ycm_extra_conf = "~/.vim/bundle/YouCompleteMe/third_party/ycmd/cpp/ycm/.ycm_extra_conf.py" "补全三方python包
```

设置完上面的配置后，基本的补全功能就实现了，效率着实可以提升不少。更多功能参考：**UltiSnips**。
