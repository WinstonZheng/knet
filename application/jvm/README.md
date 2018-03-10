# JVM概述
JVM全称是Java Virtual Machine，将高级语言的代码翻译成java字节码这种中间形式，实现跨平台的能力。
JVM主要模拟计算机的计算能力，具体能力如下：
    1. 指令集，用来计算和控制计算机系统的一套指令集合（直接用于机器识别，二进制存储，汇编语言是指令集合的助记符）；
    2. 计算单元，识别并控制指令执行的功能模块；计算单元，识别并控制指令执行的功能模块；
    3. 寄存器定义，包括操作数寄存器、变址寄存器、控制寄存器等定义、数量和使用方式；
    4. 存储单元，能够存储操作数和保存操作结构的单元，如内核级缓存、内存和磁盘等。

> CPU架构，包含寄存器、段等芯片操作。
- 和指令集关系，不同架构对应不同的指令集（采用兼容方式支持不同指令集RISC\CISC）；
- 和汇编语言的关系，一条指令集对应一个汇编语言。

# JVM结构体系