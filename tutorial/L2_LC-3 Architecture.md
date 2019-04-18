# 写一个属于自己的虚拟机（Write your Own Virtual Machine）

> 前言：这是 Write your Own Virtual Machine 的第二篇文章。主要是来介绍 LC-3 Architecture。原文在[这里](https://justinmeiners.github.io/lc3-vm/)



## LC-3 架构（LC-3 Architecture）

我们的 VM 会要模拟一个功能性的计算机 — [LC-3](https://en.wikipedia.org/wiki/LC-3)。LC-3非常适合教大学生如何用汇编语言编程。与 [x86](http://ref.x86asm.net/coder64.html) 相比，它具有简化的指令集，但包含现代 CPU 中使用的所有主要思想。



首先，我们需要模拟机器的基本硬件组件。尝试了解每个组件是什么。首先创建一个 C 文件。本节中的每个代码段都应放在此文件的全局范围内。



### 内存（Memory）



LC-3 有 65,536 个内存地址（最大的是 16 位无符号整数 $2^{16}$），每个内存地址存储一个 16 位的值。这意味着整个内存可以存储 128KB，这非常小。在我们的程序中，这个内存用一个简单的数组储存。



``` c
/* 65536 locations */
uint16_t memory[UINT16_MAX];
```



## 寄存器（Registers）



寄存器是用于在 CPU 上存储单个值的插槽。寄存器就像 CPU 的“工作台”。要使 CPU 处理一段数据，它必须位于其中一个寄存器中。但是，由于只有几个寄存器，因此在任何给定时间只能加载少量的数据。程序通过将内存中的值加载到寄存器中，将值计算到其他寄存器中，然后将最终结果存储回内存来解决此问题。



LC-3 共有10个寄存器，每个寄存器为 16 位。他们中的大多数是通用的，只有少数寄存器被指定了角色。



+ 8 个通用寄存器（R0 - R7）
+ 1 个程序计数器（PC）
+ 1 个条件标志（COND）寄存器



通用寄存器可用于执行任何程序计算。程序计数器是无符号整数，它是要执行的内存中下一条指令的地址。条件标志告诉我们有关先前计算的信息。



``` c
enum
{
    R_R0 = 0,
    R_R1,
    R_R2,
    R_R3,
    R_R4,
    R_R5,
    R_R6,
    R_R7,
    R_PC, /* program counter */
    R_COND,
    R_COUNT
};
```



就像内存一样，我们在数组中存储寄存器。

```c
uint16_t reg[R_COUNT];
```



### 指令集（Instruction Set）



指令是一个命令，它告诉 CPU 执行一些基本任务，例如两个数字相加。指令具有指示要执行的任务类型的操作码（opcode）和一组为正在执行的任务提供输入的参数（parameters）。



每个操作码（opcode）代表一个CPU“知道”如何去做的任务。 LC-3 中只有 16 个操作码。计算机可以计算的任何东西都是这些指令的组合。每条指令长 16 位，左边 4 位存储操作码。其余位用于存储参数。



稍后，我们将详细讨论每个指令会做什么。现在，定义以下操作码。确保它们保持此顺序，以便为它们分配正确的枚举值：



```c
enum
{
    OP_BR = 0, /* branch */
    OP_ADD,    /* add  */
    OP_LD,     /* load */
    OP_ST,     /* store */
    OP_JSR,    /* jump register */
    OP_AND,    /* bitwise and */
    OP_LDR,    /* load register */
    OP_STR,    /* store register */
    OP_RTI,    /* unused */
    OP_NOT,    /* bitwise not */
    OP_LDI,    /* load indirect */
    OP_STI,    /* store indirect */
    OP_JMP,    /* jump */
    OP_RES,    /* reserved (unused) */
    OP_LEA,    /* load effective address */
    OP_TRAP    /* execute trap */
};
```



**Note**：英特尔 x86 架构有数百条指令，而 ARM 和 LC-3 等其他指令则很少。小指令集称为 [RISC](https://en.wikipedia.org/wiki/Reduced_instruction_set_computer)，而较大指令集称为 [CISC](https://en.wikipedia.org/wiki/Complex_instruction_set_computer)。较大的指令集通常不提供任何在根本上新的指令，而是使写汇编的时候更加方便。CISC 中的单个指令可能取代 RISC 中的几个指令。然而，对于工程师来说，它们设计和制造往往更复杂和昂贵。



### 条件标志（Condition Flags）



R_COND 寄存器存储条件标志，提供最近执行计算的有关信息。这允许程序检查逻辑条件，例如 if(x > 0){...}。



每个 CPU 都有各种条件标志来指示各种情况。 LC-3 仅使用 3 个条件标志，表示先前计算的符号。



```c
enum
{
    FL_POS = 1 << 0, /* P */
    FL_ZRO = 1 << 1, /* Z */
    FL_NEG = 1 << 2, /* N */
};
```



**Note**：<< 符号称为左位移操作符。(n << k) 将  n 的所有位移位到左k 位。因此 1 << 2 将等于 4（1 -> 100 = 4）.如果你不熟悉，请阅读该[链接](https://msdn.microsoft.com/en-us/library/336xbhcz.aspx)。这很重要。



我们已经完成了 VM 硬件组件的设置！添加标准包含（参见参考资料）后，你的文件应如下所示：



```makefile
{Includes, 12}

{Registers, 3}
{Opcodes, 3}
{Condition Flags, 3}
```

