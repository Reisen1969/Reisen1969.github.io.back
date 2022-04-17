+++
author = "rayrain"
title = "qemu梳理(riscv64用户模式)"
date = "2022-04-09"
description = "qemu"
toc= true
math= true
tags = [
    "qemu"
]

+++
## 函数执行流程

```
main
	cpu_loop
		cpu_exec
			tb_gen_code
				gen_intermediate_code
					translator_loop
						riscv_tr_translate_insn
							decode_opc
								decode_insn16
									trans_add(/target/riscv/insn_trans/trans_rvi.c.inc)
										gen_arith(/target/riscv/translate.c)
```

## decode文件的内容和用途

从上到下的顺序,是:

### Fields字段

```
# Fields:
%rs3       27:5
%rs2       20:5
%rs1       15:5
%rd        7:5
%sh5       20:5
%sh6       20:6
```

表示参与指令的rs和rd在编码中的位置

### immediate立即数

```
# immediates:
%imm_i    20:s12
%imm_s    25:s7 7:5
%imm_b    31:s1 7:1 25:6 8:4     !function=ex_shift_1
%imm_j    31:s1 12:8 20:1 21:10  !function=ex_shift_1
%imm_u    12:s20                 !function=ex_shift_12
```

和Fields作用差不多,表示各种立即数在指令编码中的位置

### Argument sets参数集

```
&empty
&b    imm rs2 rs1
&i    imm rs1 rd
&j    imm rd
```

表示每一种类型的指令编码中 参与的立即数和寄存器

python脚本会根据这些生成对应的结构体,如:

```
typedef struct {
} arg_empty;
typedef struct {
    int imm;
    int rs1;
    int rd;
} arg_i;

typedef struct {
    int imm;
    int rd;
} arg_j;
```

###  Formats指令格式

```
# Formats 32:
@r       .......   ..... ..... ... ..... ....... &r                %rs2 %rs1 %rd
@i       ............    ..... ... ..... ....... &i      imm=%imm_i     %rs1 %rd
@b       .......   ..... ..... ... ..... ....... &b      imm=%imm_b %rs2 %rs1
@s       .......   ..... ..... ... ..... ....... &s      imm=%imm_s %rs2 %rs1
@u       ....................      ..... ....... &u      imm=%imm_u          %rd
@j       ....................      ..... ....... &j      imm=%imm_j          %rd
```

每行都是32个.  表示指令的编码格式

用空格将每部分分开

最后面的是 寄存器 和 立即数 等,它们的字段在上文中已确认

### 详细的指令集

这部分基本上就是把指令集从官方文档中搬了过来,

python脚本会根据这些指令生成`trans_xxx`的函数声明,具体的函数定义需要自己编写.





## 宏

### MAKE_64BIT_MASK

位于/include/qemu/bitops.h

```
#define MAKE_64BIT_MASK(shift, length) \
    (((~0ULL) >> (64 - (length))) << (shift))
```

0ULL: unsinged long long zero   64-bit的无符号0

作用:制造一个64位的掩码,该掩码从第shift位开始,长度是length,如下:

```
0000 ....0 11 .... .111 0.... 000
           |--length--|--shift---|

```



### FIELD

位于/include/hw/registerfields.h

```
#define FIELD(reg, field, shift, length)                                  \
    enum { R_ ## reg ## _ ## field ## _SHIFT = (shift)};                  \
    enum { R_ ## reg ## _ ## field ## _LENGTH = (length)};                \
    enum { R_ ## reg ## _ ## field ## _MASK =                             \
                                        MAKE_64BIT_MASK(shift, length)};
```

这个宏定义了三个枚举 ,各自只有一个成员

R_reg_field_SHIFT = shift

R_reg_field_LENGTH = length

R_reg_field_MASK = 对应length和shift的掩码



## 重要的结构体

### CPUArchState(某个arch的cpu状态)

根据不同的guest,会被定义为其它名称,如:CPURISCVState

> 事实上,CPUArchState 被定义于 target/riscv/cpu.h, 每种guest的这个结构体都不一样

