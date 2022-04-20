+++
author = "rayrain"
title = "riscv 启动分析"
date = "2022-04-20"
description = "C"
toc= true
math= true
tags = [
    "riscv","kernel"
]
+++

简单分析了下 relocate 和 setup_vm

首先是 setup_vm , 它设置了两个页表:

early_pg_dir,将内核自身处于的连续物理内存区域映射到位于PAGE_OFFSET的虚拟内存地址上
            而FIXMAP映射并没有映射全部的FIXMAP，而是仅仅影射了FIX_FDT部分，让后续的代码可以访问设备树。

trampoline_pg_dir  将PAGE_OFFSET后长为PMD_SIZE的区域映射到load_pa，即内核起始被装载后的起始物理内存地址。

call relocate的地方:

```
	/* Enable virtual memory and relocate to virtual address */
	la a0, swapper_pg_dir			//a0存储页表的物理地址
	XIP_FIXUP_OFFSET a0
	call relocate_enable_mmu
```

参数a0存储 页表的物理地址

```
//a0此时存储页表的物理地址
.align 2
#ifdef CONFIG_MMU
	.global relocate_enable_mmu
relocate_enable_mmu:
	/* Relocate return address */
	la a1, kernel_map        //a1 = kermap的地址
	XIP_FIXUP_OFFSET a1
	REG_L a1, KERNEL_MAP_VIRT_ADDR(a1)
	la a2, _start			//a2 = 镜像的地址
	sub a1, a1, a2        //a1 = a1 - a2				//80001012
	add ra, ra, a1			//得到ra的虚拟地址					//80201014

	/* Point stvec to virtual address of intruction after satp write */
	la a2, 1f				//a2  = label1 的地址
	add a2, a2, a1			//计算1f的新地址?				//8000101e
	csrw CSR_TVEC, a2            //mtec								//80201020

	/* Compute satp for kernel page tables, but don't load it yet */
	srl a2, a0, PAGE_SHIFT		//a2 = a0>>PAGE_SHIFT 12	应该是PPN	 //80201024
	la a1, satp_mode			//a1 = satp_mode的地址
	REG_L a1, 0(a1)
	or a2, a2, a1             //这里拼凑了stap寄存器				//80201032

	/*
	 * Load trampoline page directory, which will cause us to trap to
	 * stvec if VA != PA, or simply fall through if VA == PA.  We need a
	 * full fence here because setup_vm() just wrote these PTEs and we need
	 * to ensure the new translations are in use.
	 */
	 
	la a0, trampoline_pg_dir				//a0 = t表的地址
	XIP_FIXUP_OFFSET a0
	srl a0, a0, PAGE_SHIFT					//取t表地址的高部分		//8000103c
	or a0, a0, a1							//或 satp_mode		//8000103e
	sfence.vma								//80001040
	csrw CSR_SATP, a0					    //设置CSR_SATP寄存器  //80001044
.align 2
1:
	/* Set trap vector to spin forever to help debug */
	la a0, .Lsecondary_park
	csrw CSR_TVEC, a0							//80001050
	
	/* Reload the global pointer */
.option push
.option norelax
	la gp, __global_pointer$
.option pop

	/*
	 * Switch to kernel page tables.  A full fence is necessary in order to
	 * avoid using the trampoline translations, which are only correct for
	 * the first superpage.  Fetching the fence is guaranteed to work
	 * because that first superpage is translated the same way.
	 */
	csrw CSR_SATP, a2					//8000105c
	sfence.vma

	ret										//80001064

```



trampoline_pg_dir表实际上是多余的,
