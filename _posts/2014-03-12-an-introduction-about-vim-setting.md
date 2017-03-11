---
title: VIM IDE 环境配置
category: linux
tags:
- vim
---

## 介绍

要使VIM变成和VS一样的集成环境，可通过安装以下插件来实现，分别是
* Ctags
* Taglist
* GNU Global
* OmniCppComplete
* SuperTab
* Winmanager
* NERDTree
* MiniBufExplorer

下面将分别介绍各个插件的安装与使用。

<!--more-->

### 1. Ctags安装

安装步骤:
1) 从 http://ctags.sourceforge.net/ 下载源代码包后，解压缩生成源代码目录。
2) 进入源代码根目录执行./configure。
3) 执行make。
4) 编译成功后执行make install。

或者直接”yum install ctags”

常用命令列表：
1）`$ ctags –R *`   (为Linux系统Shell提示符)
2）`$ vim –t tag`   (请把tag替换为您欲查找的变量或函数名)
3）：ts				(ts 助记字：tags list, “:”开头的命令为vim中命令行模式命令)
4）：tp				(tp 助记字：tags preview)
5）：tn				(tn 助记字：tags next) 
6）Ctrl + ]			(跳转到该标记的定义处)
7）Ctrl + t			(返回到上次跳转前位置)

命令解释：
1) '$ ctags –R \*'："-R"表示递归创建，也就包括源代码根目录（当前目录）下的所有子目录。"*"表示所有文件。
2) 这条命令会在当前目录下产生一个“tags”文件，当用户在当前目录中运行vim时，会自动载入此tags文件。

Tags文件中包括这些对象的列表：
1）用#define定义的宏
2）枚举型变量的值
3）函数的定义、原型和声明
4）名字空间（namespace）
5）类型定义（typedefs）
6）变量（包括定义和声明）
7）类（class）、结构（struct）、枚举类型（enum）和联合（union）
8）类、结构和联合中成员变量或函数

vim用这个“tags”文件来定位上面这些做了标记的对象。

注意：打开文件时要在运行ctags的目录下打开，不能进入子文件夹后再打开。运行vim的时候，必须在“tags”文件所在的目录下运行。否则，运行vim的时候还要用“:set tags=”命令设定“tags”文件的路径，这样vim才能找到“tags”文件。例如：set tags=/root/tags

在~/.vimrc中添加以下这行：
```
map <C-F12> :!ctags -R --c++-kinds=+p --fields=+iaS --extra=+q .<CR> 
```

说明：
在vim中配置ctrl+F12组合快捷键。所以我们也可以进入代码根目录后，打开vim，按下Ctrl-F12快捷键自动生成tags文件，命令执行完后，会在源代码目录生成tags文件。

注：在我机子上不能用 map <C-F12>,用map <F12>倒是可以。

更多功能通过命令man ctags或在Vim命令行下运行help ctags查询。


### 2. Taglist安装

安装步骤：
1) 从 http://www.vim.org/scripts/script.php?script_id=273 下载安装包，也可以从 http://vim-taglist.sourceforge.net/index.html 下载。 
2) 将Taglist安装包拷贝到~/.vim目录并解压，解压后会在~/.vim目录中生成几个新子目录，如plugin和doc（安装其它插件时，可能还会新建autoload等其它目录）。
3) 进入~/.vim/doc目录，在Vim下运行"helptags ."命令。此步骤是将doc下的帮助文档加入到Vim的帮助主题中，这样我们就可以通过在Vim中运行“help taglist.txt”查看taglist帮助。
4) 打开配置文件/etc/.vimrc，加入以下几行：
```
let Tlist_Show_One_File=1           "不同时显示多个文件的tag，只显示当前文件的    
let Tlist_Exit_OnlyWindow=1         "如果taglist窗口是最后一个窗口，则退出vim   
let Tlist_Ctags_Cmd="/usr/bin/ctags"  "将taglist与ctags关联 
```

基本功能使用方法:
在Vim命令行下运行":Tlist"就可以打开Taglist窗口，再次运行":Tlist"则关闭。


