# 写一个属于自己的虚拟机（Write your Own Virtual Machine）

> 前言：这是 Write your Own Virtual Machine 的第四篇文章。主要是来介绍执行程序前的一些知识。原文在[这里](https://justinmeiners.github.io/lc3-vm/)



## 执行程序（Executing Programs）

再一次，前面的示例只是为了让你了解 VM 的功能。要编写 VM，你无需精通 assembly。只要你按照正确的步骤读取和执行指令，任何LC-3 程序都将正确运行，无论它有多复杂。从理论上讲，它甚至可以运行 Web 浏览器或 Linux 操作系统！



如果你深入思考这个属性，这是一个哲学上非凡的想法。程序本身可以做各种我们从未预料到并且可能无法理解的智能的事情，但与此同时，他们所能做的一切仅限于我们编写的简单代码！我们同时了解也不了解每个程序的工作原理。图灵讨论了这个奇妙的想法：



> "The view that machines cannot give rise to surprises is due, I believe, to a fallacy to which philosophers and mathematicians are particularly subject. This is the assumption that as soon as a fact is presented to a mind all consequences of that fact spring into the mind simultaneously with it. It is a very useful assumption under many circumstances, but one too easily forgets that it is false." — [Alan M. Turing](https://academic.oup.com/mind/article-pdf/LIX/236/433/9866119/433.pdf)



### 程序（Procedure）

以下是我们需要编写的过程的精确描述：



1. 从 PC 寄存器地址的存储器加载一条指令
2. PC 寄存器加一
3. 查看操作码以确定它应该执行哪种类型的指令
4. 使用指令中的参数执行指令
5. 返回第一步



你可能想知道，“如果循环不断增加PC，我们没有 if 或 while ，它会不会很快耗尽指令？”答案是不。正如我们之前提到的，一些类似goto 的指令通过跳转 PC 来改变执行流程。



让我们开始在主循环中概述这个过程：



```c
int main(int argc, const char* argv[])
{
    {Load Arguments, 12}
    {Setup, 12}

    /* set the PC to starting position */
    /* 0x3000 is the default */
    enum { PC_START = 0x3000 };
    reg[R_PC] = PC_START;

    int running = 1;
    while (running)
    {
        /* FETCH */
        uint16_t instr = mem_read(reg[R_PC]++);
        uint16_t op = instr >> 12;

        switch (op)
        {
            case OP_ADD:
                {ADD, 6}
                break;
            case OP_AND:
                {AND, 7}
                break;
            case OP_NOT:
                {NOT, 7}
                break;
            case OP_BR:
                {BR, 7}
                break;
            case OP_JMP:
                {JMP, 7}
                break;
            case OP_JSR:
                {JSR, 7}
                break;
            case OP_LD:
                {LD, 7}
                break;
            case OP_LDI:
                {LDI, 6}
                break;
            case OP_LDR:
                {LDR, 7}
                break;
            case OP_LEA:
                {LEA, 7}
                break;
            case OP_ST:
                {ST, 7}
                break;
            case OP_STI:
                {STI, 7}
                break;
            case OP_STR:
                {STR, 7}
                break;
            case OP_TRAP:
                {TRAP, 8}
                break;
            case OP_RES:
            case OP_RTI:
            default:
                {BAD OPCODE, 7}
                break;
        }
    }
    {Shutdown, 12}
}
```







