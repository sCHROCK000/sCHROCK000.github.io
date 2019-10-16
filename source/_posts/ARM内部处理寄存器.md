title: ARM处理器内部寄存器
date: 2019-10-14 07:26:11
tags:
author:
---

ARM处理器内部寄存器作用
===

![](/images/arm1.png)


***

r0-r15 and R0-R15  
a1-a4 (argument, result, or scratch registers, synonyms for r0 to r3)  
v1-v8 (variable registers, r4 to r11)  
sb and SB (static base, r9)  
ip and IP (intra-procedure-call scratch register, r12)  
sp and SP (stack pointer, r13)  
lr and LR (link register, r14)  
pc and PC (program counter, r15)

***

1. r0-r3用作传入函数参数，传出函数返回值。在子程序调用之间，可以将 r0-r3 用于任何用途。被调用函数在返回之前不必恢复 r0-r3。如果调用函数需要再次使用 r0-r3 的内容，则它必须保留这些内容。  
2. r4-r11被用来存放函数的局部变量。如果被调用函数使用了这些寄存器，它在返回之前必须恢复这些寄存器的值。  
3. r12是内部调用暂时寄存器 ip。它在过程链接胶合代码（例如，交互操作胶合代码）中用于此角色。在过程调用之间，可以将它用于任何用途。被调用函数在返回之前不必恢复 r12。  
4. r13是栈指针 sp。它不能用于任何其它用途。sp 中存放的值在退出被调用函数时必须与进入时的值相同。  
5. r14是链接寄存器 lr。如果您保存了返回地址，则可以在调用之间将 r14 用于其它用途，程序返回时要恢复。  
6. r15是程序计数器 PC。它不能用于任何其它用途。

###### 注意：在中断程序中，所有的寄存器都必须保护，编译器会自动保护R4～R11

原文出处：  
https://blog.csdn.net/wh8_2011/article/details/53195320  
https://blog.csdn.net/Decisiveness/article/details/44106197