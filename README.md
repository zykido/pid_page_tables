# pid_page_tables
find task page tables (arm64)


1.insmod ko
2.用法如下：https://blog.csdn.net/maybeYoc/article/details/115909592#:~:text=%E5%AF%B9%E4%B8%8EARM64%E7%94%A8%E4%BA%8Epage%E6%98%A0%E5%B0%84%E5%8C%BA%EF%BC%8Clinux%E5%86%85%E6%A0%B8%E7%94%A8page%E7%BB%93%E6%9E%84%E4%BD%93%E7%AE%A1%E7%90%86%E6%89%80%E6%9C%89%E7%89%A9%E7%90%86%E5%86%85%E5%AD%98%EF%BC%8C%E6%AF%8F%E4%B8%80%E9%A1%B5%E5%A4%A7%E5%B0%8F%E4%B8%BAPAGE_SIZE%E5%AF%B9%E4%BA%8Earm64%EF%BC%8C%E5%8F%AF%E8%83%BD%E6%98%AF4K%2C16K%2C64K%E3%80%82%E8%80%8C%E4%B8%BA%E4%BA%86%E5%BF%AB%E9%80%9F%E6%96%B9%E4%BE%BF%E6%89%BE%E5%88%B0%E5%AF%B9%E5%BA%94%E7%89%A9%E7%90%86%E9%A1%B5%E8%80%8C%E5%B0%86%E6%89%80%E6%9C%89%E7%9A%84%E9%A1%B5%E5%B8%A7%E7%BB%93%E6%9E%84%E4%BD%93%E6%98%A0%E5%B0%84%E5%88%B0%E6%AD%A4%E5%8C%BA%E5%9F%9F%EF%BC%8C%E5%90%8E%E7%BB%AD%E5%8F%AA%E9%9C%80%E4%BD%BF%E7%94%A8virt_to_page%2C,phys_to_page%E7%AD%89%E5%AE%8F%E5%AE%9E%E7%8E%B0%E8%99%9A%E6%8B%9F%E5%9C%B0%E5%9D%80%EF%BC%8C%E7%89%A9%E7%90%86%E5%9C%B0%E5%9D%80%E5%88%B0%E5%AF%B9%E5%BA%94%E9%A1%B5%E7%BB%93%E6%9E%84%E4%BD%93%E7%9A%84%E5%BF%AB%E9%80%9F%E6%9F%A5%E8%AF%A2%E3%80%82

一. 打印内核空间地址分布
使用命令echo kernel > /sys/kernel/debug/pid_page_tables,部分打印如下：

root@xxxxx:~# insmod pid_page_tables.ko 
root@xxxxx:~# echo kernel > /sys/kernel/debug/pid_page_tables
root@xxxxx:~# cat /sys/kernel/debug/pid_page_tables 
-----[Kernel space]-----

---[ Modules start ]---
0xffff000000d40000-0xffff000000d41000           4K PTE       ro x  SHD AF NG         UXN MEM/NORMAL
0xffff000000d41000-0xffff000000d42000           4K PTE       ro NX SHD AF NG         UXN MEM/NORMAL
0xffff000000d42000-0xffff000000d44000           8K PTE       RW NX SHD AF NG         UXN MEM/NORMAL
---[ Modules end ]---
---[ vmalloc() Area ]---
0xffff000008000000-0xffff000008001000           4K PTE       RW NX SHD AF NG         UXN DEVICE/nGnRE
0xffff000008002000-0xffff000008004000           8K PTE       RW NX SHD AF NG         UXN DEVICE/nGnRE
0xffff000008005000-0xffff000008006000           4K PTE       RW NX SHD AF NG         UXN DEVICE/nGnRE
0xffff000008007000-0xffff000008008000           4K PTE       RW NX SHD AF NG         UXN DEVICE/nGnRE
...
0xffff7dfffee00000-0xffff7dfffee10000          64K PTE       RW NX SHD AF NG         UXN DEVICE/nGnRE
---[ PCI I/O end ]---
---[ vmemmap start ]---
0xffff7e0000000000-0xffff7e0001000000          16M PMD       RW NX SHD AF NG     BLK UXN MEM/NORMAL
0xffff7e0002000000-0xffff7e0004000000          32M PMD       RW NX SHD AF NG     BLK UXN MEM/NORMAL
0xffff7e0006000000-0xffff7e0008000000          32M PMD       RW NX SHD AF NG     BLK UXN MEM/NORMAL
---[ vmemmap end ]---
---[ Linear Mapping ]---
0xffff800018000000-0xffff800018100000           1M PTE       RW NX SHD AF NG CON     UXN MEM/NORMAL
0xffff800080000000-0xffff800084000000          64M PMD       RW NX SHD AF NG CON BLK UXN MEM/NORMAL
0xffff800084000000-0xffff800084e00000          14M PMD       ro NX SHD AF NG     BLK UXN MEM/NORMAL
0xffff800084e00000-0xffff800084f00000           1M PTE       ro NX SHD AF NG         UXN MEM/NORMA

