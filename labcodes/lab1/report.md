# Lab1 report
范子豪 计32 2013011321

##练习1

**题目1**：操作系统镜像文件 ucore.img 是如何一步一步生成的?(需要比较详细地解释 Makefile 中每一条相关命令和命令参数的含义,以及说明命令导致的结果)

**答案1**：本题在UCOREIGM部分，参照answer的基础进行makefile的阅读
    
生成UCOREIMG， 需要prerequisite: bootblock和kernel的存在

    # create ucore.img
    UCOREIMG    := $(call totarget,ucore.img)

    $(UCOREIMG): $(kernel) $(bootblock)
        $(V)dd if=/dev/zero of=$@ count=10000
        $(V)dd if=$(bootblock) of=$@ conv=notrunc
        $(V)dd if=$(kernel) of=$@ seek=1 conv=notrunc

    $(call create_target,ucore.img)

其中，生成代码中的一些符号在function.mk中有定义，

    # list all files in some directories: (#directories, #types)
    listf = $(filter $(if $(2),$(addprefix %.,$(2)),%),\
              $(wildcard $(addsuffix $(SLASH)*,$(1))))

    # get .o obj files: (#files[, packet])
    toobj = $(addprefix $(OBJDIR)$(SLASH)$(if $(2),$(2)$(SLASH)),\
            $(addsuffix .o,$(basename $(1))))

    # get .d dependency files: (#files[, packet])
    todep = $(patsubst %.o,%.d,$(call toobj,$(1),$(2)))

    totarget = $(addprefix $(BINDIR)$(SLASH),$(1))

    # change $(name) to $(OBJPREFIX)$(name): (#names)
    packetname = $(if $(1),$(addprefix $(OBJPREFIX),$(1)),$(OBJPREFIX))

生成bootblock的代码如下，

    bootfiles = $(call listf_cc,boot)
    $(foreach f,$(bootfiles),$(call cc_compile,$(f),$(CC),$(CFLAGS) -Os -nostdinc))

    bootblock = $(call totarget,bootblock)

    $(bootblock): $(call toobj,$(bootfiles)) | $(call totarget,sign)
        @echo + ld $@
        $(V)$(LD) $(LDFLAGS) -N -e start -Ttext 0x7C00 $^ -o $(call toobj,bootblock)
        @$(OBJDUMP) -S $(call objfile,bootblock) > $(call asmfile,bootblock)
        @$(OBJCOPY) -S -O binary $(call objfile,bootblock) $(call outfile,bootblock)
        @$(call totarget,sign) $(call outfile,bootblock) $(bootblock)

    $(call create_target,bootblock)

可以看出，生成bootblock，会先生成bootasm.o, bootmain.o和sign。前面两个obj由$(call toobj,$(bootfiles))生成，编译器的选项由CFLAG定义，即调用，

    gcc -Iboot/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Os -nostdinc -c boot/bootasm.S -o obj/boot/bootasm.o
    gcc -Iboot/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc -fno-stack-protector -Ilibs/ -Os -nostdinc -c boot/bootmain.c -o obj/boot/bootmain.o

sign由以下两句代码生成，

    $(call add_files_host,tools/sign.c,sign,sign)
    $(call create_target_host,sign,sign)

得到三个prerequisite之后，展开bootblock的生成代码可得，

    ld -m elf_i386 -nostdlib -N -e start -Ttext 0x7C00 obj/boot/bootasm.o obj/boot/bootmain.o -o obj/bootblock.o
    objcopy -S -O binary obj/bootblock.o obj/bootblock.out
    利用sign处理bootblock.out得到bootblock
    bin/sign obj/bootblock.out bin/bootblock

生成kernel的相关代码是

    # create kernel target
    kernel = $(call totarget,kernel)

    $(kernel): tools/kernel.ld

    $(kernel): $(KOBJS)
        @echo + ld $@
        $(V)$(LD) $(LDFLAGS) -T tools/kernel.ld -o $@ $(KOBJS)
        @$(OBJDUMP) -S $@ > $(call asmfile,kernel)
        @$(OBJDUMP) -t $@ | $(SED) '1,/SYMBOL TABLE/d; s/ .* / /; /^$$/d' > $(call symfile,kernel)

    $(call create_target,kernel)

其中，由于上面定义了

    KSRCDIR    += kern/init \
               kern/libs \
               kern/debug \
               kern/driver \
               kern/trap \
               kern/mm

检查目录中的源文件，可以得到为了生成kernel，需要先生成kernel.ld init.o readline.o stdio.o kdebug.o kmonitor.o panic.o clock.o console.o intr.o picirq.o trap.o trapentry.o vectors.o pmm.o  printfmt.o string.o

回到UCOREIMG  
生成一个有10000个块的文件，每个块默认512字节，用0填充 dd if=/dev/zero of=bin/ucore.img count=10000  
把bootblock中的内容写到第一个块 dd if=bin/bootblock of=bin/ucore.img conv=notrunc  
从第二个块开始写kernel中的内容 dd if=bin/kernel of=bin/ucore.img seek=1 conv=notrunc  

**题目2**：一个被系统认为是符合规范的硬盘主引导扇区的特征是什么?

