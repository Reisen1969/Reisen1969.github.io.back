+++
author = "rayrain"
title = "riscv裸机"
date = "2022-05-29"
description = "qemu"
toc= true
math= true
tags = [
    "qemu"
]

+++

https://mth.st/blog/riscv-qemu/




使用virit机器启动,
-s 表示 启动gdb server 默认端口1234
-S 表示CPU不立即开始运行

```
/qemu-system-riscv64 -machine virt -nographic -s -S 



```

编译riscv程序
```
.section .init
.global _start
_start:
    li s1, 0x10000000 # s1 := 0x1000_0000
    la s2, message    # s2 := <message>
    addi s3, s2, 14   # s3 := s2 + 14
1:
    lb s4, 0(s2)      # s4 := (s2)
    sb s4, 0(s1)      # (s1) := s4
    addi s2, s2, 1    # s2 := s2 + 1
    blt s2, s3, 1b    # if s2 < s3, branch back to 1

.section .data
message:
  .string "Hello, world!\n"

```

$ riscv64-none-elf-as loop.s -g -o loop.elf
$ riscv64-none-elf-objcopy -O binary loop.elf loop.img

$ riscv64-none-elf-ld -o loop.linked.elf loop.elf
$ riscv64-none-elf-objcopy -O binary loop.linked.elf loop.linked.img
$ diff loop.img loop.linked.img

 riscv64-none-elf-ld --verbose > qemu-riscv64-virt.ld

 riscv64-none-elf-ld -T qemu-riscv64-virt.ld -o loop.linked.elf loop.elf


看支持的硬件
 qemu-system-riscv64 -machine help