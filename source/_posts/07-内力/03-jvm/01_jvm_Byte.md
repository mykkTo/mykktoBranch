---
title: JVM-01-字节码
author: mykk
top: false
cover: false
toc: true
mathjax: false
summary: 本编是JVM整篇的子篇，主要记录学习JVM过程，达到工程使用应付面试题
tags:
  - jvm
  - 字节码
categories:
  - 高阶篇
abbrlink: 6ae84987
reprintPolicy: cc_by
date: 2022-04-16 21:00:12
coverImg:
img:
password:
---



## 一、JVM概述

### 1、Java的生态



#### 1、Oracle JDK与Open JDK 关系



##### Oracle与OpenJDK之间的主要区别：

1. Oracle JDK版本将每三年发布一次LTS版本，而OpenJDK版本每三个月发布一次。
2. Oracle JDK将更多地关注稳定性，它重视更多的企业级用户，而OpenJDK经常发布以支持其他性能，这可能会导致不稳定。
3. Oracle JDK支持长期发布的更改，而Open JDK仅支持计划和完成下一个发行版。
4. Oracle JDK根据二进制代码许可协议获得许可，而OpenJDK根据GPL v2许可获得许可。 使用Oracle平台时会产生一些许可影响。如Oracle 宣布的那样，在没有商业许可的情况下，在2019年1月之后发布的Oracle Java SE 8的公开更新将无法用于商业，商业或生产用途。但是，OpenJDK是完全开源的，可以自由使用。
5. Oracle JDK的构建过程基于OpenJDK，因此OpenJDK与Oracle JDK之间没有技术差异。
6. 顶级公司正在使用Oracle JDK，例如Android Studio，Minecraft和IntelliJ IDEA开发工具，其中Open JDK不太受欢迎。
7. Oracle JDK具有Flight Recorder，Java Mission Control和Application Class-Data Sharing功能，Open JDK具有Font Renderer功能，这是OpenJDK与Oracle JDK之间的显着差异。
8. Oracle JDK具有良好的GC选项和更好的渲染器，而OpenJDK具有更少的GC选项，并且由于其包含自己的渲染器的分布，因此具有较慢的图形渲染器选项。
9. 在响应性和JVM性能方面，Oracle JDK与OpenJDK相比提供了更好的性能。
10. 与OpenJDK相比，Oracle JDK的开源社区较少，OpenJDK社区用户的表现优于Oracle JDK发布的功能，以提高性能。
11. 如果使用Oracle JDK会产生许可影响，而OpenJDK没有这样的问题，并且可以以任何方式使用，以满足完全开源和免费使用。
12. Oracle JDK在运行JDK时不会产生任何问题，而OpenJDK在为某些用户运行JDK时会产生一些问题。
13. 根据使用方的使用和许可协议，现有应用程序可以从Oracle JDK迁移到Open JDK，反之亦然。
14. Oracle JDK将从其10.0.X版本将收费，用户必须付费或必须依赖OpenJDK才能使用其免费版本。
15. Oracle JDK不会为即将发布的版本提供长期支持，用户每次都必须通过更新到最新版本获得支持来获取最新版本。
16. Oracle JDK以前的1.0版以前的版本是由Sun开发的，后来被Oracle收购并为其他版本维护，而OpenJDK最初只基于Java SDK或JDK版本7。
17. Oracle JDK发布时大多数功能都是开源的，其中一些功能免于开源，并且根据Sun的许可授权，而OpenJDK发布了所有功能，如开源和免费。
18. Oracle JDK完全由Oracle公司开发，而Open JDK项目由IBM，Apple，SAP AG，Redhat等顶级公司加入和合作。



#### 2、JDK与JVM是什么关系

