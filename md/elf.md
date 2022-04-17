+++
author = "rayrain"
title = "从ELF谈起"
date = "2022-03-13"
description = "elf"
toc= true
math= true
tags = [
    "elf","linux"
]
+++
本文信息来源:

[又是一期硬核内容：ELF文件格式](https://www.bilibili.com/video/BV1u54y1Q7qf)

[What's the difference of section and segment in ELF file format](https://stackoverflow.com/questions/14361248/whats-the-difference-of-section-and-segment-in-elf-file-format)

[ELF Sections & Segments and Linux VMA Mappings](https://web.archive.org/web/20171129031316/http://nairobi-embedded.org/040_elf_sec_seg_vma_mappings.html)

## ELF简介

ELF全称 executable and linkable format  ~~精灵~~

是一种linux下常用的可执行文件 对象 共享库的标准文件格式

>  还有许多其他可执行文件格式 PE Mach-O COFF COM	

内核中处理elf相关代码参考: binfmt_elf.c

elf中的数据按照Segment(段)和Section(节)两个概念进行划分

## ELF文件格式

![enter image description here](https://p90.f3.n0.cdn.getcloudapp.com/items/d5u9OWz0/8564ca76-29ff-4477-a52f-65c79c1c555b.jpeg?source=viewer&v=c75c9ae737d9075608e2acc178b69250)

![image-20220312151235824](https://p90.f3.n0.cdn.getcloudapp.com/items/bLuKBjJw/59ace77e-4212-46d3-91e9-6e88eed4b1be.jpeg?source=viewer&v=551de8e034df51f99ffc758fca7c756e)

### ELF Header

- 架构 ABI版本等基础信息
- program header table的位置和数量
- section header table的位置和数量

### Program header table

- 每个表项定义了一个segment
- 每个segment可包含多个section

### Section header table

- 每个表项定义了一个section

### readelf命令

可用readelf命令来展示elf文件的相关信息

**用法如下**:

```
用法：readelf <选项> elf-文件
 显示关于 ELF 格式文件内容的信息
 Options are:
  -a --all               Equivalent to: -h -l -S -s -r -d -V -A -I
  -h --file-header       Display the ELF file header
  -l --program-headers   Display the program headers
     --segments          An alias for --program-headers
  -S --section-headers   Display the sections' header
     --sections          An alias for --section-headers
  -g --section-groups    Display the section groups
  -t --section-details   Display the section details
  -e --headers           Equivalent to: -h -l -S
```

比如,使用readelf来查看date的信息

```
readelf -l /bin/date
```

输出

```
Elf 文件类型为 DYN (Position-Independent Executable file)
Entry point 0x38c0
There are 13 program headers, starting at offset 64

程序头：
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  PHDR           0x0000000000000040 0x0000000000000040 0x0000000000000040
                 0x00000000000002d8 0x00000000000002d8  R      0x8
  INTERP         0x0000000000000318 0x0000000000000318 0x0000000000000318
                 0x000000000000001c 0x000000000000001c  R      0x1
      [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
  LOAD           0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x00000000000028a8 0x00000000000028a8  R      0x1000
  LOAD           0x0000000000003000 0x0000000000003000 0x0000000000003000
                 0x0000000000010001 0x0000000000010001  R E    0x1000
  LOAD           0x0000000000014000 0x0000000000014000 0x0000000000014000
                 0x0000000000005cf0 0x0000000000005cf0  R      0x1000
  LOAD           0x0000000000019ff0 0x000000000001aff0 0x000000000001aff0
                 0x00000000000010b0 0x0000000000001268  RW     0x1000
  DYNAMIC        0x000000000001ab98 0x000000000001bb98 0x000000000001bb98
                 0x00000000000001f0 0x00000000000001f0  RW     0x8
  NOTE           0x0000000000000338 0x0000000000000338 0x0000000000000338
                 0x0000000000000040 0x0000000000000040  R      0x8
  NOTE           0x0000000000000378 0x0000000000000378 0x0000000000000378
                 0x0000000000000044 0x0000000000000044  R      0x4
  GNU_PROPERTY   0x0000000000000338 0x0000000000000338 0x0000000000000338
                 0x0000000000000040 0x0000000000000040  R      0x8
  GNU_EH_FRAME   0x0000000000018000 0x0000000000018000 0x0000000000018000
                 0x0000000000000454 0x0000000000000454  R      0x4
  GNU_STACK      0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000000 0x0000000000000000  RW     0x10
  GNU_RELRO      0x0000000000019ff0 0x000000000001aff0 0x000000000001aff0
                 0x0000000000001010 0x0000000000001010  R      0x1

 Section to Segment mapping:
  段节...
   00     
   01     .interp 
   02     .interp .note.gnu.property .note.gnu.build-id .note.ABI-tag .gnu.hash .dynsym .dynstr .gnu.version .gnu.version_r .rela.dyn .rela.plt 
   03     .init .plt .text .fini 
   04     .rodata .eh_frame_hdr .eh_frame 
   05     .init_array .fini_array .data.rel.ro .dynamic .got .data .bss 
   06     .dynamic 
   07     .note.gnu.property 
   08     .note.gnu.build-id .note.ABI-tag 
   09     .note.gnu.property 
   10     .eh_frame_hdr 
   11     
   12     .init_array .fini_array .data.rel.ro .dynamic .got 

```

可知:

- 在加载到内存中时,程序被分成了13个Segment(从PHDR到GNU_RELRO)
- 每个Segment都包含了1个或者更多的Section

## Segment vs Section

### Segment

- 包含着运行时需要的信息

- 用于告诉操作系统,段应该被加载到虚拟内存中的什么位置?每个段都有那些权限?(read, write, execute)
- 每个Segment主要包含加载地址 文件中的范围 内存权限 对齐方式等信息

### Section

- 包含着链接时需要的信息

- 用于告诉链接器,elf中每个部分是什么,哪里是代码,哪里是只读数据,哪里是重定位信息
- 每个Section主要包含Section类型 文件中的位置 大小等信息
- 链接器会把Section放入Segment中

### Segment和Section的关系

- 相同权限的Section会放入同一个Segment,例如.text和.rodata section
- 一个Segment包含许多Section,一个Section可以属于多个Segment

### 链接脚本

运行

```
ld --verbose
```

可以看到本系统中所用的脚本

我的Archlinux 5.16.13-arch1-1的链接脚本一部分是这样:

```
  .gnu.version_r  : { *(.gnu.version_r) }
  .rela.dyn       :
    {
      *(.rela.init)
      *(.rela.text .rela.text.* .rela.gnu.linkonce.t.*)
      *(.rela.fini)
      *(.rela.rodata .rela.rodata.* .rela.gnu.linkonce.r.*)
      *(.rela.data .rela.data.* .rela.gnu.linkonce.d.*)
      *(.rela.tdata .rela.tdata.* .rela.gnu.linkonce.td.*)
      *(.rela.tbss .rela.tbss.* .rela.gnu.linkonce.tb.*)
      *(.rela.ctors)
      *(.rela.dtors)
      *(.rela.got)
      *(.rela.bss .rela.bss.* .rela.gnu.linkonce.b.*)
      *(.rela.ldata .rela.ldata.* .rela.gnu.linkonce.l.*)
      *(.rela.lbss .rela.lbss.* .rela.gnu.linkonce.lb.*)
      *(.rela.lrodata .rela.lrodata.* .rela.gnu.linkonce.lr.*)
      *(.rela.ifunc)
    }

```

表示:

 ` .gnu.version_r`  Section 会被放入 `.gnu.version_r` Segment

`.rela.init`等一大堆的Section,会被放入 `.rela.dyn`  Segment

> 汇编中的伪指令全部都是Section,要等链接之后才会有Segment
>
> NASM中 `.section` 和`.segment` 这两个是等效的,都表示 `Section`

## ELF文件分类

### 可执行文件(ET_EXEC)

可直接运行的程序,必须包含segment

### 对象文件(ET_REL,*.o)

需要与其他对象文件链接,必须包含section

### 动态库(ET_DYN,*.so)

与其他对象文件/可执行文件链接

必须同时包含segment和section

## ELF的内存映射





![image-20220312152051070](https://p90.f3.n0.cdn.getcloudapp.com/items/2Nuzm5x8/6763062f-8c70-4866-8e65-3ed739cd9f59.jpeg?source=viewer&v=c0b727f3a5e6f91c0d20f433f5cfe84c)





## 查看内存映射情况

```
cat /proc/[pid]/maps
```

比如运行

```
cat /proc/self/maps
```

查看cat本身的内存映射

```
563b04d75000-563b04d77000 r--p 00000000 fe:00 6294104                    /usr/bin/cat
563b04d77000-563b04d7c000 r-xp 00002000 fe:00 6294104                    /usr/bin/cat
563b04d7c000-563b04d7f000 r--p 00007000 fe:00 6294104                    /usr/bin/cat
563b04d7f000-563b04d80000 r--p 00009000 fe:00 6294104                    /usr/bin/cat
563b04d80000-563b04d81000 rw-p 0000a000 fe:00 6294104                    /usr/bin/cat
563b058b6000-563b058d7000 rw-p 00000000 00:00 0                          [heap]
7f5f7324b000-7f5f73837000 r--p 00000000 fe:00 6364075                    /usr/lib/locale/locale-archive
7f5f73837000-7f5f7383a000 rw-p 00000000 00:00 0 
7f5f7383a000-7f5f73866000 r--p 00000000 fe:00 6294921                    /usr/lib/libc.so.6
7f5f73866000-7f5f739dc000 r-xp 0002c000 fe:00 6294921                    /usr/lib/libc.so.6
7f5f739dc000-7f5f73a30000 r--p 001a2000 fe:00 6294921                    /usr/lib/libc.so.6
7f5f73a30000-7f5f73a31000 ---p 001f6000 fe:00 6294921                    /usr/lib/libc.so.6
7f5f73a31000-7f5f73a34000 r--p 001f6000 fe:00 6294921                    /usr/lib/libc.so.6
7f5f73a34000-7f5f73a37000 rw-p 001f9000 fe:00 6294921                    /usr/lib/libc.so.6
7f5f73a37000-7f5f73a46000 rw-p 00000000 00:00 0 
7f5f73a70000-7f5f73a92000 rw-p 00000000 00:00 0 
7f5f73a92000-7f5f73a94000 r--p 00000000 fe:00 6294911                    /usr/lib/ld-linux-x86-64.so.2
7f5f73a94000-7f5f73abb000 r-xp 00002000 fe:00 6294911                    /usr/lib/ld-linux-x86-64.so.2
7f5f73abb000-7f5f73ac6000 r--p 00029000 fe:00 6294911                    /usr/lib/ld-linux-x86-64.so.2
7f5f73ac7000-7f5f73ac9000 r--p 00034000 fe:00 6294911                    /usr/lib/ld-linux-x86-64.so.2
7f5f73ac9000-7f5f73acb000 rw-p 00036000 fe:00 6294911                    /usr/lib/ld-linux-x86-64.so.2
7ffec90a6000-7ffec90c8000 rw-p 00000000 00:00 0                          [stack]
7ffec918d000-7ffec9191000 r--p 00000000 00:00 0                          [vvar]
7ffec9191000-7ffec9193000 r-xp 00000000 00:00 0                          [vdso]
ffffffffff600000-ffffffffff601000 --xp 00000000 00:00 0                  [vsyscall]
```

从左到右为:

- 虚拟地址的起始和结束
- 该内存映射的类型flag r(read) w(write) x(execute) p(private) s(shared) 
- 实际对象在该内存映射上相对于起始的偏移量
- `major:minor`: the major and minor number pairs of the device holding the file that has been mapped.
- 映射文件的索引节点号码
- 该内存映射文件的名称 
