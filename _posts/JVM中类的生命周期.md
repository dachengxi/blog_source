---
title: JVM中类的生命周期
date: 2017-08-26 19:53:29
categories: 
	- JVM
tags:
	- JVM
---

JVM中类的生命周期学习。

<!--more-->

加载、连接、初始化，对象实例化，垃圾收集，对象终结，类的卸载。

# 加载、连接、初始化

- 加载，就是把二进制形式的字节流读入JVM中
- 连接，就是把已经读入的二进制形式的类数据合并到JVM的运行时状态中去，连接又可分为验证、准备、解析三步：
  - 验证，确保二进制字节流数据格式正确并且适合于JVM使用
  - 准备，为类分配所需内存，为类变量分配内存
  - 解析，把常量池中的符号引用转换为直接引用，虚拟机的实现可以推迟解析这一步，等到运行中的程序真正使用某个符号的时候再去解析
- 初始化，就是给类变量赋合适的初值

虚拟机严格的规定了初始化的时机，所有JVM实现必须在每个类或者接口首次主动使用时初始化：

- 创建一个类的实例时（通过new指令创建、反射、克隆、反序列化）
- 调用一个类的静态方法时（invokespecial指令）
- 调用一个类或接口的静态字段，或者对该字段赋值时（getstatic指令或者putstatic指令），final修饰的静态字段除外
- 通过反射创建类的实例
- 初始化某个类的子类时，当前这个类需要先被初始化
- 虚拟机启动时被标明为启动类的类（还有main方法的那个启动类）

## 加载

加载分为三个步骤：

- 通过类的全限定名，产生一个代表类的二进制字节流
- 解析二进制字节流为方法区内部数据结构
- 创建一个表示该类的java.lang.Class类的实例对象

## 验证

确认类符合Java语言的语义，并且不会危及虚拟机的完整性。

## 准备

JVM为类变量分配内存，设置默认初始值。还可能为一些数据结构分配内存，用来提高程序运行性能，比如方法表，方法表包含指向每一个方法（包括从超类继承的方法）的指针。

## 解析

解析就是在常量池中寻找类、接口、方法的符号引用，将符号引用替换为直接引用。

class文件把所有的符号引用保存在常量池中。每个被JVM加载的类或接口都有一份内部版本的常量池，称为运行时常量池。当一个类被首次加载时，所有来自类的符号引用都加载到了类的运行时常量池。

程序运行时，如果某个符号引用将被使用，首先需要解析，解析过程就是根据符号引用查找到实体，再把符号引用替换成一个直接引用。这个过程也被称为常量池解析。

连接不仅仅包括把符号引用转换为直接引用，还包括检查正确性和权限。符号引用的存在性和访问权限是在解析的时候完成的。不管何时解析，错误只会在第一次实际使用符号引用的时候才会抛出来。

### 解析和动态扩展

可使用java.lang.Class的forName方法或者自定义类加载器的loadClass方法来动态扩展Java程序。

- forName方法有个initialize参数，如果为true，表示会进行连接并初始化；如果为false，表示类会被加载，可能会被连接，但是不会被明确的初始化。如果类已经被初始化，再使用forName方法并且initialize为false，返回的类也已经被初始化了。
- loadClass方法的resolve参数表示是否在加载类的时候执行连接，resolve为true时会保证方法返回Class实例之前就已经装载并连接。Java1.1后resolve就没有作用了。resolve为false时，返回的Class实例可能已经被连接也可能没有被连接，只是试图加载类并返回，而把连接和初始化类的时机留给虚拟机去决定。

## 初始化

初始化，就是为类变量赋予正确的初始值，也就是代码中指定的值，是通过类变量初始化语句或者静态初始化语句给出的。

所有类变量初始化语句和静态初始化语句都被Java编译器收集在一起，放到一个特殊的方法`<clinit>`，该方法只能被JVM调用。

初始化一个类包含两个步骤：

- 如果类存在直接父类，且父类还没有被初始化，就先初始化父类
- 如果类存在一个类初始化方法，先执行此方法

JVM需要确保初始化过程被正确的同步，多个线程同时初始化一个类，仅仅允许一个线程来执行初始化，其他线程需要等待。

一个类被加载、连接、初始化后，就可以被使用了，程序可以访问他的静态字段，调用静态方法或者创建他的实例。

# 对象的生命周期

## 类实例化

类可以明确或隐含的实例化，实例化类有四个途径：

- 使用new操作符实例化一个对象
- 使用Class或者`java.lang.reflect.Constructor`的newInstance方法
- 调用现有对象的clone方法
- 通过`java.io.ObjectInputStream`的getObject方法发序列化

还有几种情况下对象会被隐含的实例化：

- 保存命令行参数的String对象
- JVM加载的类，会实例化一个Class对象来表示这个类。
- JVM加载了在常量池中包含CONSTANT_String_info入口类的时候，会创建新的String对象的实例来表示这些常量字符串。
- 执行包含字符串连接操作的表达式会产生对象。

JVM创建类的实例时，会先在堆中为保存对象的实例变量分配内存，所有在对象的类中和父类中声明的变量，都要分配内存。准备好了堆内存后，就会把实例变量初始化为默认的初始值。随后会为实例变量赋正确的初始值。

- 如果是通过clone来创建对象，JVM把原来的实例变量中的值拷贝到新对象中
- 如果是反序列化，通过输入流中读取的值来初始化实例变量
- 虚拟机调用对象的实例初始化方法来初始化变量

编译器会为每个类至少生成一个实例初始化方法`<init>`，针对源码中每个类的构造方法，编译器都产生一个`<init>`方法，如果类没有明确的声明任何构造方法，编译器会产生一个无参构造方法，它仅仅调用父类的无参构造方法。

## 垃圾收集和对象的终结

对象不被使用了之后，垃圾收集器会回收垃圾对象。

## 类卸载

启动类加载器加载的类永远是可触达的，永远不会被卸载。使用用户自定义的类加载器加载的类才可能变成不可触达。

- 该类所有的实例都被回收，堆中不存在该类以及任何派生子类的实例
- 加载该类的类加载器已经被回收
- 对应的`java.lang.Class`对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法

# 参考

- 《深入Java虚拟机》
- 《实战Java虚拟机 JVM故障诊断与性能优化》
- 《HotSpot实战》
- 《深入理解Java虚拟机：JVM高级特性与最佳实践》