常用配置选项:
1) Tlist_Ctags_Cmd选项用于指定你的Exuberant ctags程序的位置，如果它没在你PATH变量所定义的路径中，需要使用此选项设置一下。
2) 如果你不想同时显示多个文件中的tag，设置Tlist_Show_One_File为1。缺省为显示多个文件中的tag。
3) 设置Tlist_Sort_Type为”name”可以使taglist以tag名字进行排序，缺省是按tag在文件中出现的顺序进行排序。按tag出现的范围（即所属的namespace或class）排序，已经加入taglist的TODO List，但尚未支持。
4) 如果想taglist窗口是最后一个窗口时退出VIM，设置Tlist_Exit_OnlyWindow为１。
5) 如果想taglist窗口出现在右侧，设置Tlist_Use_Right_Window为１。缺省显示在左侧。
6) 在gvim中，如果你想显示taglist菜单，设置Tlist_Show_Menu为１。你可以使用Tlist_Max_Submenu_Items和Tlist_Max_Tag_Length来控制菜单条目数和所显示tag名字的长度；
7) 缺省情况下，在双击一个tag时，才会跳到该tag定义的位置，如果你想单击tag就跳转，设置Tlist_Use_SingleClick为１；
8) 如果你想在启动VIM后，自动打开taglist窗口，设置Tlist_Auto_Open为1；
9) 如果你希望在选择了tag后自动关闭taglist窗口，设置Tlist_Close_On_Select为1；
10) 当同时显示多个文件中的tag时，设置Tlist_File_Fold_Auto_Close为１，可使taglist只显示当前文件tag，其它文件的tag都被折叠起来。
11) 在使用:TlistToggle打开taglist窗口时，如果希望输入焦点在taglist窗口中，设置Tlist_GainFocus_On_ToggleOpen为1；
12) 如果希望taglist始终解析文件中的tag，不管taglist窗口有没有打开，设置Tlist_Process_File_Always为1；
13) Tlist_WinHeight和Tlist_WinWidth可以设置taglist窗口的高度和宽度。
14) Tlist_Use_Horiz_Window为１设置taglist窗口横向显示；


### 3. GNU Global安装

介绍：
gnu global是一个类似cscope的工具，也能提供源文件之间的交叉索引。 其独到之处在于，当你生成索引文件以后，再修改整个项目里的一个文件，然后增量索引的过程非常快。 

安装步骤：(或者直接”yum install global”)
1) 在http://www.gnu.org/software/global/download.html下载源码包进行安装（./configure && make && make install)
2) 在安装目录拷贝两个文件gtags.vim和gtags-cscope.vim到~/.vim/plugin目录下
3) 在/etc/vimrc中添加"nmap hm ：GtagsCursor<CR>"

安装后说明：
安装好以后，有global、gtags、gtags-cscope三个命令。global是查询，gtags是生成索引文件，gtags-cscope是与cscope一样的界面。 

使用：
1）#gtags 生成数据库文件 （这样就生成了当前目录下的索引文件，包括GTAGS、GRTAGS、GPATH三个文件）
2）首先进入vim，然后执行：
```
:set cscopeprg=gtags-cscope 
:cs add GTAGS
```
3）在vim里当光标定位到某一行，执行"hm",即可查看quickfix窗口显示的目录项。

注：当我们更改了某个文件以后，比如project/subdir1/subdir2/file1.c，想更新索引文件(索引文件是project/GTAGS)，只要这样： 
```
$ cd project/subdir1/subdir2/ 
$ vim file1.c 
$ global -u 
```

global -u这个命令会自动向上找到project/GTAGS，并更新其内容。而gtags相对于cscope的优势就在这里.


### 4. OmniCppComplete安装

介绍：
OmniCppComplete主要提供输入时实时提供类或结构体的属性或方法的提示和补全。跟Tlist一样，OmniCppComplete也是一个Vim插件，同样依赖与Ctags工具生成的tags文件。

安装步骤：
1）从http://www.vim.org/scripts/script.php?script_id=1520下载安装包后。
2）进入~/.vim目录，将安装版解压缩
3）进入~/.vim/doc目录，在Vim命令行下运行"helptags .”
4）在/etc/.vimrc中加入以下几行：
```
set nocp    
filetype plugin on
```
 
