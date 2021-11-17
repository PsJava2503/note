# 操作系统的启动(bootsect、setup、head、main)

![image-20211117162659766](C:\Users\psj\AppData\Roaming\Typora\typora-user-images\image-20211117162659766.png)

​		开机的一瞬间CS和IP的值是多少这个问题是由硬件决定的，对于X86的PC机来说，开机时， CS=0xFFFF； IP=0x0000；因此地址为0xFFFF0（ROM BIOS映射区），刚开机时，内存里面只有这个地方有数据，这部分程序的功能是：检查RAM、键盘、显示器等硬件（这部分数据是固化在内存里面的，只能读不能写）。然后将磁盘0磁道0扇区（一个扇区是512个字节）内容读入0x7c00处，磁盘的0磁道0扇区存储的就是操作系统的引导程序（即bootsect.s）；同时设置CS=0x07c0,ip=0x0000，即开始执行操作系统。

菜的汇编略。

##### bootsect.s小结

bootsect.s是操作系统的最开始部分，共512个字节，在磁盘的0磁道0扇区位置，首先内存里面肯定是有代码的，具体存在哪个位置是由硬件决定的，然后从那个位置开始读入操作系统，首先读入的是操作系统的bootsect部分，对于x86PC来说，bootsect读进来是放在0x07c00这个位置，然后将其转移到0x90000这个位置，并继续执行；利用int 0x13中断，将操作系统的setup读入到0x90200开始的内存处，setup在磁盘上是第二到第五个扇区，第一个是bootsect扇区；读入setup之后，bootsect继续执行，在屏幕上显示开机logo “loading system…”。然后进入 read_it 继续读操作系统模块，然后将控制权转移到setup中，执行setup中的内容。

##### setup小结

setup是完成系统启动前设置的，它将硬件的参数存放在0x90000处，然后将system部分移动到从地址0开始的位置；临时建立gdt、idt表，并且从实模式进入到了保护模式（16位到32位）

setup.s通过int中断获取硬件参数并放置在内存合适位置,为system模块运行做准备,并将system从内存0x10000移动到0x00000处(此时system中的模块中的逻辑内存地址就是真实的物理地址,而且BIOS中设置的int中断向量表就被覆盖了,int中断用不了了),设置GDT表,将cr0寄存器PE设为1,开启保护模式(指令集改变为32or64位的cpu指令集,寻址方式也改变),PG设为1,启动分页,并跳转(jmpi 0,8)到0x00000,操作系统正式启动!

##### head.s

system开始的第一个文件是head.s，存放在地址0处，因此setup结束后执行的就是head.s文件。
setup.s进入保护模式，head.s是进入保护模式之后的初始化。

##### main.c

main函数完成了各种硬件数据结构的初始化。永远不会退出，如果退出就死机了。

![image-20211117163446257](C:\Users\psj\AppData\Roaming\Typora\typora-user-images\image-20211117163446257.png)

# 总结

其实bootsect、setup、heads、main这些文件就做了两件事：
1，读入操作系统并移动到合适的位置，
2，初始化（为每一个硬件建立数据结构、并初始化）