如上，使用echo设置了/sys/kernel/debug/pid_page_tables后cat /sys/kernel/debug/pid_page_tables将会显示所有地址空间映射属性。

内核地址空间分布说明
[Modules start] - [Modules end]
模块加载使用到的内核空间，现在只加载了pid_page_tables.ko故只显示了三行映射数据。
[vmalloc() Area] - [vmalloc() End]
所有vmalloc申请的，内核本身代码，mmap，ioremap等申请的地址空间在这一个区域。
[Fixmap start] - [Fixmap end]
arm64内核启动后为了能够读取设备树所做的一段Fixmap，用于前期映射使用其中包括了设备树映射，early ioremap，fix pgd，fix pud，fix pmd等的映射。
[PCI I/O start] - [PCI I/O end]
同上，专门用于PCI设备使用的地址空间，一般映射大小为16M
[vmemmap start] - [vmemmap end]
对与ARM64用于page映射区，linux内核用page结构体管理所有物理内存，每一页大小为PAGE_SIZE对于arm64，可能是4K,16K,64K。而为了快速方便找到对应物理页而将所有的页帧结构体映射到此区域，后续只需使用virt_to_page, phys_to_page等宏实现虚拟地址，物理地址到对应页结构体的快速查询。
[Linear Mapping]
此区域为线性映射区，所有的物理内存通过mem_map全部映射到此区域，用于伙伴系统分配，并且可使用virt_to_phys, phys_to_virt进行地址转换。
地址空间port属性说明
第一行
当前页表的映射范围地址
第二行
代表此映射范围大小
PMD PUD PTE
当标识为PMD PUD表示当前映射为block映射，如当前页表为4K,则pud的block映射一次性可映射1G范围，pmd的block映射可一次性映射2M范围。当标识为PTE表示为页表映射即PAGE_SIZE大小4K。
USR
AP标记，用于标识当前范围是否在用户空间还是内核空间可读可写或者仅读。
ro RW
当前地址范围读写权限，ro仅读，RW可读可写
NX x
NX表述当前范围特权级别模式不可执行，就是内核不能执行这段地址代码。
x表述当前范围特权级别模式可执行，就是内核的可执行代码段，在内核中这段一般指向内核的text*段
SHD
表示可共享属性，在arm64上表述为多核之间可共享其页表可见
AF
访问标志，当首次映射页表时，如果不置位，则第一次访问将会产生异常，可用于标记新页的首次访问，对于内核而言首次映射会将此bit置位。
NG
此标志可用于全局一致性。表示非全局。
CON
对于cpu进行地址翻译过程，为了能够避免多次的查找翻译过程，CON bit可用于标识当前映射为一个连续区域，则cpu可进行连续预取加速翻译过程。
BLK
对于PUD和PMD block映射的都会被标记为BLK
UXN MEM
标识非特权模式是否可执行，UXN表示当前范围用户空间不可执行，MEM表示当前范围用户空间可执行。
NORMAL/NORMAL-NC DEVICE/DEVICE/nGnRE等
arm64中一共标识了六种内存属性，下面说明使用到的：
对于系统RAM区，一般标识为NORMAL即普通内存可缓存，NORMAL-NC则不会经过cache cpu直接进行内存读写。
DEVICE/DEVICE/nGnRE等用于标识设备地址空间，此段空间除了不会经过cache外，cpu还会进行透传回写排序等操作限制，具体读写方式与G R E标识相关，感兴趣可参考ARMv8手册。
二. 打印内核空间虚拟地址对应的物理地址
使用命令echo kernel addr > /sys/kernel/debug/pid_page_tables,打印如下：