nmicppcompete功能：
1）命名空间(namespace),类(class),结构(struct)和联合(union)补全
2）函数属性成员和返回值类型补全
3）"this"指针成员补全
4）C/C++类型转换(cast)对象补全
5）类型定义(typedef)和匿名类型(anonymous types)补全

基本功能使用：
在配置好Vim，并生成了ctags标签库前提条件下，Vim中在输入 “xxx." 或者 "xxx->" 时会弹出如下补全提示：

```
+-------------------------------------+    
|method1(   f  +  MyNamespace::MyClass|    
|_member1   m  +  MyNamespace::MyClass|    
|_member2   m  #  MyNamespace::MyClass|    
|_member3   m  -  MyNamespace::MyClass|    
+-------------------------------------+    
    ^        ^   ^          ^    
   (1)      (2)  (3)       (4)    

```

说明:
1: 为符号名称
2: 为符号类型
3: 为访问控制标识
4: 为符号定义所在域(scope)

符号类型: （符号的类型，可能的值为)
c : 类(class)
d : 宏(macro definition)
e : 枚举值(enumeator)
f : 函数(function)
g : 枚举类型名称
m : 类/结构/联合成员(member)
n : 命名空间(namespace)
p : 函数原型(function prototype)
s : 结构体名称(structure name)
t : 类型定义(typedef)
u : 联合名(union name)
v : 变量定义(variable defination)

访问控制: (类成员访问控制，取值)
\+ : 公共(public)
\# : 保护(protected)
\- : 私有(private)


常用配置选项：(vim中，可以通过以下选项控制omnicppcomplete查找/补全方式)
* mniCpp_GlobalScopeSearch : 全局查找控制。0:禁止；1:允许(缺省)
* mniCpp_NamespaceSearch : 命名空间查找控制。
	* 0 : 禁止查找命名空间
	* 1 : 查找当前文件缓冲区内的命名空间(缺省)
	* 2 : 查找当前文件缓冲区和包含文件中的命名空间
* mniCpp_DisplayMode : 类成员显示控制(是否显示全部公有(public)私有(private)保护(protected)成员)。
	* 0 : 自动
	* 1 : 显示所有成员
* mniCpp_ShowScopeInAbbr : 选项用来控制匹配项所在域的显示位置。缺省情况下，omni显示的补全提示菜单中总是将匹配项所在域信息显示在缩略信息最后一列。
	* 0 : 信息缩略中不显示匹配项所在域(缺省)
	* 1 : 显示匹配项所在域，并移除缩略信息中最后一列
* mniCpp_ShowPrototypeInAbbr : 是否是补全提示缩略信息中显示函数原型。
	* 0 : 不显示(缺省)
	* 1 : 显示原型
* mniCpp_ShowAccess : 是否显示访问控制信息('+', '-', '#')。0/1, 缺省为1(显示)
* mniCpp_DefaultNamespaces : 默认命名空间列表，项目间使用','隔开。
	* 如：let OmniCpp_DefaultNamespaces = ["std', "MyNamespace"]
* mniCpp_MayCompleteDot : 在'.'号后是否自动运行omnicppcomplete给出提示信息。 
	* 0：不自动运行
	* 1：自动运行（缺省值）
* mniCpp_MayCompleteArray : 在"->"后是否自动运行omnicppcomplete给出提示信息。0：0：不自动运行
	* 1：自动运行（缺省值）
* OmniCpp_MayCompleteScope : 在域标识符"::"后是否自动运行omnicppcomplete给出提示信息。
	* 0：不自动运行
	* 1：自动运行（缺省值）
* OmniCpp_SelectFirstItem : 是否自动选择第一个匹配项。仅当"completeopt"不为"longest"时有效。
	* 0 : 不选择第一项(缺省)
	* 1 : 选择第一项并插入到光标位置
	* 2 : 选择第一项但不插入光标位置
* OmniCpp_LocalSearchDecl : 使用Vim标准查找函数/本地(local)查找函数。Vim内部用来在函数中查找变量定义的函数需要函数括号位于文本的第一列，而本地查找函数并不需要。


### 5. SuperTab安装

介绍：
SuperTab使Tab快捷键具有更快捷的上下文提示功能。跟OmniCppComplete一样，SuperTab也是一个Vim插件。

安装步骤：
1) 从http://www.vim.org/scripts/script.php?script_id=1643下载安装版。这个安装包跟先前的几个Vim插件不同，它是一个vba文件，即Vimball格式的安装包，这种格式安装包提供傻瓜式的安装插件的方法。
2) 用Vim打开.vba安装包文件。
3) 在Vim命令行下运行命令“UseVimball ~/.vim”。此命令将安装包解压缩到~/.vim目录。VImball安装方式的便利之处在于你可以在任何目录打开.vba包安装，而不用切换到安装目的地目录。而且不用运行helptags命令安装帮助文档。
4) 在/etc/.vimrc文件中加入以下这行：
```
let g:SuperTabDefaultCompletionType="context"
```

