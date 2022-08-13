+++
author = "rayrain"
title = "使用qemu启动riscv64 kernel"
date = "2022-04-16"
description = "qemu"
toc= true
math= true
tags = [
    "riscv","kernel","qemu"
]

+++



在这里只是记录一下,方便以后查阅

本文的内容来源

[riscv官方网页](https://risc-v-getting-started-guide.readthedocs.io/en/latest/linux-qemu.html) 

[解决文件系统问题](https://github.com/riscv-admin/risc-v-getting-started-guide/issues/29)

[编译busybox](https://www.cnblogs.com/jzcn/p/14932093.html)

[9p](https://wiki.qemu.org/Documentation/9psetup)

## 获取源码

```
git clone https://github.com/qemu/qemu
git clone https://github.com/torvalds/linux
git clone https://git.busybox.net/busybox
```

## 编译qemu

```
./configure --target-list=riscv64-softmmu --enable-virtfs
make -j $(nproc)
```

## 编译kernel

```
//这里设置的工具链,以自己环境里的为准
//生成配置文件
make ARCH=riscv CROSS_COMPILE=riscv64-linux-gnu- defconfig
//开始编译
make ARCH=riscv CROSS_COMPILE=riscv64-linux-gnu- -j $(nproc)
```

## 编译Busybox

```
make ARCH=riscv CROSS_COMPILE=riscv64-linux-gnu- defconfig
make ARCH=riscv CROSS_COMPILE=riscv64-linux-gnu- menuconfig
//Check Settings - Build static binary (no shared libs)
make ARCH=riscv CROSS_COMPILE=riscv64-linux-gnu- install CONFIG_PREFIX=./rootfs
//这里选择将busybox安装到本目录下的./rootfs文件夹下
```

## 制作根文件系统

```
//制作文件系统
$ cd youer_workspace
mkdir rootfs
$ cd rootfs
$ dd if=/dev/zero of=rootfs.img bs=1M count=50
$ mkfs.ext2 -L riscv-rootfs rootfs.img

//将它挂载到 mnt/rootfs
$ sudo mkdir /mnt/rootfs
$ sudo mount rootfs.img /mnt/rootfs

//将busybox编译出来的内容复制到rootfs.img文件系统中
$ sudo cp -ar your_busybox/rootfs/* /mnt/rootfs

//创建根目录
$ sudo mkdir /mnt/rootfs/{dev,home,mnt,proc,sys,tmp,var}
$ sudo chown -R -h root:root /mnt/rootfs
```

```
//检查文件系统
$ df /mnt/rootfs
Filesystem     1K-blocks  Used Available Use% Mounted on
/dev/loop5         49584  1704     45320   4% /mnt/rootfs
$ mount | grep rootfs
riscv64-linux/rootfs/rootfs.img on /mnt/rootfs type ext2 (rw,relatime)

//卸载文件系统
$ sudo umount /mnt/rootfs
$ sudo rmdir /mnt/rootfs
```



## 使用根文件系统启动内核

```
qemu-system-riscv64 -nographic -machine virt \
     -kernel linux/arch/riscv/boot/Image -append "root=/dev/vda ro console=ttyS0" \
     -drive file=your_rootfs.img,format=raw,id=hd0 \
     -device virtio-blk-device,drive=hd0 \
     -fsdev local,id=p9fs,path=./share,security_model=mapped \
-device virtio-9p-pci,fsdev=p9fs,mount_tag=p9
```

## 挂载共享目录

```
mount p9 -t 9p /mnt
```