```c
struct CPUArchState {
    target_ulong gpr[32];   //32个gpr寄存器,每个寄存器长32
    target_ulong gprh[32]; /* 128-bit寄存器的高64位,每个寄存器长32  */
    uint64_t fpr[32];     //32个浮点寄存器            /* assume both F and D extensions */

    /* vector coprocessor state. */
    uint64_t vreg[32 * RV_VLEN_MAX / 64] QEMU_ALIGNED(16);
    target_ulong vxrm;
    target_ulong vxsat;
    target_ulong vl;
    target_ulong vstart;
    target_ulong vtype;
    bool vill;

    target_ulong pc;
    target_ulong load_res;
    target_ulong load_val;

    target_ulong frm;

    target_ulong badaddr;
    uint32_t bins;

    target_ulong guest_phys_fault_addr;

    target_ulong priv_ver;
    target_ulong bext_ver;
    target_ulong vext_ver;

    /* RISCVMXL, but uint32_t for vmstate migration */
    uint32_t misa_mxl;      /* current mxl */
    uint32_t misa_mxl_max;  /* max mxl for this cpu */
    uint32_t misa_ext;      /* current extensions */
    uint32_t misa_ext_mask; /* max ext for this cpu */
    uint32_t xl;            /* current xlen */

    /* 128-bit helpers upper part return value */
    target_ulong retxh;

    uint32_t features;

#ifdef CONFIG_USER_ONLY
    uint32_t elf_flags;
#endif

#ifndef CONFIG_USER_ONLY
    target_ulong priv;
    /* This contains QEMU specific information about the virt state. */
    target_ulong virt;
    target_ulong geilen;
    target_ulong resetvec;

    target_ulong mhartid;
    /*
     * For RV32 this is 32-bit mstatus and 32-bit mstatush.
     * For RV64 this is a 64-bit mstatus.
     */
    uint64_t mstatus;

    uint64_t mip;

    uint64_t miclaim;

    uint64_t mie;
    uint64_t mideleg;

    target_ulong satp;   /* since: priv-1.10.0 */
    target_ulong stval;
    target_ulong medeleg;

    target_ulong stvec;
    target_ulong sepc;
    target_ulong scause;

    target_ulong mtvec;
    target_ulong mepc;
    target_ulong mcause;
    target_ulong mtval;  /* since: priv-1.10.0 */

    /* Machine and Supervisor interrupt priorities */
    uint8_t miprio[64];
    uint8_t siprio[64];

    /* AIA CSRs */
    target_ulong miselect;
    target_ulong siselect;

    /* Hypervisor CSRs */
    target_ulong hstatus;
    target_ulong hedeleg;
    uint64_t hideleg;
    target_ulong hcounteren;
    target_ulong htval;
    target_ulong htinst;
    target_ulong hgatp;
    target_ulong hgeie;
    target_ulong hgeip;
    uint64_t htimedelta;

    /* Hypervisor controlled virtual interrupt priorities */
    target_ulong hvictl;
    uint8_t hviprio[64];

    /* Upper 64-bits of 128-bit CSRs */
    uint64_t mscratchh;
    uint64_t sscratchh;

    /* Virtual CSRs */
    /*
     * For RV32 this is 32-bit vsstatus and 32-bit vsstatush.
     * For RV64 this is a 64-bit vsstatus.
     */
    uint64_t vsstatus;
    target_ulong vstvec;
    target_ulong vsscratch;
    target_ulong vsepc;
    target_ulong vscause;
    target_ulong vstval;
    target_ulong vsatp;

    /* AIA VS-mode CSRs */
    target_ulong vsiselect;

    target_ulong mtval2;
    target_ulong mtinst;

    /* HS Backup CSRs */
    target_ulong stvec_hs;
    target_ulong sscratch_hs;
    target_ulong sepc_hs;
    target_ulong scause_hs;
    target_ulong stval_hs;
    target_ulong satp_hs;
    uint64_t mstatus_hs;

    /* Signals whether the current exception occurred with two-stage address
       translation active. */
    bool two_stage_lookup;

    target_ulong scounteren;
    target_ulong mcounteren;

    target_ulong sscratch;
    target_ulong mscratch;

    /* temporary htif regs */
    uint64_t mfromhost;
    uint64_t mtohost;
    uint64_t timecmp;

    /* physical memory protection */
    pmp_table_t pmp_state;
    target_ulong mseccfg;

    /* machine specific rdtime callback */
    uint64_t (*rdtime_fn)(uint32_t);
    uint32_t rdtime_fn_arg;

    /* machine specific AIA ireg read-modify-write callback */
#define AIA_MAKE_IREG(__isel, __priv, __virt, __vgein, __xlen) \
    ((((__xlen) & 0xff) << 24) | \
     (((__vgein) & 0x3f) << 20) | \
     (((__virt) & 0x1) << 18) | \
     (((__priv) & 0x3) << 16) | \
     (__isel & 0xffff))
#define AIA_IREG_ISEL(__ireg)                  ((__ireg) & 0xffff)
#define AIA_IREG_PRIV(__ireg)                  (((__ireg) >> 16) & 0x3)
#define AIA_IREG_VIRT(__ireg)                  (((__ireg) >> 18) & 0x1)
#define AIA_IREG_VGEIN(__ireg)                 (((__ireg) >> 20) & 0x3f)
#define AIA_IREG_XLEN(__ireg)                  (((__ireg) >> 24) & 0xff)
    int (*aia_ireg_rmw_fn[4])(void *arg, target_ulong reg,
        target_ulong *val, target_ulong new_val, target_ulong write_mask);
    void *aia_ireg_rmw_fn_arg[4];

    /* True if in debugger mode.  */
    bool debugger;

    /*
     * CSRs for PointerMasking extension
     */
    target_ulong mmte;
    target_ulong mpmmask;
    target_ulong mpmbase;
    target_ulong spmmask;
    target_ulong spmbase;
    target_ulong upmmask;
    target_ulong upmbase;
#endif
    target_ulong cur_pmmask;
    target_ulong cur_pmbase;

    float_status fp_status;

    /* Fields from here on are preserved across CPU reset. */
    QEMUTimer *timer; /* Internal timer */

    hwaddr kernel_addr;
    hwaddr fdt_addr;

    /* kvm timer */
    bool kvm_timer_dirty;
    uint64_t kvm_timer_time;
    uint64_t kvm_timer_compare;
    uint64_t kvm_timer_state;
    uint64_t kvm_timer_frequency;
};
```

