#Lab8 Report
2013011321 范子豪

##EX1 完成读写文件操作的实现

####实现

sfs_io_nolock函数:

1. 根据读写操作 得到相应的函数指针
2. 得到inode指针
3. 计算待读取的block位置和数量
4. 开始读写block
    - 读取buf_op 即某block的后半部分 
    - 读取连续的block
    - 读取某block的前半部分

####UNIX PIPE机制
首先为所有的程序创建一个新的线程, 通过分别重定向对应的输入输出, 来实现PIPE机制. 重定向可以通过修改device实现.

##EX2 完成基于文件系统的执行程序机制的实现

load_icode函数:

目的是将硬盘中的code加载到内存并且设定程序运行的入口, 使其在用户态执行

1. 准备内存
    - 运行需要的也表和内存空间需要申请并清空

        if ((mm = mm_create()) == NULL) {
            goto bad_mm;
        }
        if (setup_pgdir(mm) != 0) {
            goto bad_pgdir_cleanup_mm;
        }

2. 将elf格式的程序从硬盘读取到内存中
    - 需要将BSS段 代码段逐个加入
    - 硬盘位置和内存位置的区别
    - 物理地址与虚拟地址的转化
    - 设定程序运行的argc, argc参数
            //setup argc, argv
            uint32_t argv_size=0, i;
            for (i = 0; i < argc; i ++) {
                argv_size += strnlen(kargv[i],EXEC_MAX_ARG_LEN + 1)+1;
            }

            uintptr_t stacktop = USTACKTOP - (argv_size/sizeof(long)+1)*sizeof(long);
            char** uargv=(char **)(stacktop  - argc * sizeof(char *));

            argv_size = 0;
            for (i = 0; i < argc; i ++) {
                uargv[i] = strcpy((char *)(stacktop + argv_size ), kargv[i]);
                argv_size +=  strnlen(kargv[i],EXEC_MAX_ARG_LEN + 1)+1;
            }
    - 设定中断帧，包括程序入口以及程序的用户态堆栈等等
            memset(tf, 0, sizeof(struct trapframe));
            tf->tf_cs = USER_CS;
            tf->tf_ds = tf->tf_es = tf->tf_ss = USER_DS;
            tf->tf_esp = stacktop;
            tf->tf_eip = elf->e_entry;
            tf->tf_eflags = FL_IF;

####实现

####UNIX的硬链接和软链接机制设计
对于硬链接, 可以再inode中增加一个计数器, 每有一个硬链接计数器加一.  
对于软连接, 可以设置一个是否为软连接的标记位, 如果是软链接, 则索引你快中的内存是软连接目的地址