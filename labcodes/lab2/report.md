#Lab2 Report
2013011321 范子豪

##练习1：实现 first-fit 连续物理内存分配算法

在实现first fit 内存分配算法的回收函数时，要考虑地址连续的空闲块之间的合并操作。提示:在建立空闲页块链表时，需要按照空闲页块起始地址来排序，形成一个有序的链表。可能会修改default_pmm.c中的default_init，default_init_memmap，default_alloc_pages， default_free_pages等相关函数。请仔细查看和理解default_pmm.c中的注释。

####实现简述
- 初始化：将所有的page frame都插入free_list中，每个连续的空闲内存块的一个page的property为该空闲快的大小，其余的页的property设置为0
- alloc: 分配page frame的时候，分配第一个满足大小要求的。
- free: 释放物理页的时候，需要把释放的物理页块插入free_list的原位置，并且与前后块合并。

####你的first fit算法是否有进一步的改进空间?
可能的改进：
- 利用二叉搜索树加速分配物理页查找

##练习2：实现寻找虚拟地址对应的页表项
通过设置页表和对应的页表项，可建立虚拟内存地址和物理内存地址的对应关系。其中的get_pte函数是设置页表项环节中的一个重要步骤。此函数找到一个虚地址对应的二级页表项的内核虚地址，如果此二级页表项不存在，则分配一个包含此项的二级页表。本练习需要补全get_pte函数 in kern/mm/pmm.c，实现其功能。请仔细查看和理解get_pte函数中的注释。

####实现简述
get_pte()函数中，通过la找到PDE。
如果PDE不存在，则创建一个PDE，其标志位为PTE_P | PTE_W | PTE_U
最后通过PDE和la找到PTE。


####请描述页目录项（Pag Director Entry）和页表（Page Table Entry）中每个组成部分的含义和以及对ucore而言的潜在用处。

- PDE的高20位位对应的页表的起始物理地址，低12位是标志位。
- PTE的高20位位对应物理页的起始物理地址，低12位是该物流页的标志位。

####如果ucore执行过程中访问内存，出现了页访问异常，请问硬件要做哪些事情？

1. 产生中断
2. __alltraps(trapentry.s)
3. trap(trap.c)
4. trap_dispatch(trap.c)
5. panic!!!

##练习3：释放某虚地址所在的页并取消对应二级页表项的映射
当释放一个包含某虚地址的物理内存页时，需要让对应此物理内存页的管理数据结构Page做相关的清除处理，使得此物理内存页成为空闲；另外还需把表示虚地址与物理地址对应关系的二级页表项清除。请仔细查看和理解page_remove_pte函数中的注释。为此，需要补全在 kern/mm/pmm.c中的page_remove_pte函数。

####实现简述

remove_pte()函数中，将PTE置零。

####数据结构Page的全局变量（其实是一个数组）的每一项与页表中的页目录项和页表项有无对应关系？如果有，其对应关系是啥？
- PTE每项的高20位，是对应物理页的物理地址x.对应的物理页是Page的x>>12项
- PDE每一项的高20位，都是对应的页表的物理地址. 对应的页表是Page的x>>12项


####如果希望虚拟地址与物理地址相等，则需要如何修改lab2，完成此事？ 鼓励通过编程来具体完成这个问题
- 可以通过修改kern/mm/pmm.h中的PADDR()和KADDR，分别去掉-+kernbase。


##Summary
本次lab修改的文件包括:

####from lab1:
- kern/debug/kdebug.c
- kern/init/init.c
- kern/trap/trap.c

####from lab2:
- kern/mm/defult_pmm.c
- kern/mm/pmm.c
