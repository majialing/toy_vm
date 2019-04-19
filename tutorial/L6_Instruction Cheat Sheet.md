# 写一个属于自己的虚拟机（Write your Own Virtual Machine）

> 前言：这是 Write your Own Virtual Machine 的第六篇文章。主要是实现上一篇文章中没有实现的指令。原文在[这里](https://justinmeiners.github.io/lc3-vm/)



## 指令备忘单（Instruction Cheat Sheet）



### RTI & RES

不使用这两条指令



```c
abort()
```



### 按位与（[Bitwise not](https://en.wikipedia.org/wiki/Bitwise_operation#NOT)）



```c
{
    uint16_t r0 = (instr >> 9) & 0x7;
    uint16_t r1 = (instr >> 6) & 0x7;
    uint16_t imm_flag = (instr >> 5) & 0x1;

    if (imm_flag)
    {
        uint16_t imm5 = sign_extend(instr & 0x1F, 5);
        reg[r0] = reg[r1] & imm5;
    }
    else
    {
        uint16_t r2 = instr & 0x7;
        reg[r0] = reg[r1] & reg[r2];
    }
    update_flags(r0);
}
```



### 按位取反（[Bitwise not](https://en.wikipedia.org/wiki/Bitwise_operation#NOT)）



```c
{
    uint16_t r0 = (instr >> 9) & 0x7;
    uint16_t r1 = (instr >> 6) & 0x7;

    reg[r0] = ~reg[r1];
    update_flags(r0);
}
```



### 分支 （Branch）

```c
{
    uint16_t pc_offset = sign_extend((instr) & 0x1ff, 9);
    uint16_t cond_flag = (instr >> 9) & 0x7;
    if (cond_flag & reg[R_COND])
    {
        reg[R_PC] += pc_offset;
    }
}
```



### 跳转（Jump）

RET 在说明书中作为单独的指令列出，因为它在汇编中是不同的关键字。但是，它实际上是 JMP 的一个特例。当 R1 为 7 时，RET 就会发生。



```c
{
    /* Also handles RET */
    uint16_t r1 = (instr >> 6) & 0x7;
    reg[R_PC] = reg[r1];
}
```



### 寄存器跳转（Jump Register）

```c
{
    uint16_t r1 = (instr >> 6) & 0x7;
    uint16_t long_pc_offset = sign_extend(instr & 0x7ff, 11);
    uint16_t long_flag = (instr >> 11) & 1;

    reg[R_R7] = reg[R_PC];
    if (long_flag)
    {
        reg[R_PC] += long_pc_offset;  /* JSR */
    }
    else
    {
        reg[R_PC] = reg[r1]; /* JSRR */
    }
    break;
}
```



### 加载（Load）

```c
{
    uint16_t r0 = (instr >> 9) & 0x7;
    uint16_t pc_offset = sign_extend(instr & 0x1ff, 9);
    reg[r0] = mem_read(reg[R_PC] + pc_offset);
    update_flags(r0);
}
```



### 加载寄存器（Load Register）

```c
{
    uint16_t r0 = (instr >> 9) & 0x7;
    uint16_t r1 = (instr >> 6) & 0x7;
    uint16_t offset = sign_extend(instr & 0x3F, 6);
    reg[r0] = mem_read(reg[r1] + offset);
    update_flags(r0);
}
```



### 加载有效地址（Load Effective Address）



```c
{
    uint16_t r0 = (instr >> 9) & 0x7;
    uint16_t pc_offset = sign_extend(instr & 0x1ff, 9);
    reg[r0] = reg[R_PC] + pc_offset;
    update_flags(r0);
}
```



### 存储（store）

```c
{
    uint16_t r0 = (instr >> 9) & 0x7;
    uint16_t pc_offset = sign_extend(instr & 0x1ff, 9);
    mem_write(reg[R_PC] + pc_offset, reg[r0]);
}
```



### 间接存储（Store Indirect）

```c
{
    uint16_t r0 = (instr >> 9) & 0x7;
    uint16_t pc_offset = sign_extend(instr & 0x1ff, 9);
    mem_write(mem_read(reg[R_PC] + pc_offset), reg[r0]);
}
```



### 寄存器存储（Store Register）

```c
{
    uint16_t r0 = (instr >> 9) & 0x7;
    uint16_t r1 = (instr >> 6) & 0x7;
    uint16_t offset = sign_extend(instr & 0x3F, 6);
    mem_write(reg[r1] + offset, reg[r0]);
}
```