root@xxxxx:~# 
root@xxxxx:~# echo kernel 0xffff00000a634000 > /sys/kernel/debug/pid_page_tables
root@xxxxx:~# cat /sys/kernel/debug/pid_page_tables 
-----[Kernel space]-----

Find virt addr: 0xffff00000a634000
Find result: I/O MEM or Resvered RAM [0x0000000031000000] No Mem Map
root@xxxxx:~# 
root@xxxxx:~# echo kernel 0xffff8000c0800000 > /sys/kernel/debug/pid_page_tables                  
root@xxxxx:~# cat /sys/kernel/debug/pid_page_tables                             
-----[Kernel space]-----

Find virt addr: 0xffff8000c0800000
Find result: System RAM [0x00000000c0800000] Mem Map
root@xxxxx:~# echo kernel 0xffff005 > /sys/kernel/debug/pid_page_tables                  
root@xxxxx:~# cat /sys/kernel/debug/pid_page_tables                    
-----[Kernel space]-----

Find virt addr: 0x000000000ffff005
Find result: (null)
root@xxxxx:~# echo kernel 0xffff00000a611000 > /sys/kernel/debug/pid_page_tables         
root@xxxxx:~# cat /sys/kernel/debug/pid_page_tables                             
-----[Kernel space]-----

Find virt addr: 0xffff00000a611000
Find result: top [0x0000000033001000] No Mem Map
root@xxxxx:~# 
root@xxxxx:~#
如上输入对应的虚拟地址，cat会显示出对应的物理地址及地址是内存区还是外设地址区。
另外，对于线性映射区，Fixmap等区如果打印虚拟地址存在而物理地址找不到，则是设备树中被标记为了no-map属性不在内核映射范围以内，或者fixmap区有部分为临时映射区用于映射页表此段只映射到了pmd则也会找不pte。
对于内核管理的系统RAM将会被标记为Mem Map，否则就是No Mem Map。

三. 打印用户空间地址分布
使用命令echo pid 587 > /sys/kernel/debug/pid_page_tables,部分打印如下：

root@xxxxx:~# 
root@xxxxx:~# echo pid 587 > /sys/kernel/debug/pid_page_tables 
root@xxxxx:~# cat /sys/kernel/debug/pid_page_tables 
-----[User space]-----
Find pid comm: a.out
Real address space distribution:
        Text code:    0x0000aaaad9a1e000-0x0000aaaad9a1e99c
        Data:         0x0000aaaad9a2ed78-0x0000aaaad9a2f010
        Brk (heap):   0x0000aaaaf88b6000-0x0000aaaaf88d7000
        Mmap (logic): 0x0000ffffb688e000-0x0000ffffb66f3000
        Stack top:    0x0000ffffc59fe320
        Arg:          0x0000ffffc59febab-0x0000ffffc59febb3
        Env:          0x0000ffffc59febb3-0x0000ffffc59feff0

