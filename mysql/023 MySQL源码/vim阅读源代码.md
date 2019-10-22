[如何使用vim的插件Ctags查看Linux源码](http://emb.hqyj.com/Column/8892.html)

为了实现类似SourceInsight功能，通过VIM+Ctags+Cscope+Taglist＋Source Explore +NERD Tree实现
##  安装vim插件Ctags和使用
#### 安装
Ctags工具是用来遍历源代码文件生成tags文件，这些tags文件能被编辑器或其它工具用来快速查找定位源代码中的符号（tag/symbol），如变量名，函数名等。比如，tags文件就是Taglist和OmniCppComplete工作的基础
```
# yum install ctags 
```
#### 使用方法
- 建立ctgs索引

```
ctags –R *
```
 “-R”表示递归创建，也就包括源代码根目录（当前目录）下的所有子目录。“*”表示所有文件。这条命令会在当前目录下产生一个“tags”文件，当用户在当前目录中运行vi时，会自动载入此tags文件。Tags文件中包括这些对象的列表：
1. 用#define定义的宏
2. 枚举型变量的值
3. 函数的定义、原型和声明
4. 名字空间（namespace）
5. 类型定义（typedefs）
6. 变量（包括定义和声明）
7. 类（class）、结构（struct）、枚举类型（enum）和联合（union）
8. 类、结构和联合中成员变量或函数
VIM用这个“tags”文件来定位上面这些做了标记的对象

- vi –t tag
1. 这个命令将打开定义tag（变量或函数或其它）的文件，并把光标定位到这一行
2. 如果这个变量或函数有多处定义，在VI命令行模式

“：ts”命令就能列出一个列表供用户选择。    

“：tp”为上一个tag标记文件，

“：tn”为下一个tag标记文件。当然，若当前tags文件中用户所查找的变量或函数名只有一个，“:tp,:tn”命令不可用

3. 更多功能通过命令man ctags或在Vim命令行下运行help ctags查询


https://blog.csdn.net/g_brightboy/article/details/16843899
## taglist
taglist插件是以vim脚本的形式存在，因此只需要将其下载下来放到相应的目录即可。taglist基于ctags才能发挥作用，因此在使用taglist之前，确保已经安装了ctags
#### 下载地址
```
下载地址(两个可选择任意一个)：
Official site  http://vim-taglist.sourceforge.net/
VIM online  http://www.vim.org/scripts/script.php?script_id=273
```
#### 安装
- 下载解压后，将插件脚本文件(.vim)和帮助文件(.txt)分别放入vim常用目录：
```

$HOME/.vim/ 或$HOME/vimfiles/ 或$VIM/vimfiles/
下的plugin/taglist.vim
doc/taglist.txt
```
- vim打开程序，在命令行模式输入：TlistOpen，就可以开启Taglist模式阅读代码
- taglist命令

```
ctrl+]，切换到定义的地方；

ctrl+o，会到调用的地方。
```

```
<CR>		跳到光标下tag所定义的位置，用鼠标双击此tag功能也一样（但要在vimrc文件中打开此项功能）
o		在一个新打开的窗口中显示光标下tag
<Space> 显示光标下tag的原型定义
u		更新taglist窗口中的tag
s		更改排序方式，在按名字排序和按出现顺序排序间切换
x		taglist窗口放大和缩小，方便查看较长的tag
+		打开一个折叠，同zo
-		将tag折叠起来，同zc
*		打开所有的折叠，同zR
=	    将所有tag折叠起来，同zM
[[		跳到前一个文件
]]		跳到后一个文件
q		关闭taglist窗口
<F1>    显示帮助
```

#### NERDTree
- 下载并安装
```
wget http://www.vim.org/scripts/download_script.php?src_id=17123 -O nerdtree.zip 
unzip nerdtree.zip

mkdir -p ~/.vim/{plugin,doc}

cp plugin/NERD_tree.vim ~/.vim/plugin/
cp doc/NERD_tree.txt ~/.vim/doc/
```
- 快捷键

快捷键|说明
---|---
ctrl + w + h |光标 focus 左侧树形目录
ctrl + w + l |光标 focus 右侧文件显示窗口
ctrl + w + w |光标自动在左右侧窗口切换
ctrl + w + r |移动当前窗口的布局位置
o |在已有窗口中打开文件、目录或书签，并跳到该窗口
go |在已有窗口 中打开文件、目录或书签，但不跳到该窗口
t |在新 Tab 中打开选中文件/书签，并跳到新 Tab
T |在新 Tab 中打开选中文件/书签，但不跳到新 Tab
i |split 一个新窗口打开选中文件，并跳到该窗口
gi |split 一个新窗口打开选中文件，但不跳到该窗口
s |vsplit 一个新窗口打开选中文件，并跳到该窗口
gs |vsplit 一个新 窗口打开选中文件，但不跳到该窗口
! |执行当前文件
O |递归打开选中 结点下的所有目录
x |合拢选中结点的父目录
X |递归 合拢选中结点下的所有目录
e |Edit the current dif
双击 |相当于 NERDTree-o
中键 |对文件相当于 NERDTree-i，对目录相当于 NERDTree-e
D |删除当前书签
P |跳到根结点
p |跳到父结点
K |跳到当前目录下同级的第一个结点
J |跳到当前目录下同级的最后一个结点
k |跳到当前目录下同级的前一个结点
j |跳到当前目录下同级的后一个结点
C |将选中目录或选中文件的父目录设为根结点
u |将当前根结点的父目录设为根目录，并变成合拢原根结点
U |将当前根结点的父目录设为根目录，但保持展开原根结点
r |递归刷新选中目录
R |递归刷新根结点
m |显示文件系统菜单
cd| 将 CWD 设为选中目录
I |切换是否显示隐藏文件
f |切换是否使用文件过滤器
F |切换是否显示文件
B |切换是否显示书签
q |关闭 NerdTree 窗口
? |切换是否显示 Quick Help


#### Cscope
Cscope是VIM适用的工具和插件，通过Cscope可以方便的获取某个函数的定义以及被那些函数调用 。
## 安装
```

```

cscope使用之前需要生成缓存文件，cscope -R之后就可以看到有名为cscope.out的文件，vim中执行