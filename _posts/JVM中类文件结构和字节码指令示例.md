---
title: JVM中类文件结构和字节码指令示例
date: 2017-08-20 20:31:37
categories: 
	- JVM
tags:
	- JVM
---

JVM中类文件结构和字节码指令示例。

<!--more-->

# class文件

```java
Classfile /xxxxx/me/cxis/dcc/loader/ConfigLoaderDelegate.class
  Last modified xxxx-x-x; size 1668 bytes
  MD5 checksum 26fb18f78a9fff6fccbe73d445ee8c58
  Compiled from "ConfigLoaderDelegate.java"
public class me.cxis.dcc.loader.ConfigLoaderDelegate
  // 次版本号
  minor version: 0

  // 主版本号
  major version: 52

  // 类的访问标志
  // ACC_PUBLIC：是否public类型
  // ACC_SUPER：是否允许使用invokespecial字节码指令的新语意，JDK1.0.2之后编译出来的类的这个标志都必须为真
  flags: ACC_PUBLIC, ACC_SUPER

// 常量池，存放字面量和符号引用
// 字面量类似Java中的常量，如文本字符串、final常量等
// 符号引用包括：类和接口的全限定名、字段的名称和描述符、方法的名称和描述符
Constant pool:
   // 方法的符号引用，指向第12和第42个常量
   // 最后结果是：java/lang/Object."<init>":()V
   // 调用父类Object的构造方法，该方法返回值是V表示void
   #1 = Methodref          #12.#42        // java/lang/Object."<init>":()V

   // 字段的符号引用，指向第7和第43常量
   // 最后结果是：me/cxis/dcc/loader/ConfigLoaderDelegate.configLoader:Lme/cxis/dcc/loader/ConfigLoader;
   // 对应源码的：private ConfigLoader configLoader;
   #2 = Fieldref           #7.#43         // me/cxis/dcc/loader/ConfigLoaderDelegate.configLoader:Lme/cxis/dcc/loader/ConfigLoader;

   // 类或接口的符号引用，指向第44个常量：me/cxis/dcc/loader/ZookeeperConfigLoader
   #3 = Class              #44            // me/cxis/dcc/loader/ZookeeperConfigLoader

   // 方法的符号引用，指向第3和第42个常量
   // 最后结果是：me/cxis/dcc/loader/ZookeeperConfigLoader."<init>":()V
   // 对应源码中无参构造方法
   #4 = Methodref          #3.#42         // me/cxis/dcc/loader/ZookeeperConfigLoader."<init>":()V

   // 方法的符号引用，指向第7和第45个常量
   // 最后结果是：me/cxis/dcc/loader/ConfigLoaderDelegate.getInstance:(Lme/cxis/dcc/loader/ConfigLoader;)Lme/cxis/dcc/loader/ConfigLoaderDelegate;
   // 对应源码中的带参数的静态方法：public static ConfigLoaderDelegate getInstance(ConfigLoader configLoader)
   // 方法参数：Lme/cxis/dcc/loader/ConfigLoader; L表示是对象类型
   // 方法返回值：Lme/cxis/dcc/loader/ConfigLoaderDelegate;
   #5 = Methodref          #7.#45         // me/cxis/dcc/loader/ConfigLoaderDelegate.getInstance:(Lme/cxis/dcc/loader/ConfigLoader;)Lme/cxis/dcc/loader/ConfigLoaderDelegate;

   // 字段的符号引用，指向第7和第46常量
   // 最后结果是：me/cxis/dcc/loader/ConfigLoaderDelegate.configLoaderDelegate:Lme/cxis/dcc/loader/ConfigLoaderDelegate;
   // 对应源码的：private static volatile ConfigLoaderDelegate configLoaderDelegate;
   #6 = Fieldref           #7.#46         // me/cxis/dcc/loader/ConfigLoaderDelegate.configLoaderDelegate:Lme/cxis/dcc/loader/ConfigLoaderDelegate;

   // 类或接口的符号引用，指向第47个常量：me/cxis/dcc/loader/ConfigLoaderDelegate
   #7 = Class              #47            // me/cxis/dcc/loader/ConfigLoaderDelegate

   // 方法的符号引用，指向第7和第48个常量
   // 最后结果是：me/cxis/dcc/loader/ConfigLoaderDelegate."<init>":(Lme/cxis/dcc/loader/ConfigLoader;)V
   // 对应源码中的有参构造方法：private ConfigLoaderDelegate(ConfigLoader configLoader)
   // 参数是：Lme/cxis/dcc/loader/ConfigLoader; 返回值是：V
   #8 = Methodref          #7.#48         // me/cxis/dcc/loader/ConfigLoaderDelegate."<init>":(Lme/cxis/dcc/loader/ConfigLoader;)V

   // 接口中方法的符号引用，指向第49和50个常量
   // 最后结果是：me/cxis/dcc/loader/ConfigLoader.get:(Ljava/lang/String;)Ljava/lang/String;
   // 表示的是ConfigLoader.get方法，参数是String类型，返回值是String类型
   #9 = InterfaceMethodref #49.#50        // me/cxis/dcc/loader/ConfigLoader.get:(Ljava/lang/String;)Ljava/lang/String;

  // 接口中方法的符号引用，指向第49和51个常量
  // 最后结果是：me/cxis/dcc/loader/ConfigLoader.addConfigListener:(Lme/cxis/dcc/listener/ConfigListener;)V
  // 表示的是ConfigLoader.addConfigListener方法，参数是ConfigListener类型，返回值是V
  #10 = InterfaceMethodref #49.#51        // me/cxis/dcc/loader/ConfigLoader.addConfigListener:(Lme/cxis/dcc/listener/ConfigListener;)V

  // 接口中方法的符号引用，指向第49和52个常量
  // 最后结果是：me/cxis/dcc/loader/ConfigLoader.addConfigListener:(Ljava/lang/String;Lme/cxis/dcc/listener/ConfigListener;)V
  // 表示的是ConfigLoader.addConfigListener方法，参数是String类型和ConfigListener类型，返回值是V
  #11 = InterfaceMethodref #49.#52        // me/cxis/dcc/loader/ConfigLoader.addConfigListener:(Ljava/lang/String;Lme/cxis/dcc/listener/ConfigListener;)V

  // 类或接口的符号引用，指向第53个常量：java/lang/Object
  #12 = Class              #53            // java/lang/Object

  // UTF-8编码的字符串，表示configLoader变量的名字
  #13 = Utf8               configLoader

  // UTF-8编码的字符串，ConfigLoader的描述符
  #14 = Utf8               Lme/cxis/dcc/loader/ConfigLoader;

  // UTF-8编码的字符串，configLoaderDelegate变量的名字
  #15 = Utf8               configLoaderDelegate

  // UTF-8编码的字符串，ConfigLoaderDelegate类描述符
  #16 = Utf8               Lme/cxis/dcc/loader/ConfigLoaderDelegate;

  // UTF-8编码的字符串，系统生成实例的初始化方法名字
  #17 = Utf8               <init>

  // UTF-8编码的字符串，方法的描述符常量，无参空返回
  #18 = Utf8               ()V

  // UTF-8编码的字符串，Code属性，用在方法表中，Java代码编译成的字节码指令
  #19 = Utf8               Code

  // UTF-8编码的字符串，LineNumberTable属性，用在Code属性中，Java源码的行号与字节码指令的对应关系
  #20 = Utf8               LineNumberTable

  // UTF-8编码的字符串，LocalVariableTable属性，用在Code属性中，方法的局部变量描述
  #21 = Utf8               LocalVariableTable

  // UTF-8编码的字符串，当前实例this
  #22 = Utf8               this

  // UTF-8编码的字符串，方法的描述符常量，参数为ConfigLoader返回值为V
  #23 = Utf8               (Lme/cxis/dcc/loader/ConfigLoader;)V

  // UTF-8编码的字符串，方法表，JDK8新增属性，用于支持将方法名编译进Class文件并可运行时获取
  #24 = Utf8               MethodParameters

  // UTF-8编码的字符串，方法名称常量
  #25 = Utf8               getInstance

  // UTF-8编码的字符串，方法的描述符常量，参数为空，返回值为ConfigLoaderDelegate
  #26 = Utf8               ()Lme/cxis/dcc/loader/ConfigLoaderDelegate;

  // UTF-8编码的字符串，方法的描述符常量，参数为ConfigLoader，返回值为ConfigLoaderDelegate
  #27 = Utf8               (Lme/cxis/dcc/loader/ConfigLoader;)Lme/cxis/dcc/loader/ConfigLoaderDelegate;

  // UTF-8编码的字符串，StackMapTable属性，用在Code属性中，JDK6新增属性，
  // 供新的类型检查验证器检查和处理目标方法的局部变量和操作数栈所需要的类型是否匹配
  #28 = Utf8               StackMapTable

  // 类或接口的符号引用，指向第53个常量：java/lang/Object
  #29 = Class              #53            // java/lang/Object

  // 类或接口的符号引用，指向第54个常量：java/lang/Throwable
  #30 = Class              #54            // java/lang/Throwable

  // UTF-8编码的字符串，方法名称常量
  #31 = Utf8               get

  // UTF-8编码的字符串，方法的描述符常量，参数为String返回值为String
  #32 = Utf8               (Ljava/lang/String;)Ljava/lang/String;

  // UTF-8编码的字符串，// TODO
  #33 = Utf8               key

  // UTF-8编码的字符串，String类描述符
  #34 = Utf8               Ljava/lang/String;

  // UTF-8编码的字符串，方法名称常量
  #35 = Utf8               addConfigListener

  // UTF-8编码的字符串，方法的描述符常量
  #36 = Utf8               (Lme/cxis/dcc/listener/ConfigListener;)V

  // UTF-8编码的字符串，// TODO
  #37 = Utf8               configListener

  // UTF-8编码的字符串，ConfigListener接口描述符
  #38 = Utf8               Lme/cxis/dcc/listener/ConfigListener;

  // UTF-8编码的字符串，方法的描述符常量
  #39 = Utf8               (Ljava/lang/String;Lme/cxis/dcc/listener/ConfigListener;)V

  // UTF-8编码的字符串，SourceFile属性，用在类文件，记录源文件名称
  #40 = Utf8               SourceFile

  // UTF-8编码的字符串，源文件名称
  #41 = Utf8               ConfigLoaderDelegate.java

  // 字段或方法的部分符号引用，指向第17和第18个常量
  // "<init>" 方法名 ()参数为空 V返回值为void
  #42 = NameAndType        #17:#18        // "<init>":()V

  // 字段或方法的部分符号引用，指向第13和第14个常量
  // configLoader变量名，Lme/cxis/dcc/loader/ConfigLoader;变量类型
  #43 = NameAndType        #13:#14        // configLoader:Lme/cxis/dcc/loader/ConfigLoader;

  // UTF-8编码的字符串，ZookeeperConfigLoader类描述符
  #44 = Utf8               me/cxis/dcc/loader/ZookeeperConfigLoader

  // 字段或方法的部分符号引用，指向第25和第27个常量
  // getInstance方法名，(Lme/cxis/dcc/loader/ConfigLoader;) 参数，Lme/cxis/dcc/loader/ConfigLoaderDelegate;返回值
  #45 = NameAndType        #25:#27        // getInstance:(Lme/cxis/dcc/loader/ConfigLoader;)Lme/cxis/dcc/loader/ConfigLoaderDelegate;

  // 字段或方法的部分符号引用，configLoaderDelegate常量
  #46 = NameAndType        #15:#16        // configLoaderDelegate:Lme/cxis/dcc/loader/ConfigLoaderDelegate;

  // UTF-8编码的字符串，类描述符
  #47 = Utf8               me/cxis/dcc/loader/ConfigLoaderDelegate

  // 字段或方法的部分符号引用，有参构造方方法
  #48 = NameAndType        #17:#23        // "<init>":(Lme/cxis/dcc/loader/ConfigLoader;)V

  //  类或接口的符号引用，指向第55个常量：me/cxis/dcc/loader/ConfigLoader
  #49 = Class              #55            // me/cxis/dcc/loader/ConfigLoader

  // 字段或方法的部分符号引用，get方法
  #50 = NameAndType        #31:#32        // get:(Ljava/lang/String;)Ljava/lang/String;

  // 字段或方法的部分符号引用，addConfigListener方法
  #51 = NameAndType        #35:#36        // addConfigListener:(Lme/cxis/dcc/listener/ConfigListener;)V

  // 字段或方法的部分符号引用，addConfigListener方法
  #52 = NameAndType        #35:#39        // addConfigListener:(Ljava/lang/String;Lme/cxis/dcc/listener/ConfigListener;)V

  // UTF-8编码的字符串，Object类描述符
  #53 = Utf8               java/lang/Object

  // UTF-8编码的字符串，Throwable描述符
  #54 = Utf8               java/lang/Throwable

  // UTF-8编码的字符串，ConfigLoader类描述符
  #55 = Utf8               me/cxis/dcc/loader/ConfigLoader
{
  // configLoader变量，类型ConfigLoader，访问标示符private，
  private me.cxis.dcc.loader.ConfigLoader configLoader;
    descriptor: Lme/cxis/dcc/loader/ConfigLoader;
    flags: ACC_PRIVATE

  // configLoaderDelegate变量，类型ConfigLoaderDelegate，访问标示符private，static，volatile
  private static volatile me.cxis.dcc.loader.ConfigLoaderDelegate configLoaderDelegate;
    descriptor: Lme/cxis/dcc/loader/ConfigLoaderDelegate;
    flags: ACC_PRIVATE, ACC_STATIC, ACC_VOLATILE

  // 无参构造方法
  private me.cxis.dcc.loader.ConfigLoaderDelegate();
    // 无参数，返回值void
    descriptor: ()V
    // 方法访问标志private
    flags: ACC_PRIVATE
    // Code属性
    Code:
      // stack：最大操作数栈，JVM运行时会根据这个值来分配栈帧中的操作栈深度，此处为1
    
      // locals：局部变量所需的存储空间，单位是Slot变量槽，
      // byte、char、float、int、short、boolean和returnAddress等长度不超过32位的数据类型，每个局部变量占用一个变量槽，
      // 而double和long这两种64位的数据类型则需要两个变量槽来存放
      // 方法参数（包括实例方法中的隐藏参数“this”）、显式异常处理程序的参数（Exception HandlerParameter，就是try-catch语句中catch块中所定义的异常）、
      // 方法体中定义的局部变量都需要依赖局部变量表来存放

      // args_size：方法参数个数，此处的1是指实例方法的隐式参数this
      stack=1, locals=1, args_size=1
         // 将第1个局部变量槽中引用类型的本地变量推送到操作数栈顶
         // 从下面的局部变量表LocalVariableTable中可以看到只有一个this
         0: aload_0

         // invokespecial：调用父类构造方法、实例初始化方法、私有方法
         // #1在常量池中是一个方法引用，指向：java/lang/Object."<init>":()V
         // 这里就是调用父类Object的构造方法，方法的接收者是上一步aload_0推送到栈顶的对象，就是this当前对象
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V

         // return：从方法返回，返回值是void
         4: return
      // 源码行号和字节码行号对应关系
      LineNumberTable:
        line 12: 0
        line 14: 4
      // 栈帧中局部变量与源码中定义的变量之间的关系
      LocalVariableTable:
        // Start： 局部变量的生命周期开始的字节码偏移量，也就是该局部变量在哪一行开始可见
        // Length：作用范围覆盖的长度，也就是可见行数
        // Slot：代表这个局部变量所在栈帧的局部变量表槽的位置
        // Name：局部变量名称，这里是隐式参数this
        // Signature：局部变量描述符，这里是当前ConfigLoaderDelegate类
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lme/cxis/dcc/loader/ConfigLoaderDelegate;

  // 带参数的构造方法
  private me.cxis.dcc.loader.ConfigLoaderDelegate(me.cxis.dcc.loader.ConfigLoader);
    // 参数ConfigLoader，返回值void
    descriptor: (Lme/cxis/dcc/loader/ConfigLoader;)V
    // 访问标示符：private
    flags: ACC_PRIVATE
    // Code属性
    Code:
      // 最大操作数栈，2
      // 两个本地变量槽
      // 两个参数
      stack=2, locals=2, args_size=2
         // 将第1个局部变量槽中引用类型本地变量推送到栈顶
         0: aload_0
         
         // 调用父类的构造方法
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         
         // 将第1个局部变量槽中引用类型本地变量推送到栈顶，this
         4: aload_0
         // 将第2个局部变量槽中引用类型本地变量推送到栈顶，参数configLoader
         5: aload_1

         // 为指定类的实例域赋值，#2是字段引用
         6: putfield      #2                  // Field configLoader:Lme/cxis/dcc/loader/ConfigLoader;

         // 返回 void
         9: return
      // 行号表
      LineNumberTable:
        line 16: 0
        line 17: 4
        line 18: 9
      // 本地变量表
      LocalVariableTable:
        // this和configLoader两个局部变量
        Start  Length  Slot  Name   Signature
            0      10     0  this   Lme/cxis/dcc/loader/ConfigLoaderDelegate;
            0      10     1 configLoader   Lme/cxis/dcc/loader/ConfigLoader;
    // JDK8新增，用在方法表中的变长属性，作用是记录方法的各个形参名称和信息
    // 这里形参是configLoader
    MethodParameters:
      Name                           Flags
      configLoader

  // getInstance方法
  public static me.cxis.dcc.loader.ConfigLoaderDelegate getInstance();
    // 无参，返回值：ConfigLoaderDelegate
    descriptor: ()Lme/cxis/dcc/loader/ConfigLoaderDelegate;
    // 访问标示符 public static
    flags: ACC_PUBLIC, ACC_STATIC
    // Code属性
    Code:
      // 最大操作数栈深度2
      // 本地变量0
      // 参数0
      stack=2, locals=0, args_size=0
         // 创建一个对象并将其引用压入栈顶，创建ZookeeperConfigLoader对象
         0: new           #3                  // class me/cxis/dcc/loader/ZookeeperConfigLoader
         // 复制栈顶数值，并将复制值压入栈顶
         3: dup
         // 调用ZookeeperConfigLoader的构造方法
         4: invokespecial #4                  // Method me/cxis/dcc/loader/ZookeeperConfigLoader."<init>":()V
         // 调用静态方法getInstance
         7: invokestatic  #5                  // Method getInstance:(Lme/cxis/dcc/loader/ConfigLoader;)Lme/cxis/dcc/loader/ConfigLoaderDelegate;
        // 返回对象引用
        10: areturn
      // 行号表
      LineNumberTable:
        line 21: 0

  // getInstance方法
  public static me.cxis.dcc.loader.ConfigLoaderDelegate getInstance(me.cxis.dcc.loader.ConfigLoader);
    // 参数ConfigLoader，返回值ConfigLoaderDelegate
    descriptor: (Lme/cxis/dcc/loader/ConfigLoader;)Lme/cxis/dcc/loader/ConfigLoaderDelegate;
    // 访问标示符public static
    flags: ACC_PUBLIC, ACC_STATIC
    // Code属性
    Code:
      // 最大操作数栈深度3
      // 本地变量表3
      // 参数1个
      stack=3, locals=3, args_size=1
         // 获取#6指向的静态域configLoaderDelegate，并将其压入栈顶
         0: getstatic     #6                  // Field configLoaderDelegate:Lme/cxis/dcc/loader/ConfigLoaderDelegate;

         // 不为null时跳转到38，38是获取#6指向的静态域configLoaderDelegate，并将其压入栈顶
         3: ifnonnull     38

         // 将#7从常量池中推送到栈顶
         6: ldc           #7                  // class me/cxis/dcc/loader/ConfigLoaderDelegate
        
         // 复制栈顶数值，并将复制值压入栈顶
         8: dup
        
         // 将栈顶的引用推送到第二个本地变量槽
         9: astore_1
        
        // 获得对象的锁, 用于同步方法或同步块
        10: monitorenter
        
        // 获取#6指向的静态域configLoaderDelegate，并将其压入栈顶
        11: getstatic     #6                  // Field configLoaderDelegate:Lme/cxis/dcc/loader/ConfigLoaderDelegate;
        
        // 不为null时跳转到28
        14: ifnonnull     28

        // 创建一个对象并将其引用压入栈顶，创建ConfigLoaderDelegate对象
        17: new           #7                  // class me/cxis/dcc/loader/ConfigLoaderDelegate

        // 复制栈顶数值，并将复制值压入栈顶
        20: dup

        // 将第一个本地变量槽中的引用推送到栈顶
        21: aload_0

        // invokespecial：调用父类构造方法、实例初始化方法、私有方法，也就是调用ConfigLoaderDelegate带参的构造方法
        22: invokespecial #8                  // Method "<init>":(Lme/cxis/dcc/loader/ConfigLoader;)V

        // 为指定类的静态域赋值，configLoaderDelegate域赋值刚才new的ConfigLoaderDelegate对象
        25: putstatic     #6                  // Field configLoaderDelegate:Lme/cxis/dcc/loader/ConfigLoaderDelegate;

        // 将第二个本地变量槽中的引用推送到栈顶，引用是ConfigLoaderDelegate
        28: aload_1

        // 释放对象的锁, 用于同步方法或同步块
        29: monitorexit

        // 无条件跳转到38
        30: goto          38

        // 将栈顶的引用推送到第三个本地变量槽
        33: astore_2

        // 将第二个本地变量槽中的引用推送到栈顶
        34: aload_1

        // 释放对象的锁, 用于同步方法或同步块
        35: monitorexit

        // 将第三个本地变量槽中的引用推送到栈顶
        36: aload_2

        // 将栈顶的异常抛出
        37: athrow

        // 获取#6指向的静态域configLoaderDelegate，并将其压入栈顶
        38: getstatic     #6                  // Field configLoaderDelegate:Lme/cxis/dcc/loader/ConfigLoaderDelegate;

        // 返回对象引用
        41: areturn

      // 异常表
      Exception table:
         // from：起始行
         // to：结束行，但不包括结束行
         // target：跳转到继续处理的行
         // 要处理的异常情况，any表示任意异常
         from    to  target type
            11    30    33   any
            33    36    33   any
      
      // 行号表
      LineNumberTable:
        line 25: 0
        line 26: 6
        line 27: 11
        line 28: 17
        line 30: 28
        line 32: 38
      // 本地变量表
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      42     0 configLoader   Lme/cxis/dcc/loader/ConfigLoader;

      // JDK6新增，是一个变长属性，位于Code属性中，这个属性会在虚拟机类加载的字节码验证阶段被新类型检查验证器（Type Checker）使用
      // 目的在于代替以前比较消耗性能的基于数据流分析的类型推导验证器。
      // 包含零至多个栈映射帧（Stack Map Frame），每个栈映射帧都显式或隐式地代表了一个字节码偏移量，用于表示执行到该字节码时局部变量表和操作数栈的验证类型
      // TODO
      StackMapTable: number_of_entries = 3
        frame_type = 252 /* append */
          offset_delta = 28
          locals = [ class java/lang/Object ]
        frame_type = 68 /* same_locals_1_stack_item */
          stack = [ class java/lang/Throwable ]
        frame_type = 250 /* chop */
          offset_delta = 4

    // JDK8新增，用在方法表中的变长属性，作用是记录方法的各个形参名称和信息
    // 这里形参是configLoader
    MethodParameters:
      Name                           Flags
      configLoader

  // get方法
  public java.lang.String get(java.lang.String);
    // 参数String，返回值String
    descriptor: (Ljava/lang/String;)Ljava/lang/String;
    // 访问标识public
    flags: ACC_PUBLIC
    // Code属性
    Code:
      // 最大操作数栈深度2
      // 本地变量2
      // 参数2
      stack=2, locals=2, args_size=2
         // 将第一个本地变量槽中的引用推送到栈顶，本地变量槽中第一个是ConfigLoaderDelegate引用
         0: aload_0

         // 获取指定类的实例域, 并将其压入栈顶， 也就是获取实例域configLoader
         1: getfield      #2                  // Field configLoader:Lme/cxis/dcc/loader/ConfigLoader;

         // 将第二个本地变量槽中的引用推送到栈顶，本地变量槽中第二个是形参String
         4: aload_1

         // 调用接口方法，ConfigLoader的get方法
         5: invokeinterface #9,  2            // InterfaceMethod me/cxis/dcc/loader/ConfigLoader.get:(Ljava/lang/String;)Ljava/lang/String;

        // 返回对象引用
        10: areturn
      // 行号表
      LineNumberTable:
        line 41: 0
      // 本地变量表
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      11     0  this   Lme/cxis/dcc/loader/ConfigLoaderDelegate;
            0      11     1   key   Ljava/lang/String;
    // JDK8新增，用在方法表中的变长属性，作用是记录方法的各个形参名称和信息
    // 这里形参是key
    MethodParameters:
      Name                           Flags
      key

  // addConfigListener方法
  public void addConfigListener(me.cxis.dcc.listener.ConfigListener);
    // 参数ConfigListener，返回值void
    descriptor: (Lme/cxis/dcc/listener/ConfigListener;)V
    // 访问标示符public
    flags: ACC_PUBLIC
    // Code属性
    Code:
      // 操作数栈最大深度2
      // 本地变量表2
      // 参数两个
      stack=2, locals=2, args_size=2
         // 将第一个本地变量槽中的引用推送到栈顶，本地变量槽中第一个是ConfigLoaderDelegate引用
         0: aload_0

         // 获取指定类的实例域, 并将其压入栈顶， 也就是获取实例域configLoader
         1: getfield      #2                  // Field configLoader:Lme/cxis/dcc/loader/ConfigLoader;

         // 将第二个本地变量槽中的引用推送到栈顶，本地变量槽中第二个是形参ConfigListener
         4: aload_1

         // 调用接口方法，ConfigLoader的addConfigListener方法
         5: invokeinterface #10,  2           // InterfaceMethod me/cxis/dcc/loader/ConfigLoader.addConfigListener:(Lme/cxis/dcc/listener/ConfigListener;)V

        // 返回void
        10: return
      // 行号表
      LineNumberTable:
        line 45: 0
        line 46: 10
      // 本地变量表
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      11     0  this   Lme/cxis/dcc/loader/ConfigLoaderDelegate;
            0      11     1 configListener   Lme/cxis/dcc/listener/ConfigListener;
    // JDK8新增，用在方法表中的变长属性，作用是记录方法的各个形参名称和信息
    // 这里形参是configListener
    MethodParameters:
      Name                           Flags
      configListener

  // addConfigListener方法
  public void addConfigListener(java.lang.String, me.cxis.dcc.listener.ConfigListener);
    // 参数ConfigListener，返回值void
    descriptor: (Ljava/lang/String;Lme/cxis/dcc/listener/ConfigListener;)V
    // 访问标示符public
    flags: ACC_PUBLIC
    // Code属性
    Code:
      // 操作数栈最大深度3
      // 本地变量表3
      // 参数三个
      stack=3, locals=3, args_size=3
         // 将第一个本地变量槽中的引用推送到栈顶，本地变量槽中第一个是ConfigLoaderDelegate引用
         0: aload_0
         
         // 获取指定类的实例域, 并将其压入栈顶， 也就是获取实例域configLoader
         1: getfield      #2                  // Field configLoader:Lme/cxis/dcc/loader/ConfigLoader;
         
         // 将第二个本地变量槽中的引用推送到栈顶，本地变量槽中第二个是形参String
         4: aload_1

         // 将第三个本地变量槽中的引用推送到栈顶，本地变量槽中第三个是形参ConfigListener
         5: aload_2

         // 调用接口方法，ConfigLoader的addConfigListener方法
         6: invokeinterface #11,  3           // InterfaceMethod me/cxis/dcc/loader/ConfigLoader.addConfigListener:(Ljava/lang/String;Lme/cxis/dcc/listener/ConfigListener;)V

        // 返回void
        11: return
      // 行号表
      LineNumberTable:
        line 49: 0
        line 50: 11
      // 本地变量表
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      12     0  this   Lme/cxis/dcc/loader/ConfigLoaderDelegate;
            0      12     1   key   Ljava/lang/String;
            0      12     2 configListener   Lme/cxis/dcc/listener/ConfigListener;
    
    // JDK8新增，用在方法表中的变长属性，作用是记录方法的各个形参名称和信息
    // 这里形参是key和configListener
    MethodParameters:
      Name                           Flags
      key
      configListener
}
SourceFile: "ConfigLoaderDelegate.java"
```

