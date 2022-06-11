---
title: Java学习之 JDK-JRE-JVM区别和JVM的工作方式
date: 2022-06-05 13:14:00
author: Cialle
categories:
- Java学习
tags:
- JVM
- JDK
- JRE
- java学习
---

# 概述

本章主要了解JDK，JRE和JVM之间的区别。JVM是如何工作的？

# Java程序执行过程

在深入了解Java内存区域之前，我们先了解Java源文件是如何执行的。

1. 我们使用编辑器在 Simple.Java 文件中编写源代码。
2. 前端编译器（javac）将源代码编译为Simple.class文件。
3. 此后缀为.class的类文件可以在任何平台/操作系统的的JVM（Java虚拟机）中执行。
4. JVM负责将字节码转换为机器可执行的本机机器代码。

![img](https://img.cialle.com/9o7qsh7xzt.png)

# 什么是JVM？

Java虚拟机（JVM）是运行Java字节码的虚拟机。可以通过javax将.java文件编译成.class文件。.class文件包含JVM可解析的字节码。

事实上，JVM只是为Java字节码提供了运行时环境和规范。不同的厂商提供此规范的不同实现。例如，[此Wiki页面列出了其它JVM实现](https://en.wikipedia.org/wiki/List_of_Java_virtual_machines)。

最受欢迎的JVM虚拟机是Oracle公司提供的Hostspot虚拟机，（前身是Sun Microsystems，Inc.）。

JVM虚拟机使用许多先进技术，结合了最新的内存模型，垃圾收集器和自适应优化器，为Java应用程序提供了最佳性能。

JVM虚拟机有两种不同的模式：client 模式 和 server 模式。尽管server和client相似，但server进行了特殊调整，以最大程度地提高峰值运行速度。它用于长时间运行的服务器应用程序，它们需要尽可能快的运行速度，而不是快速启动或较小的运行时内存占用量。开发人员可以通过指定-client或-server来选择所需的模式。

JVM之所以称为虚拟机，是因为它提供的API不依赖于底层操作系统和机器硬件体系结构。这种与硬件和操作系统的独立性是Java程序一次写入，随处运行必要基础。

## JVM架构

![img](https://img.cialle.com/202206051415498.png)

### JVM-类加载器

类加载器是用于加载类文件到JVM中。主要分为以下三步 加载，链接和初始化。

#### 加载     

1. 通过加载一个类的全限定名获取定义此类的二进制字节流
2. 将这个字节流所代表的静态存储结构方法转化为方法区的运行时数据结构
3. 在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口

- 类加载器负责从 文件系统 或 网络中加载Class文件，Class文件在文件开头有特定的文件标识(CA FE BA BE)。
- 为了加载类，JVM有3种类加载器。**引导类加载器（BootStrap ClassLoader）**, **扩展类加载器（Extension ClassLoader）**和**应用类加载器（Application ClassLoader）**。
- 加载类文件时，JVM会找到这个类的所有依赖项。
- 首先类加载会判断当前类加载器是否存在父类，如果存在则交给父加载器加载。
- **引导类加载器（BootStrap ClassLoader）**为根类加载器，它将尝试查找该类。扫描**jre/lib/rt.jar**。
- 如果找不到类，那么**扩展类加载器（Extension ClassLoader）**加载器将在**jre/lib/ext**包中搜索类文件。
- 如果还找不到类，则**应用类加载器（Application ClassLoader）**将在系统的**CLASSPATH环境变量**中搜索所有Jar文件和类
- 任何类加载程序找到了类，则由该类加载器加载类；否则抛出ClassNotFoundException。
- **类加载器（ClassLoader）**只负责Class文件的加载，至于他是否可以运行，则有**执行引擎（Execution Engine ）**决定。

#### 链接 

1. 验证
   - 目的在于确保Class文件的字节流中包含信息符合当前虚拟机要求，确保被加载类的正确性，不会危害虚拟机自身安全。
   - 主要包括四种验证，文件格式验证，元数据验证，字节码验证，符号引用认证。
2. 准备
   - 为类变量分配内存并且设置该类的默认初始值，即零值。
   - **这里不包含用final修饰的static**，因为final在编译的时候就会被分配了，准备阶段会显式初始化。
   - **这里不会为实例变量分配初始化**，类变量会分配在方法区中，而实例变量是会随着对象一起分配到Java堆之中。
3. 解析
   - 将常量池的符号引用转换为直接引用的过程
   - 事实上，**解析操作往往会伴随着JVM在执行完初始化以后再执行**。
   - 符号引用就是一组符号来描述所有的目标。符号引用的字面量形式明确定义《java虚拟机规范》的Class文件格式中。直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄。
   - 解析动作主要针对类、字段、类方法、接口方法、方法类型等。对应常量池中的CONSTANT_Class_info、CONSTANT_Fieldref_info、CONSTANT_Methodref_info等

#### 初始化

- 初始化阶段就是执行类构造器方法`<clinit>()`的过程。
- 此方法无需定义，是javac编译器自动收集类中的所有类变量的赋值动作和静态代码块中的语句合并而来。
- 构造器方法中的指令按语句在源文件中出现的顺序执行。
- **`<clinit>()`不同于类的构造器**。（关联：构造器是虚拟机视角下的`<init>()`）
- 若该类具有父类，JVM会保证子类的`<clinit>()`执行前，父类的`<clinit>()`已经执行完毕。
- 虚拟机必须保证一个类的`<clinit>()`方法在多线程下被同步加锁。

### JVM-内存区域

JVM中的内存区域分为多个部分，以存储应用程序数据的特定部分。

- 方法区：存储类结构，例如类的基本信息，常量运行时池和方法代码。
- 堆：存储在应用程序执行期间创建的所有对象。
- 栈：存储局部变量和中间结果。所有这些变量对于创建它们的线程都是私有的。每个线程都有自己的`JVM`栈，并在创建线程时同时创建。因此，所有此类局部变量都称为线程局部变量。
- PC寄存器：存储当前正在执行的语句的物理内存地址。在Java中，每个线程都有其单独的PC寄存器。
- 本地方法区：许多底层代码都是用C和C ++等语言编写的。本地方法栈保存本机代码的指令。

### JVM-执行引擎

分配给JVM的所有代码均由执行引擎执行。执行引擎读取字节码并一一执行。它使用两个内置的解释器和JIT编译器将字节码转换为机器码并执行。

![img](https://img.cialle.com/202206051412529.png)

使用JVM，解释器和编译器均会生成本机代码。不同之处在于它们如何生成本机代码，其优化程度以及优化成本。

#### 解释器

JVM解释器通过查找预定义的JVM指令到机器指令的映射，几乎将每个字节码指令转换为相应的本机指令。它直接执行字节码，不执行任何优化。

#### JIT编译器

为了提高性能，JIT编译器在运行时与JVM交互，并将适当的字节码序列编译为本地机器代码。通常，JIT编译器采用一段代码（和解释器一次一条语句不一样），优化代码，然后将其转换为优化的机器代码。

默认情况下，JIT编译器处于启用状态。您可以禁用JIT编译器，在这种情况下，解释器将要解释整个Java程序。除了诊断或解决JIT编译问题外，不建议禁用JIT编译器。

# 什么是JRE

Java运行时环境（JRE）是一个软件包，它将库（jar）和Java虚拟机以及其他组件捆绑在一起，以运行用Java编写的应用程序。JRE只是JVM的一部分。

要执行Java应用程序，只需要在计算机中安装JRE。 这是在计算机上执行Java应用程序都是最低要求。

JRE包含了以下组件–

1. Java HotSpot客户端虚拟机使用的DLL文件。
2. Java HotSpot服务器虚拟机使用的DLL文件。
3. Java运行时环境使用的代码库，属性设置和资源文件。例如rt.jar和charsets.jar。
4. Java扩展文件，例如localedata.jar。
5. 包含用于安全管理的文件。这些文件包括安全策略（java.policy）和安全属性（java.security）文件。
6. 包含applet支持类的Jar文件。
7. 包含供平台使用的TrueType字体文件。

JRE可以作为JDK的一部分下载，也可以单独下载。JRE与平台有关。您可以根据您的计算机的类型（操作系统和体系结构）选择要导入和安装的JRE软件包。

比如，你不能在32位计算机上安装64位JRE。同样，用于Windows的JRE发行版在Linux上将无法运行。反之亦然。

# 什么是JDK

JDK比JRE更加全面。JDK包含JRE拥有的所有部门以及用于开发，调试和监视Java应用程序的开发工具。当需要开发Java应用程序时，需要JDK。

JDK附带的几个重要组件如下：

- appletviewer –此工具可用于在没有Web浏览器的情况下运行和调试Java applet
- apt –注释处理工具
- extcheck –一种检测JAR文件冲突的实用程序
- javadoc –文档生成器，可从源代码注释自动生成文档
- jar –存档程序，它将相关的类库打包到一个JAR文件中。该工具还有助于管理JAR文件
- jarsigner – jar签名和验证工具javap –类文件反汇编程序
- javaws – JNLP应用程序的Java Web Start启动器
- JConsole – Java监视和管理控制台
- jhat – Java堆分析工具
- jrunscript – Java命令行脚本外壳
- jstack –打印Java线程的Java堆栈跟踪的实用程序
- keytool –用于操作密钥库的工具
- policytool –策略创建和管理工具
- xjc – XML绑定Java API（JAXB）API的一部分。它接受XML模式并生成Java类

与JRE一样，JDK也依赖于平台。因此，在为您的计算机下载JDK软件包时请多加注意。

# JDK，JRE和JVM之间的区别

基于以上讨论，我们可以得出以下这三者之间的关系

JRE = JVM + libraries to run Java application.

JDK = JRE + tools to develop Java Application.

![img](https://img.cialle.com/202206051410581.png)

简而言之，如果你是编写代码的Java应用程序开发人员，则需要在计算机中安装JDK。但是，如果只想运行用Java内置的应用程序，则只需要在计算机上安装JRE。

小结：暂时记录这些，后续学到JVM再重新编写更多细节。