---[ Start code ]---
0x0000aaaad9a1e000-0x0000aaaad9a1f000           4K PTE   USR ro NX SHD AF NG         MEM/NORMAL
---[ End code ]---
---[ Start data ]---
0x0000aaaad9a2e000-0x0000aaaad9a2f000           4K PTE   USR ro NX SHD AF NG         UXN MEM/NORMAL
0x0000aaaad9a2f000-0x0000aaaad9a30000           4K PTE   USR RW NX SHD AF NG         UXN MEM/NORMAL
---[ End data ]---
---[ Start brk (heap) ]---
0x0000aaaaf88b6000-0x0000aaaaf88b8000           8K PTE   USR RW NX SHD AF NG         UXN MEM/NORMAL
---[ End brk (heap) ]---
---[ Mmap end ]---
0x0000ffffb66f3000-0x0000ffffb66f4000           4K PTE   USR ro NX SHD AF NG         MEM/NORMAL
0x0000ffffb66f5000-0x0000ffffb671d000         160K PTE   USR ro NX SHD AF NG         MEM/NORMAL
0x0000ffffb671e000-0x0000ffffb6730000          72K PTE   USR ro NX SHD AF NG         MEM/NORMAL
0x0000ffffb6750000-0x0000ffffb677e000         184K PTE   USR ro NX SHD AF NG         MEM/NORMAL
0x0000ffffb677f000-0x0000ffffb6780000           4K PTE   USR ro NX SHD AF NG         MEM/NORMAL
0x0000ffffb6790000-0x0000ffffb67a0000          64K PTE   USR ro NX SHD AF NG         MEM/NORMAL
0x0000ffffb67b0000-0x0000ffffb67c8000          96K PTE   USR ro NX SHD AF NG         MEM/NORMAL
0x0000ffffb67c9000-0x0000ffffb67d0000          28K PTE   USR ro NX SHD AF NG         MEM/NORMAL
0x0000ffffb67f0000-0x0000ffffb6800000          64K PTE   USR ro NX SHD AF NG         MEM/NORMAL
0x0000ffffb6810000-0x0000ffffb6820000          64K PTE   USR ro NX SHD AF NG         MEM/NORMAL
0x0000ffffb6853000-0x0000ffffb6856000          12K PTE   USR ro NX SHD AF NG         UXN MEM/NORMAL
0x0000ffffb6856000-0x0000ffffb685a000          16K PTE   USR RW NX SHD AF NG         UXN MEM/NORMAL
0x0000ffffb685b000-0x0000ffffb685c000           4K PTE   USR RW NX SHD AF NG         UXN MEM/NORMAL
0x0000ffffb685c000-0x0000ffffb687b000         124K PTE   USR ro NX SHD AF NG         MEM/NORMAL
0x0000ffffb6887000-0x0000ffffb6889000           8K PTE   USR RW NX SHD AF NG         UXN MEM/NORMAL
0x0000ffffb688a000-0x0000ffffb688b000           4K PTE   USR ro NX SHD AF NG         MEM/NORMAL
0x0000ffffb688b000-0x0000ffffb688c000           4K PTE   USR ro NX SHD AF NG         UXN MEM/NORMAL
0x0000ffffb688c000-0x0000ffffb688e000           8K PTE   USR RW NX SHD AF NG         UXN MEM/NORMAL
---[ Mmap base ]---
---[ Misc start ]---
0x0000ffffc59fc000-0x0000ffffc59ff000          12K PTE   USR RW NX SHD AF NG         UXN MEM/NORMAL
---[ Misc end ]---
root@xxxxx:~# 
root@xxxxx:~#

如上所示，将会打印出该任务是内核任务还是用户任务，并且还会打印任务名，并显示该任务的各个段分布。

Text code
代码段，自底向上。
Data
仅读数据段和数据段， 自底向上。
Brk
堆区，自底向上。
Mmap逻辑区
由于text，data等也可以是mmap映射区，故对于一个使用共享库，或自己使用mmap的任务来说此区域是一个逻辑的mmap区。自顶向下。
Stack top
栈顶，自顶向下
Arg
参数区，由任务启动时设置，一般由用户执行命令时设置参数，自底向上
Env
环境变量区，任务可通过此区域获取环境变量，自底向上
注意，对于内核任务没有自己的mm_struct结构体，也就不存在上述各区段，与内核共享地址空间并借用上个用户任务的mm结构体。

四. 打印用户空间虚拟地址对应的物理地址
使用命令echo pid 587 0xaaaad9a1e000 > /sys/kernel/debug/pid_page_tables,部分打印如下：