### CPUState(通用上的CPU状态)

里面会保存tb缓存

```
struct CPUState {
    /*< private >*/
    DeviceState parent_obj;
    /*< public >*/

    int nr_cores;
    int nr_threads;

    struct QemuThread *thread;
#ifdef _WIN32
    HANDLE hThread;
#endif
    int thread_id;
    bool running, has_waiter;
    struct QemuCond *halt_cond;
    bool thread_kicked;
    bool created;
    bool stop;
    bool stopped;

    /* Should CPU start in powered-off state? */
    bool start_powered_off;

    bool unplug;
    bool crash_occurred;
    bool exit_request;
    bool in_exclusive_context;
    uint32_t cflags_next_tb;
    /* updates protected by BQL */
    uint32_t interrupt_request;
    int singlestep_enabled;
    int64_t icount_budget;
    int64_t icount_extra;
    uint64_t random_seed;
    sigjmp_buf jmp_env;

    QemuMutex work_mutex;
    QSIMPLEQ_HEAD(, qemu_work_item) work_list;

    CPUAddressSpace *cpu_ases;
    int num_ases;
    AddressSpace *as;
    MemoryRegion *memory;

    CPUArchState *env_ptr;
    IcountDecr *icount_decr_ptr;

    /* Accessed in parallel; all accesses must be atomic */
    TranslationBlock *tb_jmp_cache[TB_JMP_CACHE_SIZE];

    struct GDBRegisterState *gdb_regs;
    int gdb_num_regs;
    int gdb_num_g_regs;
    QTAILQ_ENTRY(CPUState) node;

    /* ice debug support */
    QTAILQ_HEAD(, CPUBreakpoint) breakpoints;

    QTAILQ_HEAD(, CPUWatchpoint) watchpoints;
    CPUWatchpoint *watchpoint_hit;

    void *opaque;

    /* In order to avoid passing too many arguments to the MMIO helpers,
     * we store some rarely used information in the CPU context.
     */
    uintptr_t mem_io_pc;

    /* Only used in KVM */
    int kvm_fd;
    struct KVMState *kvm_state;
    struct kvm_run *kvm_run;
    struct kvm_dirty_gfn *kvm_dirty_gfns;
    uint32_t kvm_fetch_index;
    uint64_t dirty_pages;

    /* Used for events with 'vcpu' and *without* the 'disabled' properties */
    DECLARE_BITMAP(trace_dstate_delayed, CPU_TRACE_DSTATE_MAX_EVENTS);
    DECLARE_BITMAP(trace_dstate, CPU_TRACE_DSTATE_MAX_EVENTS);

    DECLARE_BITMAP(plugin_mask, QEMU_PLUGIN_EV_MAX);

#ifdef CONFIG_PLUGIN
    GArray *plugin_mem_cbs;
    /* saved iotlb data from io_writex */
    SavedIOTLB saved_iotlb;
#endif

    /* TODO Move common fields from CPUArchState here. */
    int cpu_index;
    int cluster_index;
    uint32_t tcg_cflags;
    uint32_t halted;
    uint32_t can_do_io;
    int32_t exception_index;

    /* shared by kvm, hax and hvf */
    bool vcpu_dirty;

    /* Used to keep track of an outstanding cpu throttle thread for migration
     * autoconverge
     */
    bool throttle_thread_scheduled;

    bool ignore_memory_transaction_failures;

    /* Used for user-only emulation of prctl(PR_SET_UNALIGN). */
    bool prctl_unalign_sigbus;

    struct hax_vcpu_state *hax_vcpu;

    struct hvf_vcpu_state *hvf;

    /* track IOMMUs whose translations we've cached in the TCG TLB */
    GArray *iommu_notifiers;
};
```

### ArchCPU

每一种arc都有一个这样的结构体,包含了上面两个结构体

```
struct ArchCPU {
    /*< private >*/
    CPUState parent_obj;
    /*< public >*/
    CPUNegativeOffsetState neg;
    CPURISCVState env;

    char *dyn_csr_xml;
    char *dyn_vreg_xml;

    /* Configuration Settings */
    RISCVCPUConfig cfg;
};

```



## 函数

### env_cpu

在cpu_loop函数中调用

```
/**
 * env_archcpu(env)
 * @env: The architecture environment
 *
 * Return the ArchCPU associated with the environment.
 */
static inline ArchCPU *env_archcpu(CPUArchState *env)
{
    return container_of(env, ArchCPU, env);
}

```

这个函数用于获取当前环境env的对应的CPUState







## 随便写写

### 关于tb寻找和生成

`cpu_exec`函数,是主循环,在没发生异常和中断的情况下会一直执行tb块

首先会使用`tb_lookup`在缓存中根据pc值等参数在缓存中寻找下一个tb块

如果没找到,就会根据相同的参数生成新的tb块













