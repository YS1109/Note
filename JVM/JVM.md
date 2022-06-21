# JVM学习笔记

## 1.概述

-  JVM是直接运行在操作系统上的，它与硬件没有直接的交互
  - java代码 -> 字节码文件 -> JVM  -> 操作系统
  - 高级语言执行顺序：高级语言 -> 汇编语言 -> 机器指令 -> CPU

- JVM整体结构：
  - ![image-20220621233700205](D:\Note\JVM\images\1.png)
  - 多个线程共享方法区和堆，java栈（现在叫虚拟机栈）、本机方法栈、程序计数器是每个线程独有的
-  Java编译器输入的指令流基本上是一种基于**栈的指令集架构**，另一种指令集架构则是基于**寄存器的指令集架构**
  - 基于栈的指令集架构，指令集小（8位）但具体的指令多；基于寄存器的指令集架构，指令集大（16位）但具体的指令少
  - 基于栈的指令集结构是基于内存的，对硬件依赖较小，可以跨平台，但是性能会有所下降；基于寄存器的指令集结构是基于寄存器的，与硬件耦合性较高，性能较高
- JVM的生命周期：
  - 启动：通过引导类加载器（bootstrap class loader）创建一个初始类（initial class）来完成的，这个类是由虚拟机的具体实现指定的。
  - 执行：
    - 一个运行中的Java虚拟机有着一个清晰的任务：执行Java程序
    - 程序开始执行时他才运行，程序结束时他就停止
    - 执行一个所谓的Java程序的时候，真真正正在执行的是一个叫做Java虚拟机的进程
  - 退出
    - 程序正常执行结束
    - 程序在执行的过程中遇到了异常或错误而异常终止
    - 由于操作系统出现错误而导致Java虚拟机进程终止
    - 某线程调用Runtime类或System类的exit方法，或Runtime类的halt方法，并且Java安全管理器也允许这次exit或halt操作
    - 除此之外，JNI (Java Native Interface)规范描述了用JNI Invocation API来加载或卸载Java虚拟机时，Java虚拟退出的情况
- JVM发展历程：
  - Sun Classic VM：世界上第一款商用Java虚拟机，JDK1.4时完全被淘汰，这款虚拟机内部只提供解释器，如果使用JIT编译器，就需要外挂。但是一旦使用了JIT编译器，JIT就会接管虚拟机的执行系统，解释器就不再工作，解释器和编译器不能配合工作。现在hotspot内置了此虚拟机
  - Exact VM：JDK1.2时。sun提供了此虚拟机，可以知道内存中某个位置的数据具体是什么类型。使用热点探测技术，采用编译器和解释器混合工作模式
  - **HotSpot** VM：JDK和OpenJdk的默认虚拟机，HotSpot 意思是指采用热点代码探测技术
  - **JRockit**：专注于服务器应用，不包含解释器实现，世界上最快的JVM，JDK8将一些功能整合至HotSpot上
  - **J9**：和HotSpot，JRockit并列为三大商用虚拟机，广泛用于IBM的各种Java产品
  - KVM和CDC/ CLDC HotSpot：KVM时CLDC-HI早期产品，面向更低端设备；CDC和CLDC HotSpot主要是Oracle用在JAVAE ME产品线上的虚拟机
  - Azul VM和BEA Liquid VM：与特定硬件平台绑定、软硬件配合的专有虚拟机
  - Apache Harmony：兼容JDK 1.5H和1.6，IBM和Intel联合开发的开源JVM，它的Java类库代码吸纳进了Android SDK
  - Microsoft JVM：微软为了在IE3浏览器中支持JAVA Applets，只能在window平台下运行，当时Windows下性能最好的Java VM
  - TaobaoJVM：基于OpenJDK开发的，深度定制且开源的高性能服务器版Java虚拟机，硬件严重依赖intel的cpu，将生命周期的对象从堆中移至对外，降低GC开销，对象能够在多个Java虚拟机进程中实现共享
  - Dalvik VM：谷歌开发的应用与Android系统的虚拟机，没有遵循Java虚拟机规范，基于寄存器架构，Android 5.0使用支持提前编译的ART VM替换Dalvik VM
  - Graal VM：在HotSopt VM基础上增强而成的跨语言全栈虚拟机，可以作为任何语言的运行平台使用



## 2.类加载子系统

主要步骤：Loading -> Linking -> Initialization即 加载->链接->初始化