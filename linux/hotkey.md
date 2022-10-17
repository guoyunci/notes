### 快速纠错
___

*终端和vim插入模式*
| 用法 | 释义 |
| :--- | :---: |
| ctrl+h | 删除上一字符 |
| ctrl+w | 删除上一单词 |
| ctrl+u | 删除当前行 |
| ctrl+m | 回车 |

*终端*

| 用法 | 释义 |
| :--- | :---: |
| ctrl+b | 前移 |
| ctrl+f | 后移 |
| ctrl+k | 删除从当前光标处到当前行末尾的字符 |

*vim下快速切换insert和normal模式*

> 除了esc键,ctrl+c 和 ctrl+[ 可快速从insert模式切换到normal模式.

> gi可快速跳转到最后一次编辑的地方,并进入插入模式.



### 快速移动
___

*单词间跳转*
+ w/W移动到下一个word/WORD开头，e/E下一个word/WORD结尾。
+ b/B回到上一个word/WORD开头。
+ word指非空白符分割的单词，WORD空白符分割的单词。

*行间搜索移动*
+ f{char}移动到char字符上，t移动到char的前一个字符
+ 第一次没搜索到，分号(;)/逗号(,)搜下一个/上一个。
+ F反向搜索前面的字符。

*水平移动*
+ 0行首，^第一个非空白字符
+ $行尾，g_行尾非空白字符

*垂直移动*
+ 括号()在句子间移动
+ {}在段落间移动
+ 插件easy_motion


*页面移动*
+ gg/G移动到开头/结尾，ctrl+o快速返回
+ H/M/L跳转到屏幕的开头(head)/中间(middle)/结尾(lower)。
+ ctrl+u(upward),ctrl+f(forward)上下翻页。zz把屏幕置为中间。


### 快速编辑
___

*快速删除*
+ normal模式下x快速删除一个字符
+ d配合文本对象快速删除一个单词daw(d around word)
+ d,x都可以搭配数字执行,如2dd, 4x.

*快速修改*
+ 常用的三个，r(replace),c(change),s(substitube)
+ normal模式下，r替换一个字符，s替换并进入插入模式
+ c配合文本对象，快速进行修改

*查询*
+ /或者?进行前向或者反向搜索
+ n/N跳转到下一个或者上一个匹配
+ \*/#前向/后向匹配


### 搜索替换
___

*替换命令*
> substitute 查找并替换文本，支持正则

+ :[range]s[ubstitute]/{pattern}/{string}/{flags}
+ range表示范围 比如:10，20表示10-20行，%表全部
+ pattern是要替换的模式，string是替换后文本
+ flags常用标志:1)g(global)全局，2)c(confirm)确认，3)n(number)报告匹配到的次数而不替换，可用于查询匹配次数。
+ 精准匹配例子:% s/\<quick\>/slow/g

### 多文件操作
___

*buffer window tab*

vim打开一个文件后加载文件内容到缓冲区;
之后的修改都是在缓冲区内;
直到执行:w(write)时才会把修改内容写入到文件里;

*buffer切换*

+ 使用:ls列举当前缓冲区,可使用:b n 跳转到第n个缓冲区
+ :bpre :bnext :bfirst :blast
+ :b buffer_name 加上tab补全跳转
+ :e 表示edit,打开其他文件, 一个窗口, 多个缓冲区

*window窗口*

+ <ctrl+w>s 水平分割, <ctrl+w>v 垂直分割. 或者:sp和:vs
+ 切换窗口命令都是使用<ctrl+w> 作为前缀. <c-w>w 窗口间循环切换, h/j/k/l 左/下/上/右
+ 窗口移动, <c-w> H/J/K/L
+ 重排窗口,<c-w>= 等宽等高, <c-w>_ 最大化高度 <c-w>| 最大化宽度, [N]<c-w>_ 高设为n行, [N]<c-w>| 宽设为n列, :h window-resize查看文档

*tab(标签页)*

+ <c-w>T 把当前窗口移到一个新标签页
+ :tabe {filename} 在新标签页中打开 {filename} :tabnew {filename}
+ gt 切换到下一个标签页 gT 切换到上一个标签页


### 文本对象
___

*文本对象操作方式*

+ [number]<command>[test object], 如:3dw
+ number表示次数,command是命令,d(delete), c(change), y(yank)
+ text object是要操作的文本对象, 单词w, 句子s, 段落p

*示例*

+ iw表示inner word. aw表示a word.


### 寄存器
___

*insert模式下的复制粘贴*

+ :set autoindent 自动缩进,如果复制代码导致代码错乱,使用:set paste和:set nopaste解决
+ vim里操作的是寄存器,不是系统剪贴板
+ 默认使用的d删除和y复制的内容都放在了"无名寄存器"
+ x删除字符放到无名寄存器,p粘贴,可以调换字符.

*寄存器*

+ 通过"{register}前缀指定寄存器,不指定默认使用无名寄存器
+ 比如, "ayiw复制一个单词到寄存器a中,"bdd删除当前行到寄存器b中. :reg a 查看a寄存器内容.
+ 复制专用寄存器 "0使用y复制文本同时会被拷到复制寄存器0.
+ 系统剪贴板 "+可以在复制前加上 "+复制到系统剪贴板
+ "%当前文件名, ".上次插入的文本
+ 从vim里复制内容到系统剪贴板,需要clipboard支持,:echo has('clipboard')查看
+ :set clipboard=unnamed可以直接复制粘贴剪贴板内容
+ insert模式下,ctrl+r,然后+,也可以粘贴剪切板内容
+ :e! 重新加载并且不保存当前文件


### 宏批量处理
___

*如何使用*

+ 使用q开始录制，q结束录制
+ q{register}选择要保存的寄存器，把录制命令保存其中
+ @{register}回放寄存器中保存的命令

*例子*

给每一行开头和结尾增加""
1. normal模式下，qa,I",esc,A",esc,q
2. shift+V,G
3. :normal @a

### vim补全
___

+ ctrl+n, ctrl+p补全单词
+ ctrl+x+f补全文件路径
+ 关于全能补全,需要开启filetype=on, ctrl+x+o补全代码，需开启文件类型检查，安装插件

*技巧*

> r! echo % 输入文件路径(相对)

> r! echo %:p 输入文件全路径


### vim配色
___

+ :colorscheme 显示当前主题配色，默认default
+ :colorscheme <ctrl+d> 显示所有配色
+ :colorscheme name 修改配色
+ vim filename1 filename2 -o(水平分割) -O(垂直分割)


### vim配置
___

:h option-list 查看设置选项

### vim映射
___

*基本映射*

基本映射是在normal模式下的映射
+ 使用map就可以实现映射. unmap取消映射

*模式映射*

+ nmap/vmap/imap/定义映射只在normal/visual/insert模式分别有效.
+ nnoremap/vnoremap/inoremap, 非递归映射.
+ 任何时候,都应该使用非递归映射.
+ 默认的mapleader是 反斜线 \
+ `^ 表示上一次编辑模式停留的位置