注：在vim里的功能快捷键
Ctrl+P	向前切换成员
Ctrl+N	向后切换成员
Ctrl+E	表示退出下拉窗口, 并退回到原来录入的文字


### 6. Winmanager、NERDTree、MiniBufExplorer安装

介绍：前面几个工具和插件，主要提供快捷的编辑功能，如定义跳转，符号查询，符号提示与补全等。这里的三个插件，主要优化布置VIm的界面。具体来说：
1）NERDTree提供树形浏览文件系统的界面
2）MiniBufExplorer提供多文件同时编辑功能
3）Winmanager将这NERDTree界面和Taglist界面整合起来，使vim更像VS

安装步骤：
1) 分别从
http://www.vim.org/scripts/script.php?script_id=1658
http://www.vim.org/scripts/script.php?script_id=159
http://www.vim.org/scripts/script.php?script_id=95
下载NERDTree，MiniBufExplorer和Winmanager安装包

2) 像其它插件一样，将NERDTree安装包解压到~/.vim目录。并进入doc目录，在Vim命令行下运行"helptags ."命令。
3) MiniBufExplorer只有一个.vim文件，将其拷贝到~/.vim/plugin目录。
4) 在/etc/.vimrc文件中加入以下几行：
```
let g:miniBufExplMapWindowNavVim = 1   
let g:miniBufExplMapWindowNavArrows = 1   
let g:miniBufExplMapCTabSwitchBufs = 1   
let g:miniBufExplModSelTarget = 1  
let g:miniBufExplMoreThanOne=0  
```

5) 将Winmanager安装包解压到~/.vim目录。
6) 在/etc/.vimrc文件中加入以下几行：
```
let g:NERDTree_title="[NERDTree]"  
let g:winManagerWindowLayout="NERDTree|TagList"  
  
function! NERDTree_Start()  
    exec 'NERDTree'  
endfunction  
  
function! NERDTree_IsValid()  
    return 1  
endfunction  
  
nmap wm :WMToggle<CR>  
```

注：在vim下使用“:NERDTree”或者“wm“即可显示树形浏览文件系统的界面。

7) 这个版本的Winmanager好像有个小bug，你在打开Winmanager界面时，会同时打开一个空的文件。这会影响后续使用，所以我们要在打开Winmanager时关掉这个空文件。在~/.vim/plugin目录下的winmanager.vim文件中找到以下函数定义并在第5行下添加第6行的内容：
```
function! <SID>ToggleWindowsManager()  
   if IsWinManagerVisible()  
      call s:CloseWindowsManager()  
   else  
      call s:StartWindowsManager()  
      exe 'q'  
   end  
endfunction  
```

8）MiniBufExplorer无法使用ctrl+tab键进行切换，进行如下修改：
找到 noremap    :call CycleBuffer(1):
重新定义成自己的map即可
```
noremap wn  :call CycleBuffer(1):
noremap wp  :call CycleBuffer(0):
```
这样就可以用wn / wp 进行buffer切换

### 7. 显示效果

![vim_image](https://github.com/kulong0105/kulong0105.github.io/blob/master/assets/img/vim.jpg?raw=true)
