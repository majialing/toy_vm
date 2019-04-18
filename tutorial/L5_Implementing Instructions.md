# 写一个属于自己的虚拟机（Write your Own Virtual Machine）

> 前言：这是 Write your Own Virtual Machine 的第五篇文章。主要是来介绍指令是如何实现的。原文在[这里](https://justinmeiners.github.io/lc3-vm/)



## 指令的实现（Implementing Instructions）



现在的任务是完成上一篇文章中 opcode 对应的代码 。每个指令的详细规格包含在[项目文档](https://justinmeiners.github.io/lc3-vm/supplies/lc3-isa.pdf)中。你需要了解每条指令如何从其规范中工作并编写实现。这比听起来容易。我将在这里演示如何实现其中两个。其余的代码可以在下一节中找到。



+ **ADD**

ADD 指令采用两个数字，将它们相加，并将结果存储在寄存器中。有关它的规范，请参见第 526 页。每条 ADD 指令如下所示：

![Add Encoding](https://justinmeiners.github.io/lc3-vm/img/add_layout.gif)

编码显示两行，因为此指令有两种不同的“模式”。在我解释模式之前，让我们试着找出它们之间的相似之处。在这两行中，我们可以看到我们从 4 位 0001 开始。这是 OP_ADD 的操作码值。接下来的 3 位标记为 DR。这代表目的地寄存器（destination register）。目的寄存器是存储和的位置。接下来的 3 位是 SR1。这是要求和的第一个数字的寄存器。



所以我们知道我们想要存储结果的位置，我们知道要求和的第一个数字。我们需要的最后一点信息是要求和的第二个数字。此时，两行开始看起来不同。请注意，在第一行，第 5 位为 0，第二行为 1。该位指示是立即模式还是寄存器模式。在寄存器模式下，第二个数字与第一个数字一样存储在寄存器中。这标记为 SR2，包含在位 2-0 中。第 3 和第 4 位未使用。在汇编中，这将写成：



```assembly
ADD R2 R0 R1 ; add the contents of R0 to R1 and store in R2.
```



在立即模式中，不是添加寄存器的内容，而是将值嵌入到指令本身中。这很方便，因为程序不需要更多指令将该数字从内存加载到寄存器中。相反，当我们需要它时，它就在指令内部。它只能存储一个小数字 $2^{5}$ = 32 （无符号）。这对于递增非常有用。在汇编中，它可以写成：



```assembly
ADD R0 R0 1 ; add 1 to R0 and store back in R0
```



以下摘自[项目文档](https://justinmeiners.github.io/lc3-vm/supplies/lc3-isa.pdf)：

> If bit [5] is 0, the second source operand is obtained from SR2. If bit [5] is 1, the second source operand is obtained by **sign-extending** the imm5 field to 16 bits. In both cases, the second source operand is added to the contents of SR1 and the result stored in DR. (Pg. 526)



这看起来就和我们讨论的一模一样。但是什么是 **sign-extending**？虽然立即模式值只有 5 位，但需要将其扩展到 16 位数。这 5 位需要扩展到 16 以匹配另一个数字。对于正数，我们可以为附加位填充0，并且值相同。但是，对于负数，这会导致问题。例如，以 5 位存储的 -1 是 11111。如果我们只是将其扩展为 0，则为 0000 0000 0001 1111，等于 32！符号扩展通过填充 0 表示正数而 1 表示负数来防止此问题。



```C
// 符号扩展算法
uint16_t sign_extend(uint16_t x, int bit_count)
{
    if ((x >> (bit_count - 1)) & 1) {
        x |= (0xFFFF << bit_count);
    }
    return x;
}
```



**Note**：如果你对二进制中的负数如何表示感兴趣，可以阅读 [Two's Complement](https://en.wikipedia.org/wiki/Two%27s_complement)。但是，这不是必需的。你可以复制上面的代码，并在需要的时候使用它。



说明书中有最后一句话：



> 根据结果是负，零还是正，设置条件代码。 （第 526 页）（The condition codes are set, based on whether the result is negative, zero, or positive. (Pg. 526)）



之前我们定义了一个条件标志 enum，现在是时候使用它们了。每次将值写入寄存器时，我们都需要更新标志以指示其符号。我们将编写一个函数，以便可以重用它：



```c
void update_flags(uint16_t r)
{
    if (reg[r] == 0)
    {
        reg[R_COND] = FL_ZRO;
    }
    else if (reg[r] >> 15) /* a 1 in the left-most bit indicates negative */
    {
        reg[R_COND] = FL_NEG;
    }
    else
    {
        reg[R_COND] = FL_POS;
    }
}
```



现在我们准备为 ADD 编写代码了：

```c
{
    /* destination register (DR) */
    uint16_t r0 = (instr >> 9) & 0x7;
    /* first operand (SR1) */
    uint16_t r1 = (instr >> 6) & 0x7;
    /* whether we are in immediate mode */
    uint16_t imm_flag = (instr >> 5) & 0x1;

    if (imm_flag)
    {
        uint16_t imm5 = sign_extend(instr & 0x1F, 5);
        reg[r0] = reg[r1] + imm5;
    }
    else
    {
        uint16_t r2 = instr & 0x7;
        reg[r0] = reg[r1] + reg[r2];
    }

    update_flags(r0);
}
```



这一节包含了很多信息，让我们来总结一下：

+ ADD 采用两个值并将它们存储在寄存器中
+ 在寄存器模式中，要添加的第二个值在寄存器中找到
+ 在立即模式下，第二个值嵌入在指令的最右边 5 位中

+ 短于 16 位的值需要进行符号扩展
+ 只要指令修改寄存器，就需要更新条件标志



你可能会对写 15 条指令感到不知所措。但是，你在这里学到的所有内容都将被重用。大多数指令使用符号扩展，不同模式和更新标志的某种组合。



### LDI

LDI 代表“间接负载”。该指令用于将值从存储器中的位置加载到寄存器中。规范见第 532 页。

这是二进制代码的样子

![LDI Encoding](https://justinmeiners.github.io/lc3-vm/img/ldi_layout.gif)



与 ADD 相比，没有不同的模式而且参数更少。这次，操作码是1010，它对应于 OP_LDI 枚举值。与 ADD 一样，它包含一个 3 位DR（目标寄存器），用于存储加载的值。其余位标记为 PCoffset9。这是嵌入指令的直接值（类似于 imm5 ）。由于该指令从内存加载，我们可以猜测这个数字是某种地址告诉我们从哪里加载。说明书提供了更多细节：



> 通过将位 [8：0] 符号扩展到 16 位并将该值加上递增的 PC 来计算地址。存储在该地址中的地址是要加载到 DR 中的数据的地址。 （第532页）（An address is computed by sign-extending bits `[8:0]` to 16 bits and adding this value to the incremented `PC`. What is stored in memory at this address is the address of the data to be loaded into `DR`. (Pg. 532)）



就像以前一样，我们需要有符号扩展这个 9 位值，但这次将它与当前的 PC 相加中。 （如果回头看执行循环，PC 在加载此指令后立即递增。）结果总和是内存中某个位置的地址，该地址包含另一个值。这个值是要加载的数据的地址。



这似乎是一种从内存中读取的迂回方式，但它是不可或缺的。 LD 指令仅限于 9 位的地址偏移，而存储器需要 16 位来寻址。 LDI 对于加载存储在远离当前 PC 的位置的值很有用，但是要使用它，最终位置的地址需要存储在附近的邻域中。你可以把它想象成在 C 中有一个局部变量，它是一个指向某些数据的指针：



```c
// the value of far_data is an address
// of course far_data itself (the location in memory containing the address) has an address
char* far_data = "apple";

// In memory it may be layed out like this:

// Address Label      Value
// 0x123:  far_data = 0x456
// ...
// 0x456:  string   = 'a'

// if PC was at 0x100
// LDI R0 0x023
// would load 'a' into R0
```



与之前相同，在将值放入 DR 后需要更新标志



以下是此 case 的代码：（mem_read将在后面的部分中讨论。）



```c
{
    /* destination register (DR) */
    uint16_t r0 = (instr >> 9) & 0x7;
    /* PCoffset 9*/
    uint16_t pc_offset = sign_extend(instr & 0x1ff, 9);
    /* add pc_offset to the current PC, look at that memory location to get the final address */
    reg[r0] = mem_read(mem_read(reg[R_PC] + pc_offset));
    update_flags(r0);
}
```



正如我所说，这条指令共享了很多从 ADD 学到的代码和知识。你会发现其余指令也是这种情况。



你现在需要回到之前实现其余的指令。遵循[说明书](https://justinmeiners.github.io/lc3-vm/supplies/lc3-isa.pdf)并使用之前列出的代码完成其他代码。所有指令的代码都列在本教程的末尾。之前指定的操作码中有两个操作码将不会被使用，它们是 OP_RTI 和OP_RES。你可以忽略这些情况。完成后，你的大部分 VM 将完成！









