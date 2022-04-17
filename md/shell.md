+++
author = "rayrain"
title = "shell脚本用法"
date = "2022-04-16"
description = "linux"
toc= true
math= true
tags = [
    "linux","shell"
]

+++

## 分支

```
if [ 条件表达式 ]; then
    指令
else [ 条件表达式 ]; then
fi
```

```
if [ 条件表达式 ]; then
    指令一
else
    指令二
fi
```

```
if [[ $1=='d' ]]; then
   echo $1
else
    echo "error"
fi

```

## 传入参数

使用 $1 $2 $3 $4表示



