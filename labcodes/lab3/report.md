#Lab3 Report
2013011321 范子豪

##练习1：给未被映射的地址映射上物理页

- 如果获得pte是失败，那么需要报错退出
- 如果pte是空的那么
    - 如果swap_init_ok为false，则报错退出
    - 否则分配物理页
- 如果pte不是空的，那么页面需要被换出，换出需要更新swap_map_swappable 

####请描述页目录项（Pag Director Entry）和页表（Page Table Entry）中组成部分对ucore实现页替换算法的潜在用处。
PTE可以记录虚拟地址的状态，可以记录页被换出后所对应的扇区号，避免额外记录换出地址，同时还能记录权限相关的信息。

####如果ucore的缺页服务例程在执行过程中访问内存，出现了页访问异常，请问硬件要做哪些事情？
继续正常执行终端服务例程，如果无法解决，则报错。

##练习2：补充完成基于FIFO的页面替换算法

####如果要在ucore上实现"extended clock页替换算法"请给你的设计方案，现有的swap_manager框架是否足以支持在ucore中实现此算法？如果是，请给你的设计方案。如果不是，请给出你的新的扩展和基此扩展的设计方案。并需要回答如下问题

    - 需要被换出的页的特征是什么？
    - 在ucore中如何判断具有这样特征的页？
    - 何时进行换入和换出操作？

支持。
需要给page额外维护是否修改和是否访问（增加修改位和访问位）。ucore可以通过对这两位进行逻辑运算找到被换出的页。
此时需要被换出的页应该这两个标记为都为0.
修改_fifo_map_swappable和_fifo_swap_out_victim两个函数。在map时进行标记为修改，在swap_out时通过这两位决定是否弹出。

##Summary
本次lab修改的文件包括:

####from lab1:
- kern/debug/kdebug.c
- kern/init/init.c
- kern/trap/trap.c

####from lab2:
- kern/mm/defult_pmm.c
- kern/mm/pmm.c

####from lab3:
- kern/mm/vmm.c
- kern/mm/swap_fifo.c
