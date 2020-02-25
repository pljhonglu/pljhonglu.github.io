# 栈

向下增长。栈底在高地址。

- R0-R3:用于函数参数及返回值的传递
- R4-R6, R8, R10-R11:没有特殊规定，就是普通的通用寄存器
- R7:栈帧指针(Frame Pointer)。指向前一个保存的栈帧(stack frame)和链接寄存器(link register， lr)在栈上的地址。即指向当前函数的栈底，就是与上一个函数的交界
- R9:操作系统保留
- R12:又叫IP(intra-procedure scratch ), 要说清楚要费点笔墨，参见http://blog.csdn.net/gooogleman/article/details/3529413
- R13:又叫SP(stack pointer)，是栈顶指针
- R14:又叫LR(link register)，存放函数的返回地址（也就是执行完子函数的下一条指令地址）。
- R15:又叫PC(program counter)，指向当前指令地址。
- CPSR:当前程序状态寄存器(Current Program State Register)，在用户状态下存放像condition标志中断禁用等标志的。在其它系统状态中断状等状态下与CPSR对应还有一个SPSR，在这里不详述了。

另外还有VFP(向量浮点运算)相关的寄存器，在此我们略过，感兴趣的可以从后面的参考链接去查看。


[iOS 逆向之ARM汇编](http://www.cnblogs.com/csutanyu/p/3575297.html)
[ARM的栈帧](http://www.cnblogs.com/chyl411/p/4579053.html)
[ARM汇编一些基础知识](https://09jianfeng.github.io/2015/05/07/ARM%E6%B1%87%E7%BC%96%E4%B8%80%E4%BA%9B%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/)


