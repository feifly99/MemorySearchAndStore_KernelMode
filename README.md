2024/9/19深夜补档：
           两个问题：
           1.内存大量泄露.
           RSL链表某个节点断链时需要释放掉原来这个节点的buffer成员的内存；
           并且在不断链时要释放掉原来的内存并重新分配新字符串长度的内存；
           之前一个版本忘记了，导致了内存池为DDDD（RSL结构的buffer成员标号）的内存出现了大量泄漏，现在已经解决。
           2.蓝屏代码：INVALID_KERNEL_HANDLE.
           这个问题不是由于驱动程序引起的；
           而是在用户层调用驱动程序的时候多次调用____$_UNLOAD_DRIVER_PREPARE_$____；
           导致多次重复执行了ZwClose(g_kernelProcess)过程导致系统蓝屏，现在已经解决。
2024/9/19：对链表重新进行了整理，之前的版本是新建一个链表并释放原来的链表；
           新版本是直接对老链表进行节点移除，定义了虚拟链表头来辅助移除节点；
           改完之后对于大型应用程序的搜索效率更加高效。
2024/9/18：再一次重构了IOCTL逻辑，重新定义了RSL结构体来进行对比搜索；
           目前驱动的大多数基本功能都已经实现，包括
           内存遍历，内存查找，内存更改，进程隐藏，模块遍历，模块隐藏，寄存器读取；
           内存查找模式支持首次搜索，精确值搜索，变大的数值，变小的数值，未改变的数值；
           数值变动基于远端字节判断方法；
           在OS:Windows x64 22H2/CPU:I7-9750H架构下驱动平稳运行，不会蓝屏，所有的内存块都没有内存泄漏；
           不会蓝屏，不会内存泄漏，没有死锁/死循环风险；
           几乎对所有可能蓝屏/内存泄漏/死循环的地方都进行了严格的条件检查；
           接下来的工作：整理并优化驱动代码结构/构建用户层应用程序进行交互；
           其他开发中的功能：抹PE头，断链，PID篡改，PE特征修改。
2024/9/16：重构了IOCTL的屎山！逻辑更清晰，支持多次查找和继续查找；
           删除了大量的宏定义和typedef,增加可读性；
           六块分页内存没有一丁点泄露，所有内存和句柄都被正确释放，基于poolmon.exe检测得出；
           但首次引用了全局变量，正在准备开往第二座屎山。。。。
2024/9/12：IOCTL驱动处理已经快写成屎山了。还出现了一个内存泄漏，但是那个内存泄漏已经解决了；
           泄露原因在于释放单链表操作最后一个节点没释放；
           Driver_User_IO_Interaction_Entry早晚得重构，不能在屎山上一直拉屎。
2024/9/11：新增了内核层和用户层的交互，支持用户层传递结构体到驱动；
           目前支持内存遍历，字符串搜索，遍历进程的全部线程和全部模块，写入内存。
2024/9/10：修改了模块遍历，之前由于_LDR_DATA_TABLE_ENTRY结构和_PEB_LDR_DATA弄混了导致偏移量错误导致了不能正确地导出模块地址；
           新增了强行写内存，基于修改控制寄存器CR0实现；
           知道了KdBreakPoint怎么和Windbg交互了。
2024/9/9：修复了关机蓝屏的BUG，正确地释放了引用计数。
2024/9/8：新增进程模块遍历，基于PEB->LDR->InLoadOrderModuleListAddress硬编码指针实现。
2024/9/7：新增进程隐藏，基于断PE链实现。
2024/9/3：驱动正常运行，但是一关机就会蓝屏，蓝屏代码为REFERENCE_BY_POINTER。
