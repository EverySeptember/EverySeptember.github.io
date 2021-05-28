---
title: Java核心技术-进阶
date: 2019-3-30 20:45:11
tags: Java
---

### 设计模式

- **创建型模式**：是对对象创建过程的各种问题和解决方法的总结，包括工厂模式（Factory、Abstract Factory）、单例模式（Singleton）、构造器模式（Builder）、原型模式（ProtoType）。
- **结构型模式**：是针对软件设计的总结，关注于类、对象继承、组合方式的实践经验。常见的结构型模式，包括桥接模式（Bridge）、适配器模式（Adapter）、装饰器模式（Decorator）、代理模式（Proxy）、组合模式（Composite）、外观模式（Facade）、享元模式（Flyweight）等。
- **行为模式**：是从类或对象之间交互、职责划分等角度总结的模式。比较常见的行为型模式有策略模式（Strategy）、解释器模式（Interpreter）、命令模式（Commond）、观察者模式（Observer）、迭代器模式（Iterator）、模板方法模式（Template Method）、访问者模式（Visitor）。

### synchronized和ReentrantLock

- synchronized是Java内建的同步机制，提供了互斥的语义和可见性，当一个线程已经获取当前锁时，其他视图获取的线程只能等待或者阻塞在那里。本质上synchronized方法等同于把方法全部语句用synchronized块包起来。
- ReentrantLock，通常翻译为再入锁，是Java5提供的锁实现，它的语义和synchronized基本相同。再入锁通过代码直接调用lock()方法获取，代码书写也更加灵活。ReentrantLock也提供了很多方法，可以做到许多synchronized无法做到的细节控制。ReentrantLock需要在使用完成后明确调用unlock()方法释放，不然就会一直持有该锁。

##### 线程安全

线程安全是一个多线程环境下正确性的概念，也就是保证多线程环境下**共享的、可修改**的状态的正确性。

线程安全需要保证几个基本特性：

- 原子性：相关操作中不会中途被其他线程干扰，一般通过同步机制实现。
- 可见性：一个线程修改了某个共享变量，其状态能够立即被其他线程知晓，通常被解释为将线程本地状态反映到内存上，volatile就是负责保证可见性的。
- 有序性：保证线程内串行语义，避免指令重排等。