**答案2**：
通过阅读sing.c可以知道，一个符合规范的硬盘主引导扇区，需要满足   

- 大小为512字节
- 倒数第二个字节为0x55
- 倒数第一个字节为0xAA

##练习2

**题目1**：
从 CPU 加电后执行的第一条指令开始,单步跟踪 BIOS 的执行。

**答案1**：
根据附录B，修改/ab1/tools/gdbinit，

    set architecture i8086
    target remote :1234

然后运行

    make debug

可以看到qemu将控制权交给了gdb，可以通过si命令，在gdb中单步跟踪BIOS。


**题目2**：在初始化位置0x7c00 设置实地址断点,测试断点正常。

**答案2**：
将tools/gdbinit修改为
    
    file bin/kernel
    target remote :1234
    set architecture i8086
    b *0x7c00  
    continue          
    x /2i $pc
    set architecture i386

之后可以看到0x7c00处执行的代码为

    => 0x7c00:      cli    
       0x7c01:      cld

与bootasm.S中生成的汇编代码相符。


**题目3**：在调用qemu时增加-d in_asm -D q.log 参数，便可以将运行的汇编指令保存在q.log 中。 将执行的汇编代码与bootasm.S 和 bootblock.asm 进行比较，看看二者是否一致。

**答案3**：
在makefile中，修改debug部分，加入-d in_asm -D q.log参数，可以看到q.log中0x7c00行执行的汇编代码，与bootasm.S中一致。


##练习3

**题目1**：分析bootloader 进入保护模式的过程。

**答案1**：
根据要求，分析bootasm.S的16行起始的代码。

首先，在实模式下，清理环境，将段寄存器置零

    cli                                             # Disable interrupts
    cld                                             # String operations increment

    # Set up the important data segment registers (DS, ES, SS).
    xorw %ax, %ax                                   # Segment number zero
    movw %ax, %ds                                   # -> Data Segment
    movw %ax, %es                                   # -> Extra Segment
    movw %ax, %ss                                   # -> Stack Segment

接着开启A20，使得系统可以访问32位地址。

    seta20.1:
    inb $0x64, %al                                  # Wait for not busy(8042 input buffer empty).
    testb $0x2, %al
    jnz seta20.1

    movb $0xd1, %al                                 # 0xd1 -> port 0x64
    outb %al, $0x64                                 # 0xd1 means: write data to 8042's P2 port

    seta20.2:
    inb $0x64, %al                                  # Wait for not busy(8042 input buffer empty).
    testb $0x2, %al
    jnz seta20.2

    movb $0xdf, %al                                 # 0xdf -> port 0x60
    outb %al, $0x60                                 # 0xdf = 11011111, means set P2's A20 bit(the 1 bit) to 1

然后初始化lgdt表，

    lgdt gdtdesc

进入保护模式，将cr0寄存器的PE位使能
    
    movl %cr0, %eax
    orl $CR0_PE_ON, %eax
    movl %eax, %cr0

变更到32位模式

    ljmp $PROT_MODE_CSEG, $protcseg

设置段寄存器并建立堆栈，

    # Set up the protected-mode data segment registers
    movw $PROT_MODE_DSEG, %ax                       # Our data segment selector
    movw %ax, %ds                                   # -> DS: Data Segment
    movw %ax, %es                                   # -> ES: Extra Segment
    movw %ax, %fs                                   # -> FS
    movw %ax, %gs                                   # -> GS
    movw %ax, %ss                                   # -> SS: Stack Segment

    # Set up the stack pointer and call into C. The stack region is from 0--start(0x7c00)
    movl $0x0, %ebp
    movl $start, %esp

最后进入bootmain的C函数:
    
    call bootmain

##练习4

**题目**：bootloader如果读取硬盘扇区？如果加在ELF格式的OS？

**答案**：

通过readsect函数，从某扇区中读取数据到dst位置，再用readseg函数，可以读取任意长度的内容到虚拟地址中。  

在bootmain函数中，先读入elf的header部分，校验，根据ph数组依次读取出每一段程序，最后跳转到entry处进入内核的执行部分。


##练习5

**答案**：

实现见源代码。

最后一行输出为

    ebp:0x00007bf8 eip:0x00007d6e args:0xc032fcfa 0xc031fcfa 0x64e4d08e 0xfa7502a8
    <unknow>: -- 0x00007d6d --

进入bootmain时，call指令在0x00007d6e， 栈底在0x00007bf8。

##练习6

**问题1**：中断向量表中一个表项占多少字节？其中哪几位代表中断处理代码的入口？

**答案1**：
8个字节，2-3表示段选择子，0-1和6-7表示段内唯一，整体表示中断处理程序的入口地址。

**问题2**：请编程完善kern/trap/trap.c中对中断向量表进行初始化的函数idt_init。

**答案2**:
修改了idt_init()函数。根据xuetangx视频中老师的代码修改了源代码。初始化了中断表。

**问题3**：请编程完善trap.c中的中断处理函数trap，在对时钟中断进行处理的部分填写trap函数。

**答案3**：
修改了trap_dispatch()函数。在等一个case中维护全局变量++并在每100个tick调用print_ticks（）

