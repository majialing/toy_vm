# 写一个属于自己的虚拟机（Write your Own Virtual Machine）

> 前言：这是 Write your Own Virtual Machine 的第三篇文章。主要是来介绍基于 LC-3 Architecture 的 Assembly Examples。原文在[这里](https://justinmeiners.github.io/lc3-vm/)



## 汇编举例（Assembly Examples）



现在让我们看一个基于 LC-3 汇编程序，以了解 VM 实际运行的内容。你不需要知道如何写汇编代码或了解正在进行的所有操作。试着大致了解发生了什么。这是一个简单的“Hello World”：



```assembly
.ORIG x3000                        ; this is the address in memory where the program will be loaded
LEA R0, HELLO_STR                  ; load the address of the HELLO_STR string into R0
PUTs                               ; output the string pointed to by R0 to the console
HALT                               ; halt the program
HELLO_STR .STRINGZ "Hello World!"  ; store this string here in the program
.END                               ; mark the end of the file
```



就像在 C 中一样，程序从顶部开始并一次执行一个语句。但是，与 C 不同，没有嵌套的范围 {} 或控制结构，如 if 或 while。就是一些语句的陈列，这使执行变得容易。



请注意，某些语句的名称与我们之前定义的操作码相匹配。以前，我们了解到每条指令都是 16 位，但程序中每行看起来都是不同数量的字符。这种不一致可能吗？



这是因为我们正在阅读的代码是用汇编语言编写的，汇编是一种人类可读写的形式，以纯文本编码。**assembler** 用于将每行文本转换为 VM 可以理解的 16 位二进制指令。这种二进制形式本质上是一个16 位指令序列，称为机器代码，是 VM 实际运行的。

![assembler diagram](https://justinmeiners.github.io/lc3-vm/img/assembler.gif)

**Note**：尽管 **assembler** 和 **compiler** 在开发中起着类似的作用，但它们并不相同。**assembler** 简单地将程序员在文本中编写的内容编码为二进制文件，用其二进制替换符号并将其打包成指令。



命令 .ORIG 和 .STRINGZ 看起来像指令，但它们不是。它们是**assembler** 指令，用于生成一段代码或数据。例如，.STRINGZ 在编写的位置将一串字符插入到程序二进制文件中。



Loops 和 conditions 使用类似 goto 的指令完成。这是另一个计数到 10 的例子。



```assembly
AND R0, R0, 0                      ; clear R0
LOOP                               ; label at the top of our loop
ADD R0, R0, 1                      ; add 1 to R0 and store back in R0
ADD R1, R0, -10                    ; subtract 10 from R0 and store back in R1
BRn LOOP                           ; go back to LOOP if the result was negative
... ; R0 is now 10!
```



**Note**：本教程不需要学习编写汇编。但是，如果你有兴趣，可以使用 [LC-3 工具](http://highered.mheducation.com/sites/0072467509/student_view0/lc-3_simulator.html)编写和运行自己的 LC-3 程序。





















