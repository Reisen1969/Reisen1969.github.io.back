+++
author = "rayrain"
title = "linux相关工具"
date = "2022-04-16"
description = "linux"
toc= true
math= true
tags = [
    "linux"
]
+++

## find - 递归地在层次目录中处理文件

```
find [path...] [expression]
```



```
-type   b:特殊块文件 缓冲的
		c:特殊字符文件 不缓冲
		d:目录
		p:命名管道
		f:普通文件
		l:符号链接
		s:套接字

```



```
//设置递归深度
-maxdepth 1
```



```
//在当前目录下搜索 Videos的文件夹 默认递归
find ./  -type d -name "Videos"
```



## grep - 打印匹配给定模式的行

```
grep [options] PATTERN [FILE...]
       grep [options] [-e PATTERN | -f FILE] [FILE...]

```

```
-r   递归
-n   在输出的每行前面加上它所在的文件中它的行号
-i   忽略大小写
-I   处理一个二进制文件，但是认为它不包含匹配的内容。
```



```
//递归的查找 main 字符串
grep -rn main 
```