root@xxxxx:~# 
root@xxxxx:~# echo pid 587 0xaaaad9a1e000
pid 587 0xaaaad9a1e000
root@xxxxx:~# echo pid 587 0xaaaad9a1e000 > /sys/kernel/debug/pid_page_tables 
root@xxxxx:~# cat /sys/kernel/debug/pid_page_tables 
-----[User space]-----
Find pid comm: a.out
Real address space distribution:
        Text code:    0x0000aaaad9a1e000-0x0000aaaad9a1e99c
        Data:         0x0000aaaad9a2ed78-0x0000aaaad9a2f010
        Brk (heap):   0x0000aaaaf88b6000-0x0000aaaaf88d7000
        Mmap (logic): 0x0000ffffb688e000-0x0000ffffb66f3000
        Stack top:    0x0000ffffc59fe320
        Arg:          0x0000ffffc59febab-0x0000ffffc59febb3
        Env:          0x0000ffffc59febb3-0x0000ffffc59feff0

Find virt addr: 0x0000aaaad9a1e000
Find result: System RAM [0x00000001e450e000] Mem Map
root@xxxxx:~#

打印同内核空间相同，另外注意，如果查找的虚拟地址在Brk中存在但确没有对应的物理地址原因则可能是向内核申请的空间应用程序还未使用，而内核只做了页表映射并没有填充实际的物理地址，只有在应用程序实际访问时才会通过缺页异常来填充物理页。

五. 打印用户空间MMAP详细信息
使用命令echo pid 1 mmap > /sys/kernel/debug/pid_page_tables,部分打印如下：

root@a1000:/sys/kernel/debug# 
root@a1000:/sys/kernel/debug# 
root@a1000:/sys/kernel/debug# echo pid 1 mmap > pid_page_tables 
root@a1000:/sys/kernel/debug# cat pid_page_tables 
-----[User space]-----
Find pid comm: systemd
Real address space distribution:
        Text code:    0x0000000000400000-0x0000000000501a8c
        Data:         0x00000000005122b8-0x000000000054e458
        Brk (heap):   0x000000002819c000-0x00000000282cc000
        Mmap (logic): 0x0000ffffb1f81000-0x0000ffffa4000000
        Stack top:    0x0000ffffc49fc980
        Arg:          0x0000ffffc49fcfa8-0x0000ffffc49fcfb3
        Env:          0x0000ffffc49fcfb3-0x0000ffffc49fcfed

0x0000000000400000-0x0000000000502000   Mmap file name [systemd]
        Flags raw: [0x0000000000000875]  READ EXEC MAYREAD MAYWRITE MAYEXEC DENYWRITE

0x0000000000512000-0x000000000054e000   Mmap file name [systemd]
        Flags raw: [0x0000000000100871]  READ MAYREAD MAYWRITE MAYEXEC DENYWRITE ACCOUNT

0x000000000054e000-0x000000000054f000   Mmap file name [systemd]
        Flags raw: [0x0000000000100873]  READ WRITE MAYREAD MAYWRITE MAYEXEC DENYWRITE ACCOUNT

0x000000002819c000-0x00000000282cc000   Anonymous mapping
        Flags raw: [0x0000000000100073]  READ WRITE MAYREAD MAYWRITE MAYEXEC ACCOUNT

0x0000ffffa4000000-0x0000ffffa4021000   Anonymous mapping
        Flags raw: [0x0000000000200073]  READ WRITE MAYREAD MAYWRITE MAYEXEC NORESERVE

0x0000ffffa4021000-0x0000ffffa8000000   Anonymous mapping
        Flags raw: [0x0000000000200070]  MAYREAD MAYWRITE MAYEXEC NORESERVE

0x0000ffffac000000-0x0000ffffac021000   Anonymous mapping
        Flags raw: [0x0000000000200073]  READ WRITE MAYREAD MAYWRITE MAYEXEC NORESERVE

0x0000ffffac021000-0x0000ffffb0000000   Anonymous mapping
        Flags raw: [0x0000000000200070]  MAYREAD MAYWRITE MAYEXEC NORESERVE

0x0000ffffb08a3000-0x0000ffffb08a4000   Anonymous mapping
        Flags raw: [0x0000000000000070]  MAYREAD MAYWRITE MAYEXEC

0x0000ffffb08a4000-0x0000ffffb10a4000   Anonymous mapping
        Flags raw: [0x0000000000100073]  READ WRITE MAYREAD MAYWRITE MAYEXEC ACCOUNT
root@xxxxx:~#