![](https://v1.mykkto.cn/image/blog/2022/jvm/20220416210355.png)



##### 1、如何理解Java是跨平台的语言

![](https://v1.mykkto.cn/image/blog/2022/jvm/20220416210431.png)



当Java源代码成功编译成字节码后，如果想在不同的平台上面运行，则无须再次编译
这个优势不再那么吸引人了。Python、PHP、Perl、Ruby、Lisp等有强大的解释器。跨平台似乎已经快成为一门语言必选的特性。



##### 2、如何理解JVM跨语言的平台

![](https://v1.mykkto.cn/image/blog/2022/jvm/20220416210508.png)



Java虚拟机根本不关心运行在其内部的程序到底是使用何种编程语言编写的，`它只关心“字节码”文件。`

**Java不是最强大的语言，但是JVM是最强大的虚拟机。**



#### 3、Java发展的几个重大事件

2000年，JDK 1.3发布，`Java HotSpot Virtual Machine正式发布，成为Java的默认虚拟机。`
2002年，JDK 1.4发布，古老的Classic虚拟机退出历史舞台。
2003年年底，`Java平台的Scala正式发布，同年Groovy也加入了 Java阵营。`
2006年，JDK 6发布。同年，`Java开源并建立了 OpenJDK`。顺理成章，`Hotspot虚拟机也成为了 OpenJDK中的默认虚拟机。`
2007年，`Java平台迎来了新伙伴Clojure。`
2008 年，Oracle 收购了 BEA,得到了 `JRockit 虚拟机`。
2009年，Twitter宣布把后台大部分程序从Ruby迁移到Scala，这是Java平台的又一次大规模应用。
2010年，Oracle收购了Sun，`获得Java商标和最具价值的HotSpot虚拟机`。此时，Oracle拥有市场占用率最高的两款虚拟机HotSpot和JRockit，并计划在未来对它们进行整合：HotRockit.  JCP组织管理：Java语言
2011年，JDK7发布。在JDK 1.7u4中，`正式启用了新的垃圾回收器G1`。
2017年，JDK9发布。`将G1设置为默认GC，替代CMS (被标记为Deprecated)`
同年，`IBM的J9开源`，形成了现在的Open J9社区
2018年，Android的Java侵权案判决，Google赔偿Oracle计88亿美元
同年，JDK11发布，LTS版本的JDK`,发布革命性的ZGC,调整JDK授权许可`
2019年，JDK12发布，加入RedHat领导开发的`Shenandoah GC`





### 2、JVM的架构



#### 1、JVM架构图

### ![](https://v1.mykkto.cn/image/blog/2022/jvm/20220417095344.png)

![](https://v1.mykkto.cn/image/blog/2022/jvm/20220417095602.png)



**这个架构可以分成三层看：**

最上层：javac编译器将编译好的字节码class文件，通过java 类装载器执行机制，把对象或class文件存放在 jvm划分内存区域。
中间层：称为Runtime Data Area，主要是在Java代码运行时用于存放数据的，从左至右为方法区(永久代、元数据区)、堆(共享,GC回收对象区域)、栈、程序计数器、寄存器、本地方法栈(私有)。
最下层：解释器、JIT(just in time)编译器和 GC（Garbage Collection，垃圾回收器）



#### 2、JVM脉络

![](https://v1.mykkto.cn/image/blog/2022/jvm/20220417095808.png)



## 二、字节码文件概述

### 1、字节码文件跨平台

**Class文件结构不仅仅是Java虚拟机的执行入口，更是Java生态圈的基础和核心。**

#### 1、class文件里是什么

**字节码文件里是什么？**

源代码经过编译器编译之后便会生成一个字节码文件，字节码是一种二进制的类文件，它的内容是JVM的指令，而不像C、C++经由编译器直接生成机器码。

随着Java平台的不断发展，在将来，Class文件的内容也一定会做进一步的扩充，但是其基本的格式和结构不会做重大调整。

#### 2、☆ class文件的编译器

##### 1、从位置上理解

![](https://v1.mykkto.cn/image/blog/2022/jvm/20220417101023.png)



##### 2、前端编译器的种类

Java源代码的编译结果是字节码，那么肯定需要有一种编译器能够将Java源码编译为字节码，承担这个重要责任的就是配置在path环境变量中的`javac编译器`。javac是一种能够将Java源码编译为字节码的`前端编译器`。

HotSpot VM并没有强制要求前端编译器只能使用javac来编译字节码，其实只要编译结果符合JVM规范都可以被JVM所识别即可。
在Java的前端编译器领域，除了javac之外，还有一种被大家经常用到的前端编译器，那就是内置在Eclipse中的`ECJ (Eclipse Compiler for Java)`编译器。和Javac的全量式编译不同，ECJ是一种增量式编译器。

默认情况下，IntelliJ IDEA 使用 javac 编译器。(还可以自己设置为AspectJ编译器 ajc)

##### 3、前端编译器的任务

前端编译器的主要任务就是负责将符合Java语法规范的`Java代码`转换为符合JVM规范的`字节码文件`。



#### 3、前端编译器的局限性

前端编译器并不会直接涉及编译优化等方面的技术，而是将这些具体优化细节移交给HotSpot的JIT编译器负责。

复习：AOT(静态提前编译器，Ahead Of Time Compiler)

jdk9引入了AOT编译器(静态提前编译器，Ahead Of Time Compiler)

Java 9 引入了实验性 AOT 编译工具jaotc。它借助了 Graal 编译器，将所输入的 Java 类文件转换为机器码，并存放至生成的动态共享库之中。

所谓 AOT 编译，是与即时编译相对立的一个概念。我们知道，即时编译指的是在程序的运行过程中，将字节码转换为可在硬件上直接运行的机器码，并部署至托管环境中的过程。而 AOT 编译指的则是，在程序运行之前，便将字节码转换为机器码的过程。
.java -> .class -> .so

最大好处：Java虚拟机加载已经预编译成二进制库，可以直接执行。不必等待即时编译器的预热，减少Java应用给人带来“第一次运行慢”的不良体验。

缺点：
破坏了java“一次编译，到处运行”，必须为每个不同硬件、OS编译对应的发行包。
降低了Java链接过程的动态性，加载的代码在编译期就必须全部已知。

还需要继续优化中，最初只支持Linux x64 java base



### 2、Class对象对应类型



> （1）class：
> 外部类，成员(成员内部类，静态内部类)，局部内部类，匿名内部类
> （2）interface：接口
> （3）[]：数组
> （4）enum：枚举
> （5）annotation：注解@interface
> （6）primitive type：基本数据类型
> （7）void



```java
@Test
public void test(){
    Class c1 = Object.class;
    Class c2 = Comparable.class;
    Class c3 = String[].class;
    Class c4 = int[][].class;
    Class c5 = ElementType.class;
    Class c6 = Override.class;
    Class c7 = int.class;
    Class c8 = void.class;
    Class c9 = Class.class;
 
    int[] a = new int[10];
    int[] b = new int[100];
    Class c10 = a.getClass();
    Class c11 = b.getClass();
    // 只要元素类型与维度一样，就是同一个Class
    System.out.println(c10 == c11);
}
```





### 3、字节码指令



#### 1、什么是字节码指令

Java虚拟机的指令由一个字节长度的、代表着某种特定操作含义的`操作码（opcode）`以及跟随其后的零至多个代表此操作所需参数的`操作数（operand）`所构成。虚拟机中许多指令并不包含操作数，只有一个操作码。

![](https://v1.mykkto.cn/image/blog/2022/jvm/20220417101919.png)



#### 2、☆ 面试题

**i++和++i有什么区别**



```java
public class ByteCodeInterview {
    //面试题： i++和++i有什么区别？
    @Test
    public void test1(){
        int i = 10;
        i++;
        //++i;

        System.out.println(i);
    }

    @Test
    public void test2(){
        int i = 10;
        i = i++;
        System.out.println(i);
    }

    @Test
    public void test3(){
        int i = 2;
        i *= i++;
        System.out.println(i);
    }

    @Test
    public void test4(){
        int k = 10;
        k = k + (k++) + (++k);
        System.out.println(k);
    }

    //包装类对象的缓存问题
    @Test
    public void test5(){
//        Integer x = 5;
//        int y = 5;

        Integer i1 = 10;
        Integer i2 = 10;
        System.out.println(i1 == i2);

        Integer i3 = 128;
        Integer i4 = 128;
        System.out.println(i3 == i4);

        Boolean b1 = true;
        Boolean b2 = true;
        System.out.println(b1 == b2);
    }
    
    @Test
    public void test6(){
        String str = new String("hello") + new String("world");
        String str1 = "helloworld";
        System.out.println(str == str1);
    }
}
```



![](https://v1.mykkto.cn/image/blog/2022/jvm/20220417102212.png)



```java
class Father {
    int x = 10;
    public Father() {
        this.print();
        x = 20;
    }
    public void print() {
        System.out.println("Father.x = " + x);
    }
}

class Son extends Father {
    int x = 30;
    public Son() {
        this.print();
        x = 40;
    }
    public void print() {
        System.out.println("Son.x = " + x);

    }
}

public class SonTest {
    public static void main(String[] args) {
        Father f = new Son();
        System.out.println(f.x);
    }
} 
```





### 4、如何解读class文件


方式一：一个一个二进制的看。这里用到的是Notepad++,需要安装一个HEX-Editor插件，或者使用Binary Viewer

![](https://v1.mykkto.cn/image/blog/2022/jvm/20220417102826.png)


方式二：使用javap指令：jdk自带的反解析工具
方式三：使用IDEA插件：jclasslib 或jclasslib bytecode viewer客户端工具。（可视化更好）

![](https://v1.mykkto.cn/image/blog/2022/jvm/20220417102924.png)

 



## 三、Class文件结构细节

### 1、概述

#### 1、class文件结构概述

Class文件的结构并不是一成不变的，随着Java虚拟机的不断发展，总是不可避免地会对Class文件结构做出一些调整，但是其基本结构和框架是非常稳定的。
Class文件的总体结构如下：

- 魔数
- Class文件版本
- 常量池
- 访问标识(或标志)
- 类索引，父类索引，接口索引集合
- 字段表集合
- 方法表集合
- 属性表集合



#### 2、结构图表

 这是一张Java字节码总的结构表，我们按照上面的顺序逐一进行解读就可以了。

![](https://v1.mykkto.cn/image/blog/2022/jvm/20220417103240.png)



### 2、魔数

**Magic Number（魔数）：class文件的标志**

- 每个 Class 文件开头的4个字节的无符号整数称为魔数（Magic Number）

- 它的唯一作用是确定这个文件是否为一个能被虚拟机接受的有效合法的Class文件。即：魔数是Class文件的标识符。

- 魔数值固定为0xCAFEBABE。不会改变。

- 如果一个Class文件不以0xCAFEBABE开头，虚拟机在进行文件校验的时候就会直接抛出以下错误：

  >Error: A JNI error has occurred, please check your installation and try again
  >Exception in thread "main" java.lang.ClassFormatError: Incompatible magic value 1885430635 in class file StringTest

- 使用魔数而不是扩展名来进行识别主要是基于安全方面的考虑，因为文件扩展名可以随意地改动。



### 3、高版本执行低版本Class

**不同版本的Java编译器编译的Class文件对应的版本是不一样的。目前，高版本的Java虚拟机可以执行由低版本编译器生成的Class文件,但是低版本的Java虚拟机不能执行由高版本编译器生成的Class文件。否则JVM会抛出java.lang.UnsupportedClassVersionError异常。 （向下兼容）**

在实际应用中，由于开发环境和生产环境的不同，可能会导致该问题的发生。因此，需要我们在开发时，特别注意开发编译的JDK版本和生产环境中的JDK版本是否一致。



#### class文件版本号

- 紧接着魔数的 4 个字节存储的是 Class 文件的版本号。同样也是4个字节。第5个和第6个字节所代表的含义就是编译的副版本号minor_version，而第7个和第8个字节就是编译的主版本号major_version。
- 它们共同构成了class文件的格式版本号。譬如某个 Class 文件的主版本号为 M，副版本号为 m，那么这个Class 文件的格式版本号就确定为 M.m。
- 版本号和Java编译器的对应关系如下表：

![](https://v1.mykkto.cn/image/blog/2022/jvm/20220417162510.png)

- Java 的版本号是从45开始的，JDK 1.1之后的每个JDK大版本发布主版本号向上加1。

- 虚拟机JDK版本为1.k （k >= 2）时，对应的class文件格式版本号的范围为45.0 - 44+k.0 （含两端）。



### 4、☆ 常量池

**常量池：class文件的基石？作用是？**

#### 1、为什么需要常量池计数器

**constant_pool_count （常量池计数器）**

- 由于常量池的数量不固定，时长时短，所以需要放置两个字节来表示常量池容量计数值。

- 常量池容量计数值（u2类型）：从1开始，表示常量池中有多少项常量。即constant_pool_count=1表示常量池中有0个常量项。

- Demo的值为：

  - 其值为0x0016,掐指一算，也就是22。
    需要注意的是，这实际上只有21项常量。索引为范围是1-21。为什么呢？
    通常我们写代码时都是从0开始的，但是这里的常量池却是从1开始，因为它把第0项常量空出来了。这是为了满足后面某些指向常量池的索引值的数据在特定情况下需要表达“不引用任何一个常量池项目”的含义，这种情况可用索引值0来表示。

  - ```java
    int[] arr = new int[10];
    arr[0];
    arr[1];
    
    ar[10 - 1];
    ```

    

#### 2、常量池表

**constant_pool []（常量池）**

- constant_pool是一种表结构，以 1 ~ constant_pool_count - 1为索引。表明了后面有多少个常量项。
- 常量池主要存放两大类常量：`字面量（Literal）`和`符号引用（Symbolic References）`
- 它包含了class文件结构及其子结构中引用的所有字符串常量、类或接口名、字段名和其他常量。常量池中的每一项都具备相同的特征。第1个字节作为类型标记，用于确定该项的格式，这个字节称为tag byte （标记字节、标签字节）。

![](https://v1.mykkto.cn/image/blog/2022/jvm/20220417163310.png)



##### 1、字面量和符号引用

1、字面量和符号引用
在对这些常量解读前，我们需要搞清楚几个概念。
常量池主要存放两大类常量：字面量（Literal）和符号引用（Symbolic References）。如下表：

![](https://v1.mykkto.cn/image/blog/2022/jvm/20220417164111.png)

```java
String str = "mykk";
final int NUM = 10;
```



2、全限定名
com/kk/test/Demo这个就是类的全限定名，仅仅是把包名的"."替换成"/"，为了使连续的多个全限定名之间不产生混淆，在使用时最后一般会加入一个“;”表示全限定名结束。

3、简单名称
简单名称是指没有类型和参数修饰的方法或者字段名称，上面例子中的类的add()方法和num字段的简单名称分别是add和num。

4、描述符
`描述符的作用是用来描述字段的数据类型、方法的参数列表（包括数量、类型以及顺序）和返回值。`根据描述符规则，基本数据类型（byte、char、double、float、int、long、short、boolean）以及代表无返回值的void类型都用一个大写字符来表示，而对象类型则用字符L加对象的全限定名来表示，详见下表:   （数据类型：基本数据类型 、 引用数据类型）

![](https://v1.mykkto.cn/image/blog/2022/jvm/20220417164344.png)

用描述符来描述方法时，按照先参数列表，后返回值的顺序描述，参数列表按照参数的严格顺序放在一组小括号“()”之内。如：
方法java.lang.String toString()的描述符为() Ljava/lang/String;，
方法int abc(int[] x, int y)的描述符为([II) I。

##### 2、谈谈你对符号引用、直接引用的理解？

这里说明下符号引用和直接引用的区别与关联：
符号引用：符号引用以一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用时能无歧义地定位到目标即可。符号引用与虚拟机实现的内存布局无关，引用的目标并不一定已经加载到了内存中。 
直接引用：直接引用可以是直接指向目标的指针、相对偏移量或是一个能间接定位到目标的句柄。直接引用是与虚拟机实现的内存布局相关的，同一个符号引用在不同虚拟机实例上翻译出来的直接引用一般不会相同。如果有了直接引用，那说明引用的目标必定已经存在于内存之中了。

##### 3、常量类型和结构

常量池中每一项常量都是一个表，JDK1.7之后共有14种不同的表结构数据。如下表格所示：

![](https://v1.mykkto.cn/image/blog/2022/jvm/20220417165020.png)



这14种表（或者常量项结构）的共同点是：表开始的第一位是一个u1类型的标志位（tag），代表当前这个常量项使用的是哪种表结构，即哪种常量类型。

在常量池列表中，CONSTANT_Utf8_info常量项是一种使用改进过的UTF-8编码格式来存储诸如文字字符串、类或者接口的全限定名、字段或者方法的简单名称以及描述符等常量字符串信息。

这14种常量项结构还有一个特点是，其中13个常量项占用的字节固定，只有CONSTANT_Utf8_info占用字节不固定，其大小由length决定。为什么呢？因为从常量池存放的内容可知，其存放的是字面量和符号引用，最终这些内容都会是一个字符串，这些字符串的大小是在编写程序时才确定，比如你定义一个类，类名可以取长取短，所以在没编译前，大小不固定，编译后，通过utf-8编码，就可以知道其长度。



#### 3、访问标识

**访问标识(access_flag、访问标志、访问标记)**



- 在常量池后，紧跟着访问标记。该标记使用两个字节表示，用于识别一些类或者接口层次的访问信息，包括：这个 Class 是类还是接口；是否定义为 public 类型；是否定义为 abstract 类型；如果是类的话，是否被声明为 final 等。各种访问标记如下所示：

  ![](https://v1.mykkto.cn/image/blog/2022/jvm/20220417165620.png)

- 类的访问权限通常为 ACC_ 开头的常量。

- 每一种类型的表示都是通过设置访问标记的32位中的特定位来实现的。比如，若是public final的类，则该标记为ACC_PUBLIC | ACC_FINAL。

- 使用ACC_SUPER可以让类更准确地定位到父类的方法super.method(),现代编译器都会设置并且使用这个标记。



#### 4、类索引、父类索引、接口索引集合

- 在访问标记后，会指定该类的类别、父类类别以及实现的接口，格式如下：

  ![](https://v1.mykkto.cn/image/blog/2022/jvm/20220417165738.png)

- 这三项数据来确定这个类的继承关系。

  - 类索引用于确定这个类的全限定名
  - 父类索引用于确定这个类的父类的全限定名。由于 Java语言不允许多重继承，所以父类索引只有一个，除了java.lang.Object 之外，所有的Java类都有父类，因此除了java.lang.Object 外，所有Java类的父类索引都不为 0。
  - 接口索引集合就用来描述这个类实现了哪些接口，这些被实现的接口将按 implements 语句（如果这个类本身是一个接口，则应当是 extends 语句）后的接口顺序从左到右排列在接口索引集合中。

- this_class（类索引）

  - 字节无符号整数，指向常量池的索引。它提供了类的全限定名,如com/kk/java1/Demo。this_class的值必须是对常量池表中某项的一个有效索引值。常量池在这个索引处的成员必须为CONSTANT_Class_info类型结构体，该结构体表示这个class文件所定义的类或接口。

- super_class （父类索引）

  - 2字节无符号整数，指向常量池的索引。它提供了当前类的父类的全限定名。如果我们没有继承任何类，其默认继承的是java/lang/Object类。同时，由于Java不支持多继承，所以其父类只有一个。
  - superclass指向的父类不能是final。

- interfaces

  - 指向常量池索引集合，它提供了一个符号引用到所有已实现的接口
  - 由于一个类可以实现多个接口，因此需要以数组形式保存多个接口的索引，表示接口的每个索引也是一个指向常量池的CONSTANT_Class (当然这里就必须是接口，而不是类)。
  - interfaces_count (接口计数器)
    - interfaces_count项的值表示当前类或接口的直接超接口数量。
  - interfaces []\(接口索引集合)
    - interfaces []中每个成员的值必须是对常量池表中某项的有效索引值，它的长度为 interfaces_count。 每个成员 interfaces[i]必须为 CONSTANT_Class_info结构，其中 0 <= i < interfaces_count。在 interfaces[]中，各成员所表示的接口顺序和对应的源代码中给定的接口顺序（从左至右）一样，即` interfaces[0]对应的是源代码中最左边的接口。`

  

#### 5、字段表集合



##### 1、字段计数器

- fields_count的值表示当前class文件fields表的成员个数。使用两个字节来表示。
- fields表中每个成员都是一个field_info结构，用于表示该类或接口所声明的所有类字段或者实例字段，不包括方法内部声明的变量，也不包括从父类或父接口继承的那些字段

##### 2、字段表，fields []

- fields表中的每个成员都必须是一个fields_info结构的数据项，用于表示当前类或接口中某个字段的完整描述。

- 一个字段的信息包括如下这些信息。这些信息中，`各个修饰符都是布尔值，要么有，要么没有`。

  - 作用域（public、private、protected修饰符）
  - 是实例变量还是类变量（static修饰符）
  - 可变性（final）
  - 并发可见性（volatile修饰符，是否强制从主内存读写）
  - 可否序列化（transient修饰符）
  - 字段数据类型（基本数据类型、对象、数组）
  - 字段名称

- 字段表结构

  ![](https://v1.mykkto.cn/image/blog/2022/jvm/20220417171658.png)



#### 6、方法表集合

##### 1、方法计数器

- methods_count的值表示当前class文件methods表的成员个数。使用两个字节来表示。
- methods 表中每个成员都是一个method_info结构。

##### 2、方法表

- methods表中的每个成员都必须是一个method_info结构，用于表示当前类或接口中某个方法的完整描述。如果某个method_info结构的access_flags项既没有设置 ACC_NATIVE 标志也没有设置ACC_ABSTRACT标志，那么该结构中也应包含实现这个方法所用的Java虚拟机指令。

- method_info结构可以表示类和接口中定义的所有方法，包括实例方法、类方法、实例初始化方法和类或接口初始化方法

- 方法表的结构实际跟字段表是一样的，方法表结构如下：

  ![](https://v1.mykkto.cn/image/blog/2022/jvm/20220417211001.png)





#### 7、属性表集合

##### 1、属性计数器

attributes_count的值表示当前class文件属性表的成员个数。属性表中每一项都是一个attribute_info结构。

##### 2、属性表

- ConstantValue 属性

  - ConstantValue 属性表示一个常量字段的值。位于 field_info结构的属性表中。

  - ```java
    ConstantValue_attribute {
        u2 attribute_name_index;
        u4 attribute_length;
        u2 constantvalue_index;//字段值在常量池中的索引，常量池在该索引处的项给出该属性表示的常量值。（例如，值是long型的，在常量池中便是CONSTANT_Long）
    }
    ```

- Deprecated 属性

  - Deprecated 属性是在 JDK 1.1 为了支持注释中的关键词@deprecated 而引入的。

    ```java
    Deprecated_attribute {
        u2 attribute_name_index;
         u4 attribute_length;
    }
    ```
  
- Code 属性

  - Code属性就是存放方法体里面的代码。但是，并非所有方法表都有Code属性。像接口或者抽象方法，他们没有具体的方法体，因此也就不会有Code属性了。
    Code属性表的结构,如下图：

    ![](https://v1.mykkto.cn/image/blog/2022/jvm/20220417213251.png)

    可以看到：Code属性表的前两项跟属性表是一致的，即Code属性表遵循属性表的结构，后面那些则是他自定义的结构。

- InnerClasses 属性

  - 为了方便说明特别定义一个表示类或接口的 Class 格式为 C。如果 C 的常量池中包含某个CONSTANT_Class_info 成员，且这个成员所表示的类或接口不属于任何一个包，那么 C 的ClassFile 结构的属性表中就必须含有对应的 InnerClasses 属性。InnerClasses 属性是在 JDK 1.1 中为了支持内部类和内部接口而引入的,位于 ClassFile结构的属性表。

- LineNumberTable 属性

  - LineNumberTable 属性是可选变长属性，位于 Code结构的属性表。

  - LineNumberTable属性是用来描述Java源码行号与字节码行号之间的对应关系。这个属性可以用来在调试的时候定位代码执行的行数。

  - start_pc,即字节码行号;line_number，即Java源代码行号。

  - 在 Code 属性的属性表中,LineNumberTable 属性可以按照任意顺序出现，此外，多个 LineNumberTable属性可以共同表示一个行号在源文件中表示的内容，即 LineNumberTable 属性不需要与源文件的行一一对应。

    ![](https://v1.mykkto.cn/image/blog/2022/jvm/20220417220235.png)

- LocalVariableTable 属性

  -  LocalVariableTable 是可选变长属性，位于 Code属性的属性表中。它被调试器用于确定方法在执行过程中局部变量的信息。在 Code 属性的属性表中，LocalVariableTable 属性可以按照任意顺序出现。 Code 属性中的每个局部变量最多只能有一个 LocalVariableTable 属性。

  - start pc + length表示这个变量在字节码中的生命周期起始和结束的偏移位置（this生命周期从头0到结尾）

  - index就是这个变量在局部变量表中的槽位（槽位可复用）

  - name就是变量名称

  - Descriptor表示局部变量类型描述

    ![](https://v1.mykkto.cn/image/blog/2022/jvm/20220417220124.png)

- Signature 属性

  - Signature 属性是可选的定长属性，位于 ClassFile， field_info
    或 method_info结构的属性表中。在 Java 语言中，任何类、 接口、 初始化方法或成员的泛型签名如果包含了类型变量（ Type Variables） 或参数化类型（ Parameterized Types），则 Signature 属性会为它记录泛型签名信息。

- SourceFile属性

  - ![](https://v1.mykkto.cn/image/blog/2022/jvm/20220417213613.png)

- 其他属性

  - Java虚拟机中预定义的属性有20多个，这里就不一一介绍了，通过上面几个属性的介绍，只要领会其精髓，其他属性的解读也是易如反掌。





### 5、



## 四、javap

**oracle官方的反解析工具：javap**

### 1、解析字节码的作用

通过反编译生成的字节码文件，我们可以深入的了解java代码的工作机制。但是，自己分析类文件结构太麻烦了！除了使用第三方的jclasslib工具之外，oracle官方也提供了工具：javap。

javap是jdk自带的反解析工具。它的作用就是根据class字节码文件，反解析出当前类对应的code区（字节码指令）、局部变量表、异常表和代码行偏移量映射表、常量池等信息。

通过局部变量表，我们可以查看局部变量的作用域范围、所在槽位等信息，甚至可以看到槽位复用等信息。

### 2、javac -g操作

解析字节码文件得到的信息中，有些信息（如局部变量表、指令和代码行偏移量映射表、常量池中方法的参数名称等等）需要在使用javac编译成class文件时，指定参数才能输出。

比如，你直接javac xx.java，就不会在生成对应的局部变量表等信息，如果你使用`javac -g xx.java`就可以生成所有相关信息了。如果你使用的eclipse或IDEA，则默认情况下，eclipse、IDEA在编译时会帮你生成局部变量表、指令和代码行偏移量映射表等信息的。

### 3、javap的用法



javap的用法格式：
**javap <options> <classes>**

其中，classes就是你要反编译的class文件。

在命令行中直接输入javap或javap -help可以看到javap的options有如下选项：

![](https://v1.mykkto.cn/image/blog/2022/jvm/20220417224639.png)



 -help  --help  -?      输出此用法消息
 -version               版本信息，其实是当前javap所在jdk的版本信息，不是class在哪个jdk下生成的。
 -public                仅显示公共类和成员
 -protected             显示受保护的/公共类和成员
 -p  -private           显示所有类和成员
 -package               显示程序包/受保护的/公共类 和成员 (默认)
 -sysinfo               显示正在处理的类的系统信息 (路径, 大小, 日期, MD5 散列,源文件名)
 -constants             显示静态最终常量

 -s                     输出内部类型签名
 -l                     输出行号和本地变量表
 -c                     对代码进行反汇编
 -v  -verbose           输出附加信息（包括行号、本地变量表，反汇编等详细信息）

 -classpath <path>      指定查找用户类文件的位置
 -cp <path>             指定查找用户类文件的位置
 -bootclasspath <path>  覆盖引导类文件的位置



**一般常用的是-v -l -c三个选项。**

javap -l 会输出行号和本地变量表信息。
javap -c 会对当前class字节码进行反编译生成汇编代码。
javap -v classxx 除了包含-c内容外，还会输出行号、局部变量表信息、常量池等信息。





### 4、使用举例



```java
package com.kk.test;
public class JavapTest {
    private int num;
    boolean flag;
    protected char gender;
    public String info;

    public static final int COUNTS = 1;
    static{
        String url = "www.mykkto.com";
    }
    {
        info = "java";
    }
    public JavapTest(){

    }
    private JavapTest(boolean flag){
        this.flag = flag;
    }
    private void methodPrivate(){

    }
    int getNum(int i){
        return num + i;
    }
    protected char showGender(){
        return gender;
    }
    public void showInfo(){
        int i = 10;
        System.out.println(info + i);
    }
}

```



希望输出的信息比较完整的话，使用如下操作：

**javac JavapTest.java**

**javap -v -p JavapTest.class**



```bash
J:\0 大厂素材\jvm\00Test>javac JavapTest.java

J:\0 大厂素材\jvm\00Test>javap -v -p JavapTest.class
Classfile /J:/0 大厂素材/jvm/00Test/JavapTest.class
  Last modified 2022-4-17; size 1141 bytes
  MD5 checksum 16833cad2e0187d03ccf1baecaa23808
  Compiled from "JavapTest.java"
public class com.kk.test.JavapTest
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #16.#42        // java/lang/Object."<init>":()V
   #2 = String             #43            // java
   #3 = Fieldref           #15.#44        // com/kk/test/JavapTest.info:Ljava/lang/String;
   #4 = Fieldref           #15.#45        // com/kk/test/JavapTest.flag:Z
   #5 = Fieldref           #15.#46        // com/kk/test/JavapTest.num:I
   #6 = Fieldref           #15.#47        // com/kk/test/JavapTest.gender:C
   #7 = Fieldref           #48.#49        // java/lang/System.out:Ljava/io/PrintStream;
   #8 = Class              #50            // java/lang/StringBuilder
   #9 = Methodref          #8.#42         // java/lang/StringBuilder."<init>":()V
  #10 = Methodref          #8.#51         // java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
  #11 = Methodref          #8.#52         // java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
  #12 = Methodref          #8.#53         // java/lang/StringBuilder.toString:()Ljava/lang/String;
  #13 = Methodref          #54.#55        // java/io/PrintStream.println:(Ljava/lang/String;)V
  #14 = String             #56            // www.mykkto.com
  #15 = Class              #57            // com/kk/test/JavapTest
  #16 = Class              #58            // java/lang/Object
  #17 = Utf8               num
  #18 = Utf8               I
  #19 = Utf8               flag
  #20 = Utf8               Z
  #21 = Utf8               gender
  #22 = Utf8               C
  #23 = Utf8               info
  #24 = Utf8               Ljava/lang/String;
  #25 = Utf8               COUNTS
  #26 = Utf8               ConstantValue
  #27 = Integer            1
  #28 = Utf8               <init>
  #29 = Utf8               ()V
  #30 = Utf8               Code
  #31 = Utf8               LineNumberTable
  #32 = Utf8               (Z)V
  #33 = Utf8               methodPrivate
  #34 = Utf8               getNum
  #35 = Utf8               (I)I
  #36 = Utf8               showGender
  #37 = Utf8               ()C
  #38 = Utf8               showInfo
  #39 = Utf8               <clinit>
  #40 = Utf8               SourceFile
  #41 = Utf8               JavapTest.java
  #42 = NameAndType        #28:#29        // "<init>":()V
  #43 = Utf8               java
  #44 = NameAndType        #23:#24        // info:Ljava/lang/String;
  #45 = NameAndType        #19:#20        // flag:Z
  #46 = NameAndType        #17:#18        // num:I
  #47 = NameAndType        #21:#22        // gender:C
  #48 = Class              #59            // java/lang/System
  #49 = NameAndType        #60:#61        // out:Ljava/io/PrintStream;
  #50 = Utf8               java/lang/StringBuilder
  #51 = NameAndType        #62:#63        // append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
  #52 = NameAndType        #62:#64        // append:(I)Ljava/lang/StringBuilder;
  #53 = NameAndType        #65:#66        // toString:()Ljava/lang/String;
  #54 = Class              #67            // java/io/PrintStream
  #55 = NameAndType        #68:#69        // println:(Ljava/lang/String;)V
  #56 = Utf8               www.mykkto.com
  #57 = Utf8               com/kk/test/JavapTest
  #58 = Utf8               java/lang/Object
  #59 = Utf8               java/lang/System
  #60 = Utf8               out
  #61 = Utf8               Ljava/io/PrintStream;
  #62 = Utf8               append
  #63 = Utf8               (Ljava/lang/String;)Ljava/lang/StringBuilder;
  #64 = Utf8               (I)Ljava/lang/StringBuilder;
  #65 = Utf8               toString
  #66 = Utf8               ()Ljava/lang/String;
  #67 = Utf8               java/io/PrintStream
  #68 = Utf8               println
  #69 = Utf8               (Ljava/lang/String;)V
{
  private int num;
    descriptor: I
    flags: ACC_PRIVATE

  boolean flag;
    descriptor: Z
    flags:

  protected char gender;
    descriptor: C
    flags: ACC_PROTECTED

  public java.lang.String info;
    descriptor: Ljava/lang/String;
    flags: ACC_PUBLIC

  public static final int COUNTS;
    descriptor: I
    flags: ACC_PUBLIC, ACC_STATIC, ACC_FINAL
    ConstantValue: int 1

  public com.kk.test.JavapTest();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: ldc           #2                  // String java
         7: putfield      #3                  // Field info:Ljava/lang/String;
        10: return
      LineNumberTable:
        line 16: 0
        line 14: 4
        line 18: 10

  private com.kk.test.JavapTest(boolean);
    descriptor: (Z)V
    flags: ACC_PRIVATE
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: ldc           #2                  // String java
         7: putfield      #3                  // Field info:Ljava/lang/String;
        10: aload_0
        11: iload_1
        12: putfield      #4                  // Field flag:Z
        15: return
      LineNumberTable:
        line 19: 0
        line 14: 4
        line 20: 10
        line 21: 15

  private void methodPrivate();
    descriptor: ()V
    flags: ACC_PRIVATE
    Code:
      stack=0, locals=1, args_size=1
         0: return
      LineNumberTable:
        line 24: 0

  int getNum(int);
    descriptor: (I)I
    flags:
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: getfield      #5                  // Field num:I
         4: iload_1
         5: iadd
         6: ireturn
      LineNumberTable:
        line 26: 0

  protected char showGender();
    descriptor: ()C
    flags: ACC_PROTECTED
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: getfield      #6                  // Field gender:C
         4: ireturn
      LineNumberTable:
        line 29: 0

  public void showInfo();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=3, locals=2, args_size=1
         0: bipush        10
         2: istore_1
         3: getstatic     #7                  // Field java/lang/System.out:Ljava/io/PrintStream;
         6: new           #8                  // class java/lang/StringBuilder
         9: dup
        10: invokespecial #9                  // Method java/lang/StringBuilder."<init>":()V
        13: aload_0
        14: getfield      #3                  // Field info:Ljava/lang/String;
        17: invokevirtual #10                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        20: iload_1
        21: invokevirtual #11                 // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
        24: invokevirtual #12                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
        27: invokevirtual #13                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        30: return
      LineNumberTable:
        line 32: 0
        line 33: 3
        line 34: 30

  static {};
    descriptor: ()V
    flags: ACC_STATIC
    Code:
      stack=1, locals=1, args_size=0
         0: ldc           #14                 // String www.mykkto.com
         2: astore_0
         3: return
      LineNumberTable:
        line 11: 0
        line 12: 3
}
SourceFile: "JavapTest.java"

J:\0 大厂素材\jvm\00Test>
```





### 5、总结

1、通过javap命令可以查看一个java类反汇编得到的Class文件版本号、常量池、访问标识、变量表、指令代码行号表等等信息。不显示类索引、父类索引、接口索引集合、<clinit>()、<init>()等结构

2、通过对前面例子代码反汇编文件的简单分析，可以发现，一个方法的执行通常会涉及下面几块内存的操作：
（1）java栈中：局部变量表、操作数栈。
（2）java堆。通过对象的地址引用去操作。
（3）常量池。
（4）其他如帧数据区、方法区的剩余部分等情况，测试中没有显示出来，这里说明一下。

3、平常，我们比较关注的是java类中每个方法的反汇编中的指令操作过程，这些指令都是顺序执行的，可以参考官方文档查看每个指令的含义，很简单：
https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-6.html



## 五、字节码指令集与解析概述



Java字节码对于虚拟机，就好像汇编语言对于计算机，属于基本执行指令。

Java 虚拟机的指令由一个字节长度的、代表着某种特定操作含义的数字（称为操作码，Opcode）以及跟随其后的零至多个代表此操作所需参数（称为操作数，Operands）而构成。由于 Java 虚拟机采用面向操作数栈而不是寄存器的结构，所以大多数的指令都不包含操作数，只有一个操作码。

由于限制了 Java 虚拟机操作码的长度为一个字节（即 0～255），这意味着指令集的操作码总数不可能超过 256 条。

官方文档：https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html

熟悉虚拟机的指令对于动态字节码生成、反编译Class文件、Class文件修补都有着非常重要的价值。因此，阅读字节码作为了解 Java 虚拟机的基础技能，需要熟练掌握常见指令。

 



### 1、字节码与数据类型

在Java虚拟机的指令集中，大多数的指令都包含了其操作所对应的数据类型信息。例如，iload指令用于从局部变量表中加载int型的数据到操作数栈中，而fload指令加载的则是float类型的数据。

对于大部分与数据类型相关的字节码指令，它们的操作码助记符中都有特殊的字符来表明专门为哪种数据类型服务：

- i代表对int类型的数据操作
- l代表long类型的数据操作
- s代表short类型的数据操作
- b代表byte类型的数据操作
- c代表char类型的数据操作
- f代表float类型的数据操作
- d代表double类型的数据操作

也有一些指令的助记符中没有明确地指明操作类型的字母，如arraylength指令，它没有代表数据类型的特殊字符，但操作数永远只能是一个数组类型的对象。

还有另外一些指令，如无条件跳转指令goto则是与数据类型无关的。

大部分的指令都没有支持整数类型byte、char和short，甚至没有任何指令支持boolean类型。编译器会在编译期或运行期将byte和short类型的数据带符号扩展（Sign-Extend）为相应的int类型数据，将boolean和char类型数据零位扩展（Zero-Extend）为相应的int类型数据。与之类似，在处理boolean、byte、short和char类型的数组时，也会转换为使用对应的int类型的字节码指令来处理。因此，大多数对于boolean、byte、short和char类型数据的操作，实际上都是使用相应的int类型作为运算类型。

```java
byte b1 = 12;
short s1 = 10;
int i = b1 + s1;
```





### 2、指令分类

由于完全介绍和学习这些指令需要花费大量时间。为了让大家能够更快地熟悉和了解这些基本指令，这里将JVM中的字节码指令集按用途大致分成 9 类。

- 加载与存储指令
- 算术指令
- 类型转换指令
- 对象的创建与访问指令
- 方法调用与返回指令
- 操作数栈管理指令
- 控制转移指令
- 异常处理指令
- 同步控制指令

(说在前面)在做值相关操作时：

- 一个指令，可以从局部变量表、常量池、堆中对象、方法调用、系统调用中等取得数据，这些数据（可能是值，可能是对象的引用）被压入操作数栈。
- 一个指令，也可以从操作数栈中取出一到多个值（pop多次），完成赋值、加减乘除、方法传参、系统调用等等操作。



## 六、★ 字节码指令

### 1、加载与存储指令

### 2、算术指令

### 3、类型转换指令

### 4、对象的创建与访问指令

### 5、方法调用与返回指令

### 6、操作数栈管理指令

### 7、控制转移指令

### 8、异常处理指令

### 9、同步控制指令