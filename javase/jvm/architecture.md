# JVM体系结构
这系列的JVM知识都是基于概念模型（规范），而虚拟机具体的实现细节不同虚拟机也有差别。JVM实现的功能是运行Java代码并产生输出，而这个过程可以分为三步，
- 第一步，Javac将Java源码编译成字节码，存储为Class文件；
- 第二步，ClassLoader类加载器读取类的字节码文件，并将其加载到内存中并初始化；
- 第三步，执行引擎读取字节码运行，输出执行结果，执行过程中垃圾收集器运行自动回收内存；
- 第四步，程序执行结束，进行收尾工作，将类卸载并结束虚拟机进程；

JVM体系结构由5部分组成：
1. **Javac编译器**，将Java源文件编译成Class文件；
2. **类加载器**：在JVM启动时或者在类运行时将需要的class加载到JVM中；（每个被JVM装载的类型都有一个对应的java.lang.Class类的实例来表示该类型，用于唯一表示被JVM装载的class类）
3. **执行引擎**：执行引擎任务是负责执行class文件中包含的字节码指令，相当于实际机器上的CPU；（规范规定输入和输出，具体实现由不同厂家实现，如SUN的hotspot是基于栈的执行引擎；而google的dalvik是基于寄存器的执行引擎）,**执行引擎**是重复执行一条条代码，一个Java线程是一个执行引擎的实例，一个Java进程包含多个执行引擎在工作。
4. **内存区**：将内存划分为若干个区以模拟实际机器上的存储、记录和调度功能模块，如实际机器上的各种功能的寄存器或者Pc指针的记录器等（垃圾收集器）；
5. **本地方法调用**：调用C或C++实现本地方法的代码返回结果;

![](/images/application/jvm/jvm-arch.png)

> 指令的核心目的是确定需要运算的种类（操作码）和运算所需要的数据（操作数），以及从哪里（寄存器或栈）获取操作数，将运算结果存放到什么地方（寄存器或栈）等。正常编译器，是将高级语言翻译成机器指令（硬件可识别的）。

- **提问：为什么JVM选择基于栈的架构？** <br>
JVM有选择基于寄存器架构和栈架构两种，下面对两种架构进行简单比较：
    - 栈相对于寄存器的优点：栈的实现简单，不需要考虑临时变量存储；栈具备良好的可移植性，由于不同CPU架构具备不同数量的物理寄存器，需要将虚拟的寄存器进行映射，如果数量不符，实现复杂且影响效率；
    - 栈相对于寄存器的缺点：实现相同的操作，栈操作步骤更多，更加复杂。

## 执行引擎的架构设计
1. 每创建一个线程，创建一个Java栈；
2. 每创建一个线程，分配一个PC寄存器，指向第一行可执行代码；
3. 线程中每调用一个方法，栈上会创建一个新的栈帧数据结构，保存方法的局部变量。（栈帧，保留方法的元数据，局部变量，支持常量池解析，正常方法返回，异常处理机制等）

![](/images/application/jvm/jvm-exec.png)

简单描述一下执行引擎执行方法的过程：

![](/images/application/jvm/jvm-exec2.png)

在程序执行前，将程序转化为字节码指令。进入方法之后，在java栈中新建一个栈帧，栈帧主要包括内容如上图。
    - PC寄存器，用与存储当前执行位置；
    - 局部变量表，存储方法局部变量的值，局部变量所需的内存空间在编译期完成分配（执行期间不改变局部变量表大小）；
    - 操作栈，存储具体操作的过程。<br>
方法执行完成以后，局部变量区释放，PC寄存器销毁，JAVA栈中对应的栈帧销毁。

## JVM方法调用栈
JVM方法调用分为两种：
    1. Java方法调用；
    2. 本地方法调用；
当java中在一个方法中调用方法，会清空Pc寄存器，指向方法起始的位置，然后，新建一个新的栈帧数据结构，并将方法参数存储在局部变量区中（新的栈帧包含局部变量区和操作栈）。


