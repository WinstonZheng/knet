[Back](/language)
---

# Basic

![](/images/java/java-lang.jpg)

- 对java平台的认识？

本人将java平台认知分为基础和扩展两个部分（类似于se和ee）。

在基础部分，主要基于java语言本身，java是一门面向对象的语言，具备良好的可移植性(Write once, run anywhere.)，其提供了一系列基础的类库(io/nio/net/util/lang/math等)，能满足多线程并发（java只提供多线程）、文件读写、网络通信等多种任务需求。同时，java运行于jvm之上，jvm提供了自动的垃圾收集、类加载等功能，一方面通过字节码屏蔽底层，提高了java语言的跨平台特性，另一方面，减少使用者申请、释放内存的操作，更易操作。

在扩展方面，基于java语言，开发出许多框架与组件，应用广泛。其中Spring的IOC/AOP特性，ORM框架Mybatis，能够解决网站开发，数据库交互等多方面业务需求，而一些中间件，例如：消息中间件RocketMQ，可以应用于系统模块通信，实现模块解耦。

从中能看出java语言的广泛应用与强大的潜力。


- java是解释执行的吗？

java首先通过javac将java源码编译成字节码(byetcode)，然后，在运行时，通过Java虚拟机(JVM)内嵌的解释器将字节码转换成最终的机器码。此外，常用的Hotspot JVM提供了JIT(Just in time)编译器，JIT能够在运行时间热点代码编译成机器码，属于编译执行。

java主要分为编译期和运行期，编译期通过javac将java源码编译成字节码(.class文件)；运行期，JVM通过类加载器(Class-Loader)加载字节码，解释或编译执行。

JDK8中实际是解释和编译混合的一种模式，所谓混合模式(-Xmixed)，Oracle Hotspot JVM内置两个不同的JIT complier，C1对应client模式，适用于对启动速度敏感的应用(桌面应用)；C2对应server模式，针对长时间运行的服务端。


# Content




> **注意**：本目录结构，主要根据java包结构命名方式组织，例如：集合框架在java.util，则可以在javase/util下找到相关内容。

# Reference
- 《深入分析Java Web技术内幕》

