title: Use GCC to build STM32 projects
tags: []
categories: []
date: 2019-02-01 15:31:00
author:
---
最近突然想建立一个利用makefile编译的STM32工程模板，记录一下辛酸的过程。。。

   其实Makefile虽然我不是很熟，但是写一个简单的makefile还是OK的，所以在makefile阶段并没有遇到什么奇怪的问题，注意好makefile的规则就排除了，比如有时候没注意看TAB之类的，然后就是文件的路径。
   
   在make的过程中出现以下报错信息。
   ```
/tmp/cc6kleCZ.s: Assembler messages:
/tmp/cc6kleCZ.s:488: Error: registers may not be the same -- `strexb r0,r0,[r1]'
/tmp/cc6kleCZ.s:512: Error: registers may not be the same -- `strexh r0,r0,[r1]'
makefile:60: recipe for target 'core/CM3/core_cm3.o' failed
make: *** [core/CM3/core_cm3.o] Error 1
   ```
   作为一个菜鸡，看到Assembler 之类的error是比较慌的，但是还是不能畏惧的，正面刚。打开了core_cm3.c，找到了与它相关的函数，google后发现github上有人解决了，于是看了一下分析，做出了修改。然后成功编译过这一步。
   ```
   uint32_t __STREXB(uint8_t value, uint8_t *addr)
{
   uint32_t result=0;

   //__ASM volatile ("strexb %0, %2, [%1]" : "=r" (result) : "r" (addr), "r" (value) );
   __ASM volatile ("strexb %0, %2, [%1]" : "=&r" (result) : "r" (addr), "r" (value) );

   return(result);

}
  ```
  
  ```
  uint32_t __STREXH(uint16_t value, uint16_t *addr)
{
   uint32_t result=0;

   //__ASM volatile ("strexh %0, %2, [%1]" : "=r" (result) : "r" (addr), "r" (value) );
   __ASM volatile ("strexh %0, %2, [%1]" : "=&r" (result) : "r" (addr), "r" (value) );

   return(result);

}
  ```
  然后还有一个问题就是GCC编译一个文件时出现
  ```
  error: expected '(' before 'void'
 __asm void MSR_MSP(u32 addr)
  ```
  之类的报错，这个打开源文件后发现其实是GCC在C语言内嵌汇编需要注意的一个地方，修改为如下即可。
  ```
  void MSR_MSP(u32 addr)
{
	__ASM volatile("MSR MSP, r0");
	__ASM volatile("BX r14");
    
}
  ```
  还有要注意的就是，在stm32 标准固件库内，CORE的startup中有多个文件夹，Keil使用的启动文件在arm文件夹内，这和使用GCC时是不一样的，GCC下应该使用gcc_ride7中的启动文件,TrueSTUDIO是官方的IDE软件，其使用的也是GCC所以也可以使用其中的。然后就是链接文件(.ld)，可以使用stm32标准固件库文件夹下/Project/STM32F10x_StdPeriph_Template/TrueSTUDIO/中的链接文件。
  
  然后成功make出了 hex bin elf等文件，download到stm32后也是正常的。