# ConfigLoaderDelegate源文件

```java
public class ConfigLoaderDelegate {

    private ConfigLoader configLoader;

    private static volatile ConfigLoaderDelegate configLoaderDelegate;

    private ConfigLoaderDelegate() {

    }

    private ConfigLoaderDelegate(ConfigLoader configLoader) {
        this.configLoader = configLoader;
    }

    public static ConfigLoaderDelegate getInstance() {
        return getInstance(new ZookeeperConfigLoader());
    }

    public static ConfigLoaderDelegate getInstance(ConfigLoader configLoader) {
        if (configLoaderDelegate == null) {
            synchronized (ConfigLoaderDelegate.class) {
                if (configLoaderDelegate == null) {
                    configLoaderDelegate = new ConfigLoaderDelegate(configLoader);
                }
            }
        }
        return configLoaderDelegate;
    }

    /**
     * 根据key获取value
     * @param key ${projectName}.key
     * @return
     */
    public String get(String key) {
        return configLoader.get(key);
    }

    public void addConfigListener(ConfigListener configListener) {
        configLoader.addConfigListener(configListener);
    }

    public void addConfigListener(String key, ConfigListener configListener) {
        configLoader.addConfigListener(key, configListener);
    }
}
```

# ConfigLoader源文件

```java
public interface ConfigLoader {

    String get(String key);

    String parseKey(String key);

    void addConfigListener(ConfigListener configListener);

    void addConfigListener(String key, ConfigListener configListener);
}
```

