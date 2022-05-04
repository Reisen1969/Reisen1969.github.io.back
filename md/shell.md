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
if [[ $1 == 'qd' ]]; then
   echo '1'
elif [[ $1 == 'qi' ]]; then
 echo '2'
elif [[ $1 == 'ob' ]]; then
 echo '3'
else
  echo 'error'
fi

```



## 传入参数

使用 $1 $2 $3 $4表示



## 命令行里的循环

```
while true ; do ./a.out ; done
```

