+++
author = "rayrain"
title = "常用的正则表达式"
date = "2022-04-17"
description = "正则"
toc= true
math= true
tags = [
    "正则"
]

+++

本文记录一下浏览代码时比较常用的正则表达式

[在线练习网站](https://regex101.com/)

## 单词边界 \b

匹配单词的开头和结尾

re: `\bhi`    str: `history`  匹配前两个字符

re: `ry\b`    str: `history`  匹配后两个字符

re: `\bhistory\b`    str: `history`  匹配整个字符



什么能被称为边界?

字母和 [空格 汉字 标点] 之间的分界,但是空格 汉字 标点之间不能算是分界



## 单词非边界 \B

匹配的结果不能是字母和 [空格 汉字 标点] 之间的分界

re: `ry\B`    str: `history`  无法匹配

re: `\Bst\B`    str: `history`  可以匹配到中间的字符



## 匹配文件的开头 \A

只能匹配文件的开头



## 匹配文件的结尾 \Z

只能匹配文件的结尾



## 匹配一行的开头 ^



## 匹配一行的结尾 $





## 匹配某个函数

```
main_\w+_start\(\)
```

```
main_AA_start()

main_BB_start()

main_CC_start()
```



