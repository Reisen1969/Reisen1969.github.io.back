+++
author = "rayrain"
title = "VIM技巧(持续更新)"
date = "2022-04-05"
description = "vim"
toc= true
math= true
tags = [
    "vim"
]
+++

 简单记录一下vim的技巧



## 打开nerdtree 目录

命令行下:

```
vim ./
```

vim中:

```
:e %:h    //打开当前文件所在目录
:e ./     //貌似这样也可以
```



## 在nerdtree目录下

```
s            //默认打开一个垂直的分割窗口
t            //打开一个新的tab窗口
```

##  操作tab page

```
gt           //默认移动到下一个tab
1gt          //打开第一个tab
2gt          //打开第二个tab
```

##  操作分割窗口

```
:sp          //横向打开分割窗口
:vsp         //垂直打开分割窗口
```

```
ctrl w + 方向键  //在分割窗口之间移动
```

## 内置grep搜索

```
:vim[grep][!] /{pattern}/[g][j] {file} ...
:vim[grep][!] {pattern} {file} ...

vimgrep可以简写为vim
```

举例

```
在当前目录下的linux-user文件夹下 的c文件中搜索 main( 字符串
:vimgrep main( ./linux-user/*.c
在当前目录下递归寻找
:vimgrep main( ./**/*.c
```

更多

```
:cnext, :cn         # 当前页下一个结果
:cprevious, :cp     # 当前页上一个结果
:clist, :cl         # 使用 more 打开 Quickfix 窗口
:copen, :cope, :cw  # 打开 Quickfix 窗口，列出所有结果
:ccl[ose]           # 关闭 Quickfix 窗口。
```

```
lvimgrep 命令
lvimgrep 与 vimgrep 搜索命令基本一样，不同点在于搜索结果不是显示在 Quickfix 中而是显示在 location-list 中
```

## 移动

使用大括号在空行之间移动

## 迅速查找字符串

将光标移动到字符串上然后输入

```
shift + 8
```





## Ctags使用

生成tags 数据库文件

```
ctags -R .            //递归生成当前根目录的tags文件
```

vim可以自动匹配tags文件

使用:

```
ctrl + ]             //跳转到定义处
ctrl + T             //返回到跳转前的位置
ctrl + W + ]         //分割当前窗口,并在新窗口中显示跳转到的定义
ctrl + O             //返回之前的位置
:ts                  //列出所有匹配的标签
```



## CScope使用

生成数据库文件

```
cscope -Rbq
```

在vim中:

```
cscope 命令:(以下都已:cs开头)
add  : 添加一个新的数据库             (Usage: add file|dir [pre-path] [flags])
find : 查询一个模式                   (Usage: find a|c|d|e|f|g|i|s|t name)
       a: Find assignments to this symbol
       c: Find functions calling this function
       d: Find functions called by this function   查找被这个函数调用的函数
       e: Find this egrep pattern
       f: Find this file                           查找这个文件
       g: Find this definition					   查找定义
       i: Find files #including this file          查找include了这个文件的所有文件
       s: Find this C symbol			           查找C符号
       t: Find this text string                    查找这个文本字符串
help : 显示此信息                     (Usage: help)
kill : 结束一个连接                   (Usage: kill #)
reset: 重置所有连接                   (Usage: reset)
show : 显示连接                       (Usage: show)

```





## :s替换字符串

```
:s/helllo/sky/   替换当前行第一个hello为sky
:s/helllo/sky/g  替换当前行的所有hello为sky
:n,$s/hello/sky 替换第n行开始到最后一行的第一个hello为sky
:n,$s/hello/sky/g 替换第n行开始到最后一行的所有hello为sky
:%s/hello/sky    替换每一行的第一个hello为sky
:%s/hello/sky/g    替换每一行的所有hello为sky

```

## 寄存器

![img](https://pic3.zhimg.com/v2-9752311b4c4e51ddb02f59ce25e4830e_b.jpg)

### 访问寄存器

```
"registers
```



## 正则表达式

全字搜索

```
/\<word\>
```

