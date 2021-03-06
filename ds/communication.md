# 通信
本章讨论主要围绕五个通信模型：
- RPC(remote procedure call)，远程过程调用；
- RMI(remote method invocationi)，远程方法调用；
- MOM(message-oriented middleware)，面向消息中间件；
- Stream，流；
- 多播 <br>

首先，我们在介绍模型前提出三个问题：
1. 为什么出现这种模型？模型解决的核心问题情景是什么？
2. 模型是如何解决这种问题的？
3. 这种模型有什么优缺点？

## 基础知识
- OSI模型，协议栈。
- TCP/IP协议栈。
- 中间件协议（应用层）。

### 通信模型
- 持久通信，消息不会被丢弃，由通信中间件存储；
- 瞬时通信，传输中断/接收方不活动，消息即丢弃；
- 异步通信，发送方提交信息，立即执行；
- 同步通信，发送方阻塞等待请求接受；
    - 可能被阻塞，等待中间件通知请求传输完成；
    - 发送发为同步化，直到请求被发送到目标接收方；
    - 等到接收方返回响应，实现同步化。

## 远程过程调用