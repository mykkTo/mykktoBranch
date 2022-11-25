---
title: Netty-实用篇
author: mykk
top: false
cover: false
toc: true
mathjax: false
summary: 本篇会有，NIO，再到 Netty
tags:
  - netty
  - nio
  - 网络编程
categories:
  - 内力篇
abbrlink: bb8f1edf
reprintPolicy: cc_by
date: 2022-08-21 13:22:33
coverImg:
img:
password:
---





## 源代码位置

https://gitee.com/TK_LIMR/springcloud2021To2021

![](https://v1.mykkto.cn/image/blog/2022/netty/202208201706838.png)









## 目录：

- 网络编程
  - 三次握手
  - 基础后面会在其他文章更新
- nio
  - 三大组件
  - ByteBuffer详解
  - 文件编程
  - 网络编程
  - nio vs bio vs aio
- netty
  - 入门
  - 进阶
  - 优化
  - 实战
  - 源码





![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202208201259923.png)









## Ⅰ、NIO

## 一、三大组件

### 1、Channel & Buffer

channel 有一点类似于 stream，它就是读写数据的**双向通道**，可以从 channel 将数据读入 buffer，也可以将 buffer 的数据写入 channel，而之前的 stream 要么是输入，要么是输出，channel 比 stream 更为底层



![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202208201612733.png)



常见的 Channel 有

- FileChannel
- DatagramChannel
- SocketChannel
- ServerSocketChannel



buffer 则用来缓冲读写数据，常见的 buffer 有

- ByteBuffer
  - MappedByteBuffer
  - DirectByteBuffer
  - HeapByteBuffer
- ShortBuffer
- IntBuffer
- LongBuffer
- FloatBuffer
- DoubleBuffer
- CharBuffer

### 2、Selector

selector 单从字面意思不好理解，需要结合服务器的设计演化来理解它的用途



#### 2-1、多线程版设计



![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202208201636971.png)



##### ⚠️ 多线程版缺点

- 内存占用高
- 线程上下文切换成本高
- 只适合连接数少的场景



#### 2-2、线程池版设计

![](https://v1.mykkto.cn/image/blog/2022/bigdata/flink/202208201639068.png)



##### ⚠️ 线程池版缺点



- 阻塞模式下，线程仅能处理一个 socket 连接
- 仅适合短连接场景



#### 2-3、selector 版设计

selector 的作用就是配合一个线程来管理多个 channel，获取这些 channel 上发生的事件，这些 channel 工作在非阻塞模式下，不会让线程吊死在一个 channel 上。适合连接数特别多，但流量低的场景（low traffic）



![](https://v1.mykkto.cn/image/blog/2022/netty/202208201647534.png)



调用 selector 的 select() 会阻塞直到 channel 发生了读写就绪事件，这些事件发生，select 方法就会返回这些事件交给 thread 来处理







## 二、ByteBuffer详解

有一普通文本文件 data.txt，内容为

```txt
1234567890abcd
```



使用 FileChannel 来读取文件内容

```java
package com.kk.netty.nio;

import lombok.extern.slf4j.Slf4j;

import java.io.IOException;
import java.io.RandomAccessFile;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

@Slf4j
public class ChannelDemo1 {
    // 
    public static void main(String[] args) {
        /**
         * try () 1.7的语法糖，方便关闭 RandomAccessFile 资源
         * FileChannel
         * 1、输入输出流，RandomAccessFile
         */
        try (RandomAccessFile file = new RandomAccessFile ("boot_netty/helloword/data.txt", "rw")) {
            FileChannel channel = file.getChannel ( );
            ByteBuffer buffer = ByteBuffer.allocate (10);
            do {
                // 准备缓冲区，向 buffer 写入
                int len = channel.read (buffer);
                log.debug ("读到字节数：{}", len);
                if (len == -1) {// 没有内容了
                    break;
                }
                // 切换 buffer 读模式
                // 翻转缓冲区，为写做准备
                buffer.flip ( );
                while (buffer.hasRemaining ( )) {// 是否还有剩余未读数据
                    byte b = buffer.get ( );
                    log.debug ("{}", (char) b);
                }
                // 切换 buffer 写模式
                // 清楚缓冲区，为读做准备
                buffer.clear ( );
            } while (true);
        } catch (IOException e) {
            e.printStackTrace ( );
        }
    }

}

```

输出

> 19:02:52.712 [main] DEBUG com.kk.netty.nio.ChannelDemo1 - 读到字节数：10
> 19:02:52.717 [main] DEBUG com.kk.netty.nio.ChannelDemo1 - 1
> 19:02:52.717 [main] DEBUG com.kk.netty.nio.ChannelDemo1 - 2
> 19:02:52.717 [main] DEBUG com.kk.netty.nio.ChannelDemo1 - 3
> 19:02:52.718 [main] DEBUG com.kk.netty.nio.ChannelDemo1 - 4
> 19:02:52.718 [main] DEBUG com.kk.netty.nio.ChannelDemo1 - 5
> 19:02:52.718 [main] DEBUG com.kk.netty.nio.ChannelDemo1 - 6
> 19:02:52.718 [main] DEBUG com.kk.netty.nio.ChannelDemo1 - 7
> 19:02:52.718 [main] DEBUG com.kk.netty.nio.ChannelDemo1 - 8
> 19:02:52.718 [main] DEBUG com.kk.netty.nio.ChannelDemo1 - 9
> 19:02:52.718 [main] DEBUG com.kk.netty.nio.ChannelDemo1 - 0
> 19:02:52.718 [main] DEBUG com.kk.netty.nio.ChannelDemo1 - 读到字节数：6
> 19:02:52.718 [main] DEBUG com.kk.netty.nio.ChannelDemo1 - a
> 19:02:52.718 [main] DEBUG com.kk.netty.nio.ChannelDemo1 - b
> 19:02:52.718 [main] DEBUG com.kk.netty.nio.ChannelDemo1 - c
> 19:02:52.718 [main] DEBUG com.kk.netty.nio.ChannelDemo1 - d
> 19:02:52.718 [main] DEBUG com.kk.netty.nio.ChannelDemo1 - 
> 19:02:52.718 [main] DEBUG com.kk.netty.nio.ChannelDemo1 - 
>
> </br>
>
> 19:02:52.718 [main] DEBUG com.kk.netty.nio.ChannelDemo1 - 读到字节数：-1



### 1、ByteBuffer 步骤



1. 向 buffer 写入数据，例如调用 channel.read(buffer)
2. 调用 flip() 切换至**读模式**
3. 从 buffer 读取数据，例如调用 buffer.get()
4. 调用 clear() 或 compact() 切换至**写模式**
5. 重复 1~4 步骤





### 2、ByteBuffer 结构

ByteBuffer 有以下重要属性

- capacity 容量
- position 写入位置，移动的游标
- limit  写入或者读取限制（根据读写模式切换决定）



##### 1、开始

![](https://v1.mykkto.cn/image/blog/2022/netty/202208201929345.png)



##### 2、写模式下，position 是写入位置，limit 等于容量，下图表示写入了 4 个字节后的状态

![](https://v1.mykkto.cn/image/blog/2022/netty/202208201933170.png)



##### 3、flip 动作发生后，position 切换为读取位置，limit 切换为读取限制

![](https://v1.mykkto.cn/image/blog/2022/netty/202208201934450.png)



##### 4、读取 4 个字节后，状态

![](https://v1.mykkto.cn/image/blog/2022/netty/202208201937372.png)



##### 5、clear 动作发生后，状态

![](https://v1.mykkto.cn/image/blog/2022/netty/202208201937628.png)



##### 6、compact 方法，是把未读完的部分向前压缩，然后切换至写模式

![](https://v1.mykkto.cn/image/blog/2022/netty/202208201938152.png)



#### 💡 调试工具类

```java
package com.kk.netty.nio;


import io.netty.util.internal.StringUtil;

import java.nio.ByteBuffer;

import static io.netty.util.internal.MathUtil.isOutOfBounds;
import static io.netty.util.internal.StringUtil.NEWLINE;

public class ByteBufferUtil {
    private static final char[] BYTE2CHAR = new char[256];
    private static final char[] HEXDUMP_TABLE = new char[256 * 4];
    private static final String[] HEXPADDING = new String[16];
    private static final String[] HEXDUMP_ROWPREFIXES = new String[65536 >>> 4];
    private static final String[] BYTE2HEX = new String[256];
    private static final String[] BYTEPADDING = new String[16];

    static {
        final char[] DIGITS = "0123456789abcdef".toCharArray ( );
        for (int i = 0; i < 256; i++) {
            HEXDUMP_TABLE[i << 1] = DIGITS[i >>> 4 & 0x0F];
            HEXDUMP_TABLE[(i << 1) + 1] = DIGITS[i & 0x0F];
        }

        int i;

        // Generate the lookup table for hex dump paddings
        for (i = 0; i < HEXPADDING.length; i++) {
            int padding = HEXPADDING.length - i;
            StringBuilder buf = new StringBuilder (padding * 3);
            for (int j = 0; j < padding; j++) {
                buf.append ("   ");
            }
            HEXPADDING[i] = buf.toString ( );
        }

        // Generate the lookup table for the start-offset header in each row (up to 64KiB).
        for (i = 0; i < HEXDUMP_ROWPREFIXES.length; i++) {
            StringBuilder buf = new StringBuilder (12);
            buf.append (NEWLINE);
            buf.append (Long.toHexString (i << 4 & 0xFFFFFFFFL | 0x100000000L));
            buf.setCharAt (buf.length ( ) - 9, '|');
            buf.append ('|');
            HEXDUMP_ROWPREFIXES[i] = buf.toString ( );
        }

        // Generate the lookup table for byte-to-hex-dump conversion
        for (i = 0; i < BYTE2HEX.length; i++) {
            BYTE2HEX[i] = ' ' + StringUtil.byteToHexStringPadded (i);
        }

        // Generate the lookup table for byte dump paddings
        for (i = 0; i < BYTEPADDING.length; i++) {
            int padding = BYTEPADDING.length - i;
            StringBuilder buf = new StringBuilder (padding);
            for (int j = 0; j < padding; j++) {
                buf.append (' ');
            }
            BYTEPADDING[i] = buf.toString ( );
        }

        // Generate the lookup table for byte-to-char conversion
        for (i = 0; i < BYTE2CHAR.length; i++) {
            if (i <= 0x1f || i >= 0x7f) {
                BYTE2CHAR[i] = '.';
            } else {
                BYTE2CHAR[i] = (char) i;
            }
        }
    }

    /**
     * 打印所有内容
     *
     * @param buffer
     */
    public static void debugAll(ByteBuffer buffer) {
        int oldlimit = buffer.limit ( );
        buffer.limit (buffer.capacity ( ));
        StringBuilder origin = new StringBuilder (256);
        appendPrettyHexDump (origin, buffer, 0, buffer.capacity ( ));
        System.out.println ("+--------+-------------------- all ------------------------+----------------+");
        System.out.printf ("position: [%d], limit: [%d]\n", buffer.position ( ), oldlimit);
        System.out.println (origin);
        buffer.limit (oldlimit);
    }

    /**
     * 打印可读取内容
     *
     * @param buffer
     */
    public static void debugRead(ByteBuffer buffer) {
        StringBuilder builder = new StringBuilder (256);
        appendPrettyHexDump (builder, buffer, buffer.position ( ), buffer.limit ( ) - buffer.position ( ));
        System.out.println ("+--------+-------------------- read -----------------------+----------------+");
        System.out.printf ("position: [%d], limit: [%d]\n", buffer.position ( ), buffer.limit ( ));
        System.out.println (builder);
    }

    private static void appendPrettyHexDump(StringBuilder dump, ByteBuffer buf, int offset, int length) {
        if (isOutOfBounds (offset, length, buf.capacity ( ))) {
            throw new IndexOutOfBoundsException (
                    "expected: " + "0 <= offset(" + offset + ") <= offset + length(" + length
                            + ") <= " + "buf.capacity(" + buf.capacity ( ) + ')');
        }
        if (length == 0) {
            return;
        }
        dump.append (
                "         +-------------------------------------------------+" +
                        NEWLINE + "         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |" +
                        NEWLINE + "+--------+-------------------------------------------------+----------------+");

        final int startIndex = offset;
        final int fullRows = length >>> 4;
        final int remainder = length & 0xF;

        // Dump the rows which have 16 bytes.
        for (int row = 0; row < fullRows; row++) {
            int rowStartIndex = (row << 4) + startIndex;

            // Per-row prefix.
            appendHexDumpRowPrefix (dump, row, rowStartIndex);

            // Hex dump
            int rowEndIndex = rowStartIndex + 16;
            for (int j = rowStartIndex; j < rowEndIndex; j++) {
                dump.append (BYTE2HEX[getUnsignedByte (buf, j)]);
            }
            dump.append (" |");

            // ASCII dump
            for (int j = rowStartIndex; j < rowEndIndex; j++) {
                dump.append (BYTE2CHAR[getUnsignedByte (buf, j)]);
            }
            dump.append ('|');
        }

        // Dump the last row which has less than 16 bytes.
        if (remainder != 0) {
            int rowStartIndex = (fullRows << 4) + startIndex;
            appendHexDumpRowPrefix (dump, fullRows, rowStartIndex);

            // Hex dump
            int rowEndIndex = rowStartIndex + remainder;
            for (int j = rowStartIndex; j < rowEndIndex; j++) {
                dump.append (BYTE2HEX[getUnsignedByte (buf, j)]);
            }
            dump.append (HEXPADDING[remainder]);
            dump.append (" |");

            // Ascii dump
            for (int j = rowStartIndex; j < rowEndIndex; j++) {
                dump.append (BYTE2CHAR[getUnsignedByte (buf, j)]);
            }
            dump.append (BYTEPADDING[remainder]);
            dump.append ('|');
        }

        dump.append (NEWLINE +
                "+--------+-------------------------------------------------+----------------+");
    }

    private static void appendHexDumpRowPrefix(StringBuilder dump, int row, int rowStartIndex) {
        if (row < HEXDUMP_ROWPREFIXES.length) {
            dump.append (HEXDUMP_ROWPREFIXES[row]);
        } else {
            dump.append (NEWLINE);
            dump.append (Long.toHexString (rowStartIndex & 0xFFFFFFFFL | 0x100000000L));
            dump.setCharAt (dump.length ( ) - 9, '|');
            dump.append ('|');
        }
    }

    public static short getUnsignedByte(ByteBuffer buffer, int index) {
        return (short) (buffer.get (index) & 0xFF);
    }
}

```



#### 调用案例

```java
package com.kk.netty.nio.demo;

import java.nio.ByteBuffer;

import static com.kk.netty.nio.ByteBufferUtil.debugAll;

public class TestByteBufferReadWrite {
    public static void main(String[] args) {
        ByteBuffer buffer = ByteBuffer.allocate (10);
        System.out.println ("1、先插入 a ，再插入 b c d" );
        // ASCLL码（十进制） 97 = 十六进制 0x61 =  a
        buffer.put ((byte)0x61);// 写入一个字符 a
        debugAll(buffer);// 调试打印
        buffer.put (new byte[]{0x62,0x63,0x64});// 写入字符 b c d
        debugAll(buffer);// 调试打印

        System.out.println (" 2、切换 读模式，取一个字符" );
        // 在没有切换为 读 模式的时候，直接拿是拿不到的
        //System.out.println (buffer.get ( ));
        buffer.flip ();// 切换读模式
        System.out.println (buffer.get ( ));// 取出一个，还剩3个
        System.out.println (buffer.get ( ));// 取出一个，还剩2个
        debugAll(buffer);// 调试打印

        System.out.println ("3、未读取的往前移(压缩),第3、4位会遗留二个63、64是因为没有清零，然后后面插入不影响，会从已有的后面插入" );
        buffer.compact ();
        debugAll(buffer);// 调试打印

        System.out.println ("4、插入两个字符，从已有的后面插入" );
        buffer.put (new byte[]{0x65,0x6f});
        debugAll(buffer);// 调试打印
        
    }
}

```





### 3、ByteBuffer 常见 API

- 分配空间
- 向 buffer 写入数据
- 从 buffer 读取数据
- mark 和 reset
- 字符串与 ByteBuffer 互转
- Buffer 的线程安全



#### 3-1、分配空间

可以使用 allocate 方法为 ByteBuffer 分配空间，其它 buffer 类也有该方法

> Bytebuffer buf = ByteBuffer.allocate(16);



#### 3-2、buffer 写入数据

有两种办法

- 调用 channel 的 read 方法
- 调用 buffer 自己的 put 方法



```java
int readBytes = channel.read(buf);
```

和

```java
buf.put((byte)127);
```





#### 3-3、buffer 读取数据

同样有两种办法

- 调用 channel 的 write 方法
- 调用 buffer 自己的 get 方法

```java
int writeBytes = channel.write(buf);
```

和

```java
byte b = buf.get();
```

get 方法会让 position 读指针向后走，如果想重复读取数据

- 可以调用 rewind 方法将 position 重新置为 0
- 或者调用 get(int i) 方法获取索引 i 的内容，它不会移动读指针



#### 3-4、mark 和 reset

mark 是在读取时，做一个标记，即使 position 改变，只要调用 reset 就能回到 mark 的位置

> **注意**
>
> rewind 和 flip 都会清除 mark 位置



#### 



#### 3-5、字符串与 ByteBuffer 互转

#### 

```java
package com.kk.netty.nio.demo;

import java.nio.ByteBuffer;
import java.nio.CharBuffer;
import java.nio.charset.Charset;
import java.nio.charset.StandardCharsets;

import static com.kk.netty.nio.ByteBufferUtil.debugAll;

public class TestButeBufferApi {
    public static void main(String[] args) {
        ByteBuffer buffer1 = StandardCharsets.UTF_8.encode ("你好");
        ByteBuffer buffer2 = Charset.forName ("utf-8").encode ("你好");

        debugAll (buffer1);
        debugAll (buffer2);

        CharBuffer buffer3 = StandardCharsets.UTF_8.decode (buffer1);
        System.out.println (buffer3.getClass ( ));
        System.out.println (buffer3.toString ( ));
    }
}

```

输出

```
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| e4 bd a0 e5 a5 bd                               |......          |
+--------+-------------------------------------------------+----------------+
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| e4 bd a0 e5 a5 bd                               |......          |
+--------+-------------------------------------------------+----------------+
class java.nio.HeapCharBuffer
你好
```





#### 3-6、Buffer 的线程安全

> Buffer 是**非线程安全的**



### 4、Scattering Reads

#### 1、分散集中读

分散读取，有一个文本文件 3parts.txt

```
onetwothree
```

使用如下方式读取，可以将数据填充至多个 buffer

```java
package com.kk.netty.nio.demo;

import java.io.IOException;
import java.io.RandomAccessFile;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

import static com.kk.netty.nio.ByteBufferUtil.debugAll;

public class TestScatteringReads {
    public static void main(String[] args) {
        try (RandomAccessFile file = new RandomAccessFile ("boot_netty/helloword/3parts.txt", "rw")) {
            FileChannel channel = file.getChannel ( );
            ByteBuffer a = ByteBuffer.allocate (3);
            ByteBuffer b = ByteBuffer.allocate (3);
            ByteBuffer c = ByteBuffer.allocate (5);
            channel.read (new ByteBuffer[]{a, b, c});
            a.flip ( );
            b.flip ( );
            c.flip ( );
            debugAll (a);
            debugAll (b);
            debugAll (c);
        } catch (IOException e) {
            e.printStackTrace ( );
        }
    }
}

```

结果

```
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 6f 6e 65                                        |one             |
+--------+-------------------------------------------------+----------------+
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 74 77 6f                                        |two             |
+--------+-------------------------------------------------+----------------+
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 74 68 72 65 65                                  |three           |
+--------+-------------------------------------------------+----------------+
```





#### 2、分散集中写

```java
package com.kk.netty.nio.demo;

import java.io.IOException;
import java.io.RandomAccessFile;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;
import java.nio.charset.StandardCharsets;

/**
 * 分散集中写
 */
public class TestScatteringWrites {
    public static void main(String[] args) {
        ByteBuffer b1 = StandardCharsets.UTF_8.encode ("hello");
        ByteBuffer b2 = StandardCharsets.UTF_8.encode ("world");
        ByteBuffer b3 = StandardCharsets.UTF_8.encode ("你好，世界");
        try (FileChannel file = new RandomAccessFile ("boot_netty/helloword/2parts.txt", "rw").getChannel ( )) {
            file.write (new ByteBuffer[]{b1, b2, b3});
        } catch (IOException e) {
            e.printStackTrace ( );
        }
    }
}

```





### 5、Gathering Writes

网络上有多条数据发送给服务端，数据之间使用 \n 进行分隔
但由于某种原因这些数据在接收时，被进行了重新组合，例如原始数据有3条为

- Hello,world\n
- I'm zhangsan\n
- How are you?\n

变成了下面的两个 byteBuffer (黏包，半包)

- Hello,world\nI'm zhangsan\nHo
- w are you?\n

现在要求你编写程序，将错乱的数据恢复成原始的按 \n 分隔的数据

```java
package com.kk.netty.nio.demo;

import java.nio.ByteBuffer;

import static com.kk.netty.nio.ByteBufferUtil.debugAll;

public class TestGatheringWrites {
    public static void main(String[] args) {
        ByteBuffer source = ByteBuffer.allocate (32);
        //                     11            24
        source.put ("Hello,world\nI'm zhangsan\nHo".getBytes ( ));
        split (source);

        source.put ("w are you?\nhaha!\n".getBytes ( ));
        split (source);
    }

    private static void split(ByteBuffer source) {
        source.flip ( );
        // source.limit ( ) 缓冲区容量长度
        for (int i = 0; i < source.limit ( ); i++) {
            // 找到一条完整信息
            if (source.get (i) == '\n') {
                System.out.println (i);
                // 计算每个 \n 词组的长度（换行符号 +1 - 游标起始位置）
                int length = i + 1 - source.position ( );
                // 将这条完整信息存入新的 bytebuffer
                ByteBuffer target = ByteBuffer.allocate (length);
                // 从 source 读，向 target 写
                for (int j = 0; j < length; j++) {
                    target.put (source.get ());
                }
                debugAll (target);
            }
        }
        // 未读的往前移动
        source.compact ( );
    }
}

```





## 三、文件编程

### 1、FileChannel



#### 1、⚠️ FileChannel 工作模式

> FileChannel 只能工作在阻塞模式下



#### 2、获取

不能直接打开 FileChannel，必须通过 FileInputStream、FileOutputStream 或者 RandomAccessFile 来获取 FileChannel，它们都有 getChannel 方法

- 通过 FileInputStream 获取的 channel 只能读
- 通过 FileOutputStream 获取的 channel 只能写
- 通过 RandomAccessFile 是否能读写根据构造 RandomAccessFile 时的读写模式决定



#### 3、读取

会从 channel 读取数据填充 ByteBuffer，返回值表示读到了多少字节，-1 表示到达了文件的末尾

```java
int readBytes = channel.read(buffer);
```



#### 4、写入

写入的正确姿势如下， SocketChannel

```java
ByteBuffer buffer = ...;
buffer.put(...); // 存入数据
buffer.flip();   // 切换读模式

while(buffer.hasRemaining()) {
    channel.write(buffer);
}
```

在 while 中调用 channel.write 是因为 write 方法并不能保证一次将 buffer 中的内容全部写入 channel



#### 5、位置

获取当前位置

```java
long pos = channel.position();
```

设置当前位置

```java
long newPos = ...;
channel.position(newPos);
```

设置当前位置时，如果设置为文件的末尾

- 这时读取会返回 -1 
- 这时写入，会追加内容，但要注意如果 position 超过了文件末尾，再写入时在新内容和原末尾之间会有空洞（00）



#### 6、大小

使用 size 方法获取文件的大小



#### 7、强制写入

操作系统出于性能的考虑，会将数据缓存，不是立刻写入磁盘。可以调用 force(true)  方法将文件内容和元数据（文件的权限等信息）立刻写入磁盘





### 2、两个 Channel 传输数据

基础版：限制2g

```java
package com.kk.netty.nio.file;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.channels.FileChannel;

public class TestFileChannel {
    public static void main(String[] args) {
        String FROM = "boot_netty/helloword/data.txt";
        String TO = "boot_netty/helloword/to.txt";
        long start = System.nanoTime ( );
        try (
                FileChannel from = new FileInputStream (FROM).getChannel ( );
                FileChannel to = new FileOutputStream (TO).getChannel ( );
        ) {
            // 数据 from 传输到  to*（就是一个拷贝过程）
            // 效率会比传统的原生api 高，底层会用操作系统的零拷贝进行优化
            from.transferTo (0, from.size ( ), to);
        } catch (IOException e) {
            e.printStackTrace ( );
        }
        long end = System.nanoTime ( );
        System.out.println ("transferTo 用时：" + (end - start) / 1000_000.0);
    }
}

```

优化版：循环插入，解决2g限制

```java
package com.kk.netty.nio.file;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.channels.FileChannel;

public class TestFileChannel {
    public static void main(String[] args) {
        String FROM = "boot_netty/helloword/data.txt";
        String TO = "boot_netty/helloword/to.txt";
        long start = System.nanoTime ( );
        try (
                FileChannel from = new FileInputStream (FROM).getChannel ( );
                FileChannel to = new FileOutputStream (TO).getChannel ( );
        ) {
            // 数据 from 传输到  to*（就是一个拷贝过程）
            // 效率会比传统的原生api 高，底层会用操作系统的零拷贝进行优化
            long size = from.size ( );
            // left 变量代表还剩余多少字节
            for (long left = size; left > 0; ) {
                System.out.println ("position:" + (size - left) + " left:" + left);
                left -= from.transferTo (size - left, left, to);
            }
        } catch (IOException e) {
            e.printStackTrace ( );
        }
        long end = System.nanoTime ( );
        System.out.println ("transferTo 用时：" + (end - start) / 1000_000.0);
    }
}

```





### 3、Path

jdk7 引入了 Path 和 Paths 类

- Path 用来表示文件路径
- Paths 是工具类，用来获取 Path 实例



```java
Path source = Paths.get("1.txt"); // 相对路径 使用 user.dir 环境变量来定位 1.txt

Path source = Paths.get("d:\\1.txt"); // 绝对路径 代表了  d:\1.txt

Path source = Paths.get("d:/1.txt"); // 绝对路径 同样代表了  d:\1.txt

Path projects = Paths.get("d:\\data", "projects"); // 代表了  d:\data\projects
```



- `.` 代表了当前路径
- `..` 代表了上一级路径

例如目录结构如下

```
d:
	|- data
		|- projects
			|- a
			|- b
```

代码

```java
Path path = Paths.get("d:\\data\\projects\\a\\..\\b");
System.out.println(path);
System.out.println(path.normalize()); // 正常化路径
```

会输出

```
d:\data\projects\a\..\b
d:\data\projects\b
```





### 4、Files

##### 检查文件是否存在

```java
Path path = Paths.get("helloword/data.txt");
System.out.println(Files.exists(path));
```



##### 创建一级目录

```java
Path path = Paths.get("helloword/d1");
Files.createDirectory(path);
```

- 如果目录已存在，会抛异常 FileAlreadyExistsException
- 不能一次创建多级目录，否则会抛异常 NoSuchFileException



##### 创建多级目录用

```java
Path path = Paths.get("helloword/d1/d2");
Files.createDirectories(path);
```



##### 拷贝文件

```java
Path source = Paths.get("helloword/data.txt");
Path target = Paths.get("helloword/target.txt");

Files.copy(source, target);
```

- 如果文件已存在，会抛异常 FileAlreadyExistsException

如果希望用 source 覆盖掉 target，需要用 StandardCopyOption 来控制

```java
Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
```



##### 移动文件

```java
Path source = Paths.get("helloword/data.txt");
Path target = Paths.get("helloword/data.txt");

Files.move(source, target, StandardCopyOption.ATOMIC_MOVE);
```

- StandardCopyOption.ATOMIC_MOVE 保证文件移动的原子性



##### 删除文件

```java
Path target = Paths.get("helloword/target.txt");

Files.delete(target);
```

- 如果文件不存在，会抛异常 NoSuchFileException



##### 删除目录

```java
Path target = Paths.get("helloword/d1");

Files.delete(target);
```

- 如果目录还有内容，会抛异常 DirectoryNotEmptyException



##### 遍历目录文件

设计模式：访问者模式

```java
    public static void main(String[] args) throws IOException {
        // 目录个数
        AtomicInteger direCount = new AtomicInteger ( );
        // 文件个数
        AtomicInteger fileCount = new AtomicInteger ( );
        // 目录对象
        Path path = Paths.get ("C:\\Program Files\\Java\\jdk1.8.0_202");
        // 设计模式：访问者模式
        Files.walkFileTree (path,new SimpleFileVisitor<Path>(){
            // 获取目录
            @Override
            public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs) throws IOException {
                System.out.println (dir );
                direCount.incrementAndGet ();
                return super.preVisitDirectory (dir, attrs);
            }
            // 获取文件
            @Override
            public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
                System.out.println (file );
                fileCount.incrementAndGet ();
                return super.visitFile (file, attrs);
            }
        }  );
        System.out.println(direCount); // 133
        System.out.println(fileCount); // 1479
    }
```

统计 jar 的数目

```java
 public static void main(String[] args) throws IOException {
        // 文件个数
        AtomicInteger fileCount = new AtomicInteger ( );
        // 目录对象
        Path path = Paths.get ("C:\\Program Files\\Java\\jdk1.8.0_202");
        // 设计模式：访问者模式
        Files.walkFileTree (path,new SimpleFileVisitor<Path> (){
            @Override
            public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
                // 根据文件后缀筛选
                if (file.toFile ().getName ().endsWith (".jar")){
                    fileCount.incrementAndGet ();
                }
                return super.visitFile (file, attrs);
            }
        });
        System.out.println(fileCount); // 705
    }
```



删除多级目录

```java
public class TestRemoveFile {
    public static void main(String[] args) throws IOException {
        Path path = Paths.get ("j:\\test");
        Files.walkFileTree (path,new SimpleFileVisitor<Path> (){
            // 先删除文件
            @Override
            public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
                Files.delete (file);
                return super.visitFile (file, attrs);
            }

            // 再删除目录
            @Override
            public FileVisitResult postVisitDirectory(Path dir, IOException exc) throws IOException {
                Files.delete (dir);
                return super.postVisitDirectory (dir, exc);
            }
        });
    }
}
```

⚠️ 删除很危险

> 删除是危险操作，确保要递归删除的文件夹没有重要内容



拷贝多级目录

```java
public class TestCopyMultDirectory {
    public static void main(String[] args) throws IOException {
        String source = "j:\\test";
        String target = "j:\\testTemp";
        long start = System.currentTimeMillis ();

        Files.walk (Paths.get (source)).forEach (path -> {
            try {
                String targetName = path.toString ( ).replace (source, target);
                // 如果是目录
                if (Files.isDirectory (path)){
                    Files.createDirectories (Paths.get (targetName));
                }
                // 如果是文件
                else if (Files.isRegularFile (path)){
                    Files.copy (path,Paths.get (targetName));
                }
            } catch (Exception e) {
                e.printStackTrace ( );
            }
        });

        long end = System.currentTimeMillis();
        System.out.println(end - start);
    }
}

```







## 四、网络编程

### 1、非阻塞 vs 阻塞

#### 1.1、阻塞

- 阻塞模式下，相关方法都会导致线程暂停
  - ServerSocketChannel.accept 会在没有连接建立时让线程暂停
  - SocketChannel.read 会在没有数据可读时让线程暂停
  - 阻塞的表现其实就是线程暂停了，暂停期间不会占用 cpu，但线程相当于闲置
- 单线程下，阻塞方法之间相互影响，几乎不能正常工作，需要多线程支持
- 但多线程下，有新的问题，体现在以下方面
  - 32 位 jvm 一个线程 320k，64 位 jvm 一个线程 1024k，如果连接数过多，必然导致 OOM，并且线程太多，反而会因为频繁上下文切换导致性能降低
  - 可以采用线程池技术来减少线程数和线程上下文切换，但治标不治本，如果有很多连接建立，但长时间 inactive，会阻塞线程池中所有线程，因此不适合长连接，只适合短连接
- 阻塞在哪里(人话)
  - 1、连接的阻塞（accept）
  - 2、写入数据的阻塞（read）
  - 3、accept 的时候不能 read ,read 时候不能 accept ，这就是阻塞
  - 4、一句话：这个线程没有发生，还会就会等待（反之：不会等待则是非阻塞）



##### server

```java
@Slf4j
public class Server {
    public static void main(String[] args) throws IOException {
        // 使用 nio 来理解阻塞模式，单线程
        ByteBuffer buffer = ByteBuffer.allocate (16);
        // 1、创建服务端
        ServerSocketChannel ssc = ServerSocketChannel.open ( );
        // 2、绑定监听端口
        ssc.bind (new InetSocketAddress (8080));
        // 3、创建连接集合
        List<SocketChannel> channels = new ArrayList<> ( );
        // 循环运行等待
        while(true){
            log.debug ("connection...");
            // 4、accept 建立与客户端连接， SocketChannel 用来与客户端之间通信
            SocketChannel sc = ssc.accept ( );//阻塞方法，线程停止运行
            log.debug("connected... {}", sc);
            channels.add (sc);
            for (SocketChannel channel : channels) {
                // 5、接受客户端发送的数据
                log.debug("before read... {}", channel);
                channel.read (buffer);// 写入（阻塞方法，线程停止运行）
                buffer.flip ();//切换为读模式
                debugRead(buffer);// 打印可读取的内容
                buffer.clear ();// 切换为写模式
                log.debug("after read...{}", channel);
            }
        }
    }
}

```





##### client

```java
public class Client {
    public static void main(String[] args) throws IOException {
        SocketChannel sc = SocketChannel.open ( );
        sc.connect (new InetSocketAddress ("localhost",8080));
        System.out.println ("waiting..." );
    }
}

```





![](https://v1.mykkto.cn/image/blog/2022/netty/202208212225480.png)

![](https://v1.mykkto.cn/image/blog/2022/netty/202208212234376.png)





#### 1.2、非阻塞

- 非阻塞模式下，相关方法都会不会让线程暂停
  - 在 ServerSocketChannel.accept 在没有连接建立时，会返回 null，继续运行
  - SocketChannel.read 在没有数据可读时，会返回 0，但线程不必阻塞，可以去执行其它 SocketChannel 的 read 或是去执行 ServerSocketChannel.accept 
  - 写数据时，线程只是等待数据写入 Channel 即可，无需等 Channel 通过网络把数据发送出去
- 但非阻塞模式下，即使没有连接建立，和可读数据，线程仍然在不断运行，白白浪费了 cpu
- 数据复制过程中，线程实际还是阻塞的（AIO 改进的地方）



##### server：服务器端，客户端代码不变

```java
@Slf4j
public class NotBlockServer {
    public static void main(String[] args) throws IOException {
        // 使用 nio 来理解阻塞模式，单线程
        ByteBuffer buffer = ByteBuffer.allocate (16);
        // 1、创建服务端
        ServerSocketChannel ssc = ServerSocketChannel.open ( );
        ssc.configureBlocking (false);// 切换为非阻塞模式，默认为阻塞
        // 2、绑定监听端口
        ssc.bind (new InetSocketAddress (8080));
        // 3、创建连接集合
        List<SocketChannel> channels = new ArrayList<> ( );
        // 循环运行等待
        while (true) {
            log.debug ("connection...");
            // 4、accept 建立与客户端连接， SocketChannel 用来与客户端之间通信
            SocketChannel sc = ssc.accept ( );//非阻塞方法，线程继续运行
            log.debug ("connected... {}", sc);
            channels.add (sc);
            for (SocketChannel channel : channels) {
                // 5、接受客户端发送的数据
                // 如果没有读到数据，read 返回 0
                int read = channel.read (buffer);// 写入（非阻塞方法，线程继续运行）
                if (read > 0) {
                    log.debug ("before read... {}", channel);
                    buffer.flip ( );//切换为读模式
                    debugRead (buffer);// 打印可读取的内容
                    buffer.clear ( );// 切换为写模式
                    log.debug ("after read...{}", channel);
                }
            }
        }
    }
}

```



多个客户端调试发现：其实不用的时候还在一直打印，CPU状态是打满的，自然也是性能的消耗



#### 1.3、多路复用

单线程可以配合 Selector 完成对多个 Channel 可读写事件的监控，这称之为多路复用

- 多路复用仅针对网络 IO、普通文件 IO 没法利用多路复用
- 如果不用 Selector 的非阻塞模式，线程大部分时间都在做无用功，而 Selector 能够保证
  - 有可连接事件时才去连接
  - 有可读事件才去读取
  - 有可写事件才去写入
    - 限于网络传输能力，Channel 未必时时可写，一旦 Channel 可写，会触发 Selector 的可写事件



### 2、Selector

![](https://v1.mykkto.cn/image/blog/2022/netty/202208242246169.png)

好处

- 一个线程配合 selector 就可以监控多个 channel 的事件，事件发生线程采取处理。避免非阻塞模式下所做的无用功
- 让这个线程能够被充分的利用
- 节约了线程的数量
- 减少了线程上下文切换



#### 1、创建

```java
Selector selector = Selector.open ( );
```



#### 2、绑定 Channel 事件

也称之为注册事件，绑定的事件 selector 才回关心

```java
channel.configureBlocking (false);
// 根据常量值来选择绑定类型 SelectionKey.OP_ACCEPT
SelectionKey key = channel.register (selector, 绑定事件);
```

- channel 必须在非阻塞模式下工作
- FileChannel 没有非阻塞模式，所以不能配合 selector 使用
- 绑定的事件类型
  - connect - 客户端连接成功时触发
  - accept   - 服务端成功连接时触发
  - read      - 数据可读入时触发
  - write     -  数据可写入时触发



#### 3、监听 Channel 事件

可以通过下面三种方法来监听是否有事件发生，方法的返回值代表有多少 channel 发生了事件。

方法一：阻塞直到绑定事件发生

```java
int count = selector.select();
```



方法二：阻塞直到绑定事件发生，或是超时（时间单位为 ms）

```java
int count = selector.select(long timeout)
```



方法三：不会阻塞，也就是不管有没有事件发生，就立即返回，自己根据返回值检查当前是否有事件发生。

```java
int count = selector.selectNow();
```



#### 4、💡 select 何时不阻塞

> - 事件发生时
>   - 客户端发起连接请求，会触发 accept事件
>   - 客户端发送数据过来，客户端正常，或者异常关闭时，都会触发 read 事件，另外如果发送的数据大于 buffer 缓冲区，会触发多次读取事件
>   - channel 可写，会触发 write 事件
>   - 在 Linux 下 nio bug 发生时
> - 调用 selector.wakeup()，唤醒
> - 调用 selector.clone()，关闭
> - selector 所在线程 interrupt（打断，阻断，暂定）



### 3、处理 accept 事件

主要用于建立连接

##### 客户端代码

```java
package com.kk.netty.nio.network.selector;

import java.io.IOException;
import java.net.Socket;

public class Client {
    public static void main(String[] args) {
        try (Socket socket = new Socket ("localhost", 8080)) {
            System.out.println (socket);
            socket.getOutputStream ( ).write ("hello!".getBytes ( ));
            System.in.read ( );
        } catch (IOException e) {
            e.printStackTrace ( );
        }
    }
}

```



##### 服务端代码

```java
package com.kk.netty.nio.network.selector;

import lombok.extern.slf4j.Slf4j;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.util.Iterator;
import java.util.Set;

@Slf4j
public class AcceptService {
    public static void main(String[] args) throws IOException {
        // 1、创建 selector 管理多个 channel
        Selector selector = Selector.open ( );
        ServerSocketChannel channel = ServerSocketChannel.open ( );
        channel.configureBlocking (false);

        // 2、建立 selector 和 channel 的联系(注册)
        channel.register (selector, SelectionKey.OP_ACCEPT);
        // SelectionKey 当事件发生后，通过它得到事件类型 和 哪个 channel 发生了该事件
        SelectionKey ckey = channel.register (selector, 0, null);

        // key 只关注 accept 事件
        // interestOps 只关注
        ckey.interestOps (SelectionKey.OP_ACCEPT);
        log.debug ("register key:{}", ckey);
        channel.bind (new InetSocketAddress (8080));

        while (true) {
            // 3、select 方法，没有事件发生，线程阻塞；有事件，才恢复运行
            //select 在事件未处理的时候，它不会阻塞；事件发生后，要么处理或者取消，不能置之不理
            selector.select ( );

            // 4、处理事件，selectedKey 内部包含了所有发生的事件
            Set<SelectionKey> keys = selector.selectedKeys ( );
            // 遍历所有事件，逐一处理
            Iterator<SelectionKey> iter = keys.iterator ( );
            while (iter.hasNext ( )) {
                SelectionKey key = iter.next ( );
                key.cancel ( );
            }
        }

    }
}

```





#### 💡 事件发生后能否不处理

```java
事件发生后，要么处理，要么取消（cancel），不能什么都不做，否则下次该事件仍会触发，这是因为 nio底层使用的是水平触发
```



### 4、处理 read 事件

##### service

```java
package com.kk.netty.nio.network.selector;

import lombok.extern.slf4j.Slf4j;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.util.Iterator;
import java.util.Set;

import static com.kk.netty.nio.ByteBufferUtil.debugRead;

@Slf4j
public class ReadService {
    // sscKey 、 scKey 一人一个管理员
    public static void main(String[] args) throws IOException {
        // 1、创建 selector 管理多个 channel
        Selector selector = Selector.open ( );
        ServerSocketChannel channel = ServerSocketChannel.open ( );
        channel.configureBlocking (false);

        // 2、建立 selector 和 channel 的联系(注册)
        channel.register (selector, SelectionKey.OP_ACCEPT);
        // SelectionKey 当事件发生后，通过它得到事件类型 和 哪个 channel 发生了该事件
        SelectionKey sscKey = channel.register (selector, 0, null);

        // key 只关注 accept 事件
        // interestOps 只关注
        sscKey.interestOps (SelectionKey.OP_ACCEPT);
        log.debug ("register key:{}", sscKey);
        channel.bind (new InetSocketAddress (8080));

        while (true) {
            // 3、select 方法，没有事件发生，线程阻塞；有事件，才恢复运行
            //select 在事件未处理的时候，它不会阻塞；事件发生后，要么处理或者取消，不能置之不理
            selector.select ( );

            // 4、处理事件，selectedKey 内部包含了所有发生的事件
            Set<SelectionKey> keys = selector.selectedKeys ( );
            // 遍历所有事件，逐一处理
            Iterator<SelectionKey> iter = keys.iterator ( );
            while (iter.hasNext ( )) {
                SelectionKey key = iter.next ( );
                log.debug ("key:{}", key);

                // 5、区分事件类型
                if (key.isAcceptable ( )) {// 如果是 accept
                    ServerSocketChannel channelTest1 = (ServerSocketChannel) key.channel ( );
                    SocketChannel sc = channelTest1.accept ( );
                    sc.configureBlocking (false);
                    SelectionKey scKey = sc.register (selector, 0, null);
                    scKey.interestOps (SelectionKey.OP_READ);
                } else if (key.isReadable ( )) {// 如果是 read
                    // 拿到触发的事件 channel
                    SocketChannel channlTest2 = (SocketChannel) key.channel ( );
                    ByteBuffer buffer = ByteBuffer.allocate (16);
                    channlTest2.read (buffer);
                    buffer.flip ( );
                    debugRead (buffer);
                }
                // key.cancel ( );
            }
        }

    }
}

```



输出结果：

![](https://v1.mykkto.cn/image/blog/2022/netty/202208292359698.png)



**因为 select 用完 key 没有 remove()**





#### 4.1、为什么要remove

![](https://v1.mykkto.cn/image/blog/2022/netty/202208302330191.png)



> 因为 select 在事件发生后，就会将相关的 key 放入 selectedKeys 集合，但不会在处理完后从 selectedKeys 集合中移除，需要我们自己编码删除。例如
>
> - 第一次触发了 ssckey 上的 accept 事件，没有移除 ssckey 
> - 第二次触发了 sckey 上的 read 事件，但这时 selectedKeys 中还有上次的 ssckey ，在处理时因为没有真正的 serverSocket 连上了，就会导致空指针异常



#### 4.2、remove 代码

拿到后就删掉，方式重复事件处理（写在后面也可以）

![](https://v1.mykkto.cn/image/blog/2022/netty/202208302331079.png)



#### 4.3、处理客户端断开

![](https://v1.mykkto.cn/image/blog/2022/netty/202208312250110.png)





#### 4.4、不处理边界问题

##### 以 BIO 为案例代码

**service**

```java
public class BoundaryService {
    public static void main(String[] args) throws Exception {
        ServerSocket ss = new ServerSocket (9000);
        while (true) {
            Socket socket = ss.accept ( );
            InputStream in = socket.getInputStream ( );
            // 这里这么写，有没有问题
            byte[] arr = new byte[4];
            while (true) {
                int read = in.read (arr);
                // 这里这么写，有没有问题
                if (read == -1) {
                    break;
                }
                System.out.println (new String (arr, 0, read));
            }
        }
    }
}
```







**client**

```java
public class BoundDaryClient {
    public static void main(String[] args) {
        Socket socket = null;
        try {
            socket = new Socket ("localhost", 9000);
            OutputStream out = socket.getOutputStream ( );
            out.write ("hello".getBytes ( ));
            out.write ("world".getBytes ( ));
            out.write ("你好".getBytes ( ));
            out.close ( );
        } catch (IOException e) {
            e.printStackTrace ( );
        }
    }
}
```



**输出**

```txt
hell
owor
ld�
�好

```



#### 4.5、处理消息的边界（容量超出）

**容量超出，就需要考虑扩容问题，这边没去考虑缩容，nettry 会有适配这些问题**

![](https://v1.mykkto.cn/image/blog/2022/netty/202209031033587.png)

- 一种思路是固定消息长度，数据包大小一样，服务器按预定长度读取，缺点是浪费带宽
- 另一种思路是按分隔符拆分，缺点是效率低
- TLV 格式，即 Type 类型、Length 长度、Value 数据，类型和长度已知的情况下，就可以方便获取消息大小，分配合适的 buffer，缺点是 buffer 需要提前分配，如果内容过大，则影响 server 吞吐量
  - Http 1.1 是 TLV 格式
  - Http 2.0 是 LTV 格式

![](https://v1.mykkto.cn/image/blog/2022/netty/202209031858792.png)



**这边 service 会用到附件知识，会把 bytebuffer 作为事件注册到 SelectorKey 上 **

既 ， register（t1,t2,t3）第三个参数上



**service**

```java
    // sscKey 、 scKey 一人一个管理员
    public static void main(String[] args) throws IOException {
        // 1、创建 selector 管理多个 channel
        Selector selector = Selector.open ( );
        ServerSocketChannel ssc = ServerSocketChannel.open ( );
        ssc.configureBlocking (false);

        // 2、建立 selector 和 channel 的联系(注册)
        ssc.register (selector, SelectionKey.OP_ACCEPT);
        // SelectionKey 当事件发生后，通过它得到事件类型 和 哪个 channel 发生了该事件
        SelectionKey sscKey = ssc.register (selector, 0, null);

        // key 只关注 accept 事件
        // interestOps 只关注
        sscKey.interestOps (SelectionKey.OP_ACCEPT);
        log.debug ("register key:{}", sscKey);
        ssc.bind (new InetSocketAddress (8080));

        while (true) {
            // 3、select 方法，没有事件发生，线程阻塞；有事件，才恢复运行
            //select 在事件未处理的时候，它不会阻塞；事件发生后，要么处理或者取消，不能置之不理
            selector.select ( );

            // 4、处理事件，selectedKey 内部包含了所有发生的事件
            Set<SelectionKey> keys = selector.selectedKeys ( );
            // 遍历所有事件，逐一处理
            Iterator<SelectionKey> iter = keys.iterator ( );
            while (iter.hasNext ( )) {
                SelectionKey key = iter.next ( );
                iter.remove ( );
                log.debug ("key:{}", key);

                // 5、区分事件类型
                if (key.isAcceptable ( )) {// 如果是 accept
                    ServerSocketChannel channel = (ServerSocketChannel) key.channel ( );
                    SocketChannel sc = channel.accept ( );
                    sc.configureBlocking (false);
                    ByteBuffer buffer = ByteBuffer.allocate (16);// attachment
                    // 将一个 byteBuffer 作为附件关联到 selectionKey 上
                    SelectionKey scKey = sc.register (selector, 0, buffer);
                    scKey.interestOps (SelectionKey.OP_READ);
                } else if (key.isReadable ( )) {// 如果是 read
                    try {
                        // 拿到触发的事件 channel
                        SocketChannel channel = (SocketChannel) key.channel ( );
                        // 获取 selectionKey 上关联的事件
                        //ByteBuffer buffer = ByteBuffer.allocate (16);
                        ByteBuffer buffer = (ByteBuffer) key.attachment ( );
                        // 如果是正常断开。read 的方法返回值是 -1
                        int read = channel.read (buffer);
                        if (read == -1) {
                            key.cancel ( );
                        } else {
                            // 压缩读取一次，确保数据完整
                            split (buffer);
                            // 如果游标的起始相等，说明容量已满，需要扩容
                            if (buffer.position ( ) == buffer.limit ( )) {
                                // 扩容两倍
                                ByteBuffer newBuffer = ByteBuffer.allocate (buffer.capacity ( ) * 2);
                                buffer.flip ( );
                                newBuffer.put (buffer);
                                key.attach (newBuffer);
                            }
                        }

                    } catch (Exception e) {
                        e.printStackTrace ( );
                        // 因为客户端断开了，因此需要将 key 取消（从 selector 得keys 集合中真正删除）
                        key.cancel ( );
                    }
                }

            }
        }

    }
```



**client**

```java
    public static void main(String[] args) throws IOException {
        SocketChannel sc =  SocketChannel.open ();
        sc.connect (new InetSocketAddress ("localhost", 8080));
        SocketAddress address = sc.getLocalAddress ( );
        sc.write (Charset.defaultCharset ().encode ("0123456789abcdef"));
        sc.write (Charset.defaultCharset ().encode ("012345Q\n6789abcdef3333\n"));
    }
```



#### 4.6、ByteBuffer 大小分配

- 每个 channel 都需要记录可能被切分的消息，因为 ByteBuffer 不能被多个 channel 共同使用，因此需要为每个 channel 维护一个独立的 ByteBuffer
- ByteBuffer 不能太大，比如一个 ByteBuffer 1Mb 的话，要支持百万连接就要 1Tb 内存，因此需要设计大小可变的 ByteBuffer
  - 一种思路是首先分配一个较小的 buffer，例如 4k，如果发现数据不够，再分配 8k 的 buffer，将 4k buffer 内容拷贝至 8k buffer，优点是消息连续容易处理，缺点是数据拷贝耗费性能，参考实现 [http://tutorials.jenkov.com/java-performance/resizable-array.html](http://tutorials.jenkov.com/java-performance/resizable-array.html)
  - 另一种思路是用多个数组组成 buffer，一个数组不够，把多出来的内容写入新的数组，与前面的区别是消息存储不连续解析复杂，优点是避免了拷贝引起的性能损耗



### 5、处理 write 事件

##### 一次性无法写完的例子

- 非阻塞模式下，无法保证把 buffer 中所有数据都写入 channel，因此需要追踪 write 方法的返回值（代表实际写入字节数）
- 用 selector 监听所有 channel 的可写事件，每个 channel 都需要一个 key 来跟踪 buffer，但这样又会导致占用内存过多，就有两阶段策略
  - 当消息处理器第一次写入消息时，才将 channel 注册到 selector 上
  - selector 检查 channel 上的可写事件，如果所有的数据写完了，就取消 channel 的注册
  - 如果不取消，会每次可写均会触发 write 事件



**service**

```java
public class WriteServer {
    public static void main(String[] args) throws Exception {
        ServerSocketChannel ssc = ServerSocketChannel.open ( );
        ssc.configureBlocking (false);
        ssc.bind (new InetSocketAddress (8080));

        Selector selector = Selector.open ( );
        ssc.register (selector, SelectionKey.OP_ACCEPT);

        while (true) {
            selector.select ( );
            Iterator<SelectionKey> iter = selector.selectedKeys ( ).iterator ( );

            while (iter.hasNext ( )) {
                SelectionKey key = iter.next ( );
                iter.remove ( );

                if (key.isAcceptable ( )) {
                    SocketChannel sc = ssc.accept ( );
                    sc.configureBlocking (false);
                    SelectionKey scKey = sc.register (selector, SelectionKey.OP_READ);
                    // 1、向客户端发送大量数据
                    StringBuffer sb = new StringBuffer ( );
                    for (int i = 0; i < 3000000; i++) {
                        sb.append ("a");
                    }
                    ByteBuffer buffer = Charset.defaultCharset ( ).encode (sb.toString ( ));

                    // 2、返回值代表实际写入的字节数
                    int write = sc.write (buffer);
                    System.out.println ("实际写入字节:" + write);

                    // 3、判断是否有剩余内容
                    if (buffer.hasRemaining ( )) {
                        // 4、关注可写事件(1+4)
                        // read 1  write 4
                        scKey.interestOps (scKey.interestOps ( ) + SelectionKey.OP_WRITE);
                        // 5、把未写完的数据 挂到 scKey上
                        scKey.attach (buffer);
                    }
                } else if (key.isWritable ( )) {
                    ByteBuffer buffer = (ByteBuffer) key.attachment ( );
                    SocketChannel sc = (SocketChannel) key.channel ( );
                    int write = sc.write (buffer);
                    System.out.println ("实际写入字节:" + write);
                    if (!buffer.hasRemaining ( )) {// 写完了(切换为读模式)
                        key.interestOps (key.interestOps ( ) - SelectionKey.OP_WRITE);
                        // 需要清除 buffer
                        key.attach (null);
                    }
                }
            }
        }
    }
}
```



**client**

```java
public class WriteClient {
    public static void main(String[] args) throws IOException {
        SocketChannel sc = SocketChannel.open ( );
        sc.connect (new InetSocketAddress (8080));

        // 接收数据
        int count = 0;
        while (true) {
            ByteBuffer buffer = ByteBuffer.allocate (1 << 20);
            count += sc.read (buffer);
            System.out.println (count );
            buffer.clear ();
        }
    }
}
```



##### 💡 write 为何要取消

只要向 channel 发送数据时，socket 缓冲可写，这个事件会频繁触发;

因此应当只在 socket 缓冲区写不下时再关注可写事件，数据写完之后再取消关注



### 6、多线程优化

##### 1、编码思路

- 两个线程之间传递数据，用队列在中间进行传输（这里用安全队列），netty底层也是
- 根据处理器数量，轮训（这边1.8的api jvm 有问题，在jdk 10的时候才修复）
- 单线程配一个选择器，专门处理 accept 事件
- 创建 cpu 核心数的线程，每个线程配一个选择器，轮流处理 read 事件





**2、如何拿到 cpu 个数**

> - Runtime.getRuntime().availableProcessors() 如果工作在 docker 容器下，因为容器不是物理隔离的，会拿到物理 cpu 个数，而不是容器申请时的个数
> - 这个问题直到 jdk 10 才修复，使用 jvm 参数 UseContainerSupport 配置， 默认开启



```java
@Slf4j
public class MultiThreadServer {
    public static void main(String[] args)  throws IOException{
        Thread.currentThread ().setName ("boss");
        ServerSocketChannel ssc = ServerSocketChannel.open ( );
        ssc.configureBlocking (false);
        Selector boos = Selector.open ( );
        SelectionKey boosKey = ssc.register (boos, 0, null);
        boosKey.interestOps (SelectionKey.OP_ACCEPT);
        ssc.bind (new InetSocketAddress (8080));
        // 1、创建固定数量的 worker 并初始化
        // Runtime.getRuntime ().availableProcessors () 处理器数量
        Worker[] workers = new Worker[Runtime.getRuntime ().availableProcessors ()];
        for (int i = 0; i < workers.length; i++) {
            workers[i] = new Worker ("WORKER-"+i);
        }
        AtomicInteger index = new AtomicInteger ();
        while (true) {
            boos.select ();
            Iterator<SelectionKey> iter = boos.selectedKeys ( ).iterator ( );
            while (iter.hasNext ( )) {
                SelectionKey key = iter.next ( );
                iter.remove ();
                if (key.isAcceptable ( )) {
                    SocketChannel sc = ssc.accept ( );
                    sc.configureBlocking (false);
                    log.debug("connected...{}", sc.getRemoteAddress());
                    // 2. 关联 selector
                    log.debug("before register...{}", sc.getRemoteAddress());
                    // 轮训
                    workers[index.getAndIncrement ()%workers.length].register (sc);
                    log.debug("after register...{}", sc.getRemoteAddress());
                }
            }
        }
    }

    static class Worker implements Runnable {
        private Thread thread;
        private Selector selector;
        private String name;
        private volatile boolean start = false;// 还未初始化
        private ConcurrentLinkedQueue<Runnable> queue = new ConcurrentLinkedQueue<> ( );

        private Worker(String name) {
            this.name = name;
        }

        // 初始化线程 和 selector
        public void register(SocketChannel sc) throws IOException {
            if (!start) {
                selector = Selector.open ( );
                thread = new Thread (this, name);
                thread.start ( );
                start = true;
            }
            // 唤醒 select 方法 boos
            selector.wakeup ( );
            // boos
            sc.register (selector, SelectionKey.OP_READ, null);
        }

        @Override
        public void run() {
            while (true) {
                try {
                    selector.select ( );// worker-0  阻塞
                    Iterator<SelectionKey> iter = selector.selectedKeys ( ).iterator ( );
                    while (iter.hasNext ( )) {
                        SelectionKey key = iter.next ( );
                        iter.remove ( );
                        if (key.isReadable ( )) {
                            ByteBuffer buffer = ByteBuffer.allocate (16);
                            SocketChannel channel = (SocketChannel) key.channel ( );
                            log.debug ("read...{}", channel.getRemoteAddress ( ));
                            channel.read (buffer);
                            buffer.flip ( );
                            debugAll (buffer);
                        }
                    }
                } catch (IOException e) {
                    e.printStackTrace ( );
                }
            }
        }
    }
}
```





### 7、UDP

- UDP 是无连接的，client 发送数据不会管 server 是否开启
- server 这边的 receive 方法会将接收到的数据存入 byte buffer，但如果数据报文超过 buffer 大小，多出来的数据会被默默抛弃









## 五、Nio vs Bio or Aio

### 1、stream vs channel



- stream 不会自动缓冲数据，channel 会利用系统提供的发送缓冲区、接收缓冲区（更为底层）
- stream 仅支持阻塞 API，channel 同时支持阻塞、非阻塞 API，网络 channel 可配合 selector 实现多路复用
- 二者均为全双工，即读写可以同时进行



### 2、IO模型





### 3、零拷贝

##### 3-1、传统IO问题

传统的 IO 将一个文件通过 socket 写出

```java
File f = new File("helloword/data.txt");
RandomAccessFile file = new RandomAccessFile(file, "r");

byte[] buf = new byte[(int)f.length()];
file.read(buf);

Socket socket = ...;
socket.getOutputStream().write(buf);
```

内部工作流程是这样的：

![](https://v1.mykkto.cn/image/blog/2022/netty/202209142205360.png)



1. java 本身并不具备 IO 读写能力，因此 read 方法调用后，要从 java 程序的**用户态**切换至**内核态**，去调用操作系统（Kernel）的读能力，将数据读入**内核缓冲区**。这期间用户线程阻塞，操作系统使用 DMA（Direct Memory Access）来实现文件读，其间也不会使用 cpu

   > DMA 也可以理解为硬件单元，用来解放 cpu 完成文件 IO

2. 从**内核态**切换回**用户态**，将数据从**内核缓冲区**读入**用户缓冲区**（即 byte[] buf），这期间 cpu 会参与拷贝，无法利用 DMA

3. 调用 write 方法，这时将数据从**用户缓冲区**（byte[] buf）写入 **socket 缓冲区**，cpu 会参与拷贝

4. 接下来要向网卡写数据，这项能力 java 又不具备，因此又得从**用户态**切换至**内核态**，调用操作系统的写能力，使用 DMA 将 **socket 缓冲区**的数据写入网卡，不会使用 cpu



可以看到中间环节较多，java 的 IO 实际不是物理设备级别的读写，而是缓存的复制，底层的真正读写是操作系统来完成的

- 用户态与内核态的切换发生了 3 次，这个操作比较重量级
- 数据拷贝了共 4 次



##### 3-2、NIO优化

通过 DirectByteBuf 

- ByteBuffer.allocate(10)  HeapByteBuffer 使用的还是 java 内存
- ByteBuffer.allocateDirect(10)  DirectByteBuffer 使用的是操作系统内存

![](https://v1.mykkto.cn/image/blog/2022/netty/202209142208809.png)

大部分步骤与优化前相同，不再赘述。唯有一点：java 可以使用 DirectByteBuf 将堆外内存映射到 jvm 内存中来直接访问使用

- 这块内存不受 jvm 垃圾回收的影响，因此内存地址固定，有助于 IO 读写
- java 中的 DirectByteBuf 对象仅维护了此内存的虚引用，内存回收分成两步
  - DirectByteBuf 对象被垃圾回收，将虚引用加入引用队列
  - 通过专门线程访问引用队列，根据虚引用释放堆外内存
- 减少了一次数据拷贝，用户态与内核态的切换次数没有减少



进一步优化（底层采用了 linux 2.1 后提供的 sendFile 方法），java 中对应着两个 channel 调用 transferTo/transferFrom 方法拷贝数据

![](https://v1.mykkto.cn/image/blog/2022/netty/202209142208773.png)

1. java 调用 transferTo 方法后，要从 java 程序的**用户态**切换至**内核态**，使用 DMA将数据读入**内核缓冲区**，不会使用 cpu
2. 数据从**内核缓冲区**传输到 **socket 缓冲区**，cpu 会参与拷贝
3. 最后使用 DMA 将 **socket 缓冲区**的数据写入网卡，不会使用 cpu

可以看到

- 只发生了一次用户态与内核态的切换
- 数据拷贝了 3 次

进一步优化（linux 2.4）

![](https://v1.mykkto.cn/image/blog/2022/netty/202209142209047.png)

1. java 调用 transferTo 方法后，要从 java 程序的**用户态**切换至**内核态**，使用 DMA将数据读入**内核缓冲区**，不会使用 cpu
2. 只会将一些 offset 和 length 信息拷入 **socket 缓冲区**，几乎无消耗
3. 使用 DMA 将 **内核缓冲区**的数据写入网卡，不会使用 cpu

整个过程仅只发生了一次用户态与内核态的切换，数据拷贝了 2 次。所谓的【零拷贝】，并不是真正无拷贝，而是在不会拷贝重复数据到 jvm 内存中，零拷贝的优点有

- 更少的用户态与内核态的切换
- 不利用 cpu 计算，减少 cpu 缓存伪共享
- 零拷贝适合小文件传输



### 4、AIO

AIO 用来解决数据复制阶段的阻塞问题

- 同步意味着，在进行读写操作时，线程需要等待结果，还是相当于闲置
- 异步意味着，在进行读写操作时，线程不必等待结果，而是将来由操作系统来通过回调方式由另外的线程来获得结果

> 异步模型需要底层操作系统（Kernel）提供支持
>
> - Windows 系统通过 IOCP 实现了真正的异步 IO
> - Linux 系统异步 IO 在 2.6 版本引入，但其底层实现还是用多路复用模拟了异步 IO，性能没有优势





### 5、

## Ⅱ、Netty

## 一、入门

### 1、概述

#### 1、Nerry是什么

Netty 是一个异步的、基于事件驱动的网络应用框架，用于快速开发可维护、高性能的网络服务器和客户端



#### 2、优势

- Netty vs NIO，工作量大，bug 多
  - 需要自己构建协议
  - 解决 TCP 传输问题，如粘包、半包
  - epoll 空轮询导致 CPU 100%
  - 对 API 进行增强，使之更易用，如 FastThreadLocal => ThreadLocal，ByteBuff => ByteBuffer
- Netty vs 其它网络应用框架
  - Mina 由 apache 维护，将来 3.x 版本可能会有较大重构，破坏 API 向下兼容性，Netty 的开发迭代更迅速，API 更简洁、文档更优秀
  - 久经考验，18年，Netty 版本
    - 2.x 2004
    - 3.x 2008
    - 4.x 2013
    - 5.x 已废弃（没有明显的性能提升，维护成本高）



#### 3、地位

Netty 在 Java 网络应用框架中的地位就好比：Spring 框架在 JavaEE 开发中的地位

以下的框架都使用了 Netty，因为它们有网络通信需求！

- Cassandra - nosql 数据库
- Spark - 大数据分布式计算框架
- Hadoop - 大数据分布式存储框架
- RocketMQ - ali 开源的消息队列
- ElasticSearch - 搜索引擎
- gRPC - rpc 框架
- Dubbo - rpc 框架
- Spring 5.x - flux api 完全抛弃了 tomcat ，使用 netty 作为服务器端
- Zookeeper - 分布式协调框架



### 2、Hello World

#### 1、目标

开发一个简单的服务器端和客户端

- 客户端向服务器端发送 hello, world
- 服务器仅接收，不返回



加入依赖

```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.39.Final</version>
</dependency>
```







#### 2、服务端

```java
public class Service {
    public static void main(String[] args) throws Exception {
        new ServerBootstrap ( )
                .group (new NioEventLoopGroup ( )) // 1
                .channel (NioServerSocketChannel.class) // 2
                .childHandler (new ChannelInitializer<NioSocketChannel> ( ) { // 3
                    protected void initChannel(NioSocketChannel ch) {
                        ch.pipeline ( ).addLast (new StringDecoder ( )); // 5
                        ch.pipeline ( ).addLast (new SimpleChannelInboundHandler<String> ( ) { // 6
                            @Override
                            protected void channelRead0(ChannelHandlerContext ctx, String msg) {
                                System.out.println (msg);
                            }
                        });
                    }
                }).bind (8080); // 4
    }
}
```

- 1 处，创建 NioEventLoopGroup，可以简单理解为 `线程池 + Selector` 

- 2 处，选择服务 Scoket 实现类，其中 NioServerSocketChannel 表示基于 NIO 的服务器端实现，其它实现还有

  ![](https://v1.mykkto.cn/image/blog/2022/netty/202209152257915.png)

- 3 处，为啥方法叫 childHandler，是接下来添加的处理器都是给 SocketChannel 用的，而不是给 ServerSocketChannel。ChannelInitializer 处理器（仅执行一次），它的作用是待客户端 SocketChannel 建立连接后，执行 initChannel 以便添加更多的处理器

- 4 处，ServerSocketChannel 绑定的监听端口

- 5 处，SocketChannel 的处理器，解码 ByteBuf => String

- 6 处，SocketChannel 的业务处理器，使用上一个处理器的处理结果



#### 3、客户端

```java
public class Client {
    public static void main(String[] args) throws InterruptedException {
        new Bootstrap ( )
                .group (new NioEventLoopGroup ( )) // 1
                .channel (NioSocketChannel.class) // 2
                .handler (new ChannelInitializer<Channel> ( ) { // 3
                    @Override
                    protected void initChannel(Channel ch) {
                        ch.pipeline ( ).addLast (new StringEncoder ( )); // 8
                    }
                })
                .connect ("127.0.0.1", 8080) // 4
                .sync ( ) // 5
                .channel ( ) // 6
                .writeAndFlush (new Date ( ) + ": hello world!"); // 7

    }
}
```



- 1 处，创建 NioEventLoopGroup，同 Server

- 2 处，选择客户 Socket 实现类，NioSocketChannel 表示基于 NIO 的客户端实现，其它实现还有

  ![](https://v1.mykkto.cn/image/blog/2022/netty/202209152303842.png)

- 3 处，添加 SocketChannel 的处理器，ChannelInitializer 处理器（仅执行一次），它的作用是待客户端 SocketChannel 建立连接后，执行 initChannel 以便添加更多的处理器

- 4 处，指定要连接的服务器和端口

- 5 处，Netty 中很多方法都是异步的，如 connect，这时需要使用 sync 方法等待 connect 建立连接完毕

- 6 处，获取 channel 对象，它即为通道抽象，可以进行数据读写操作

- 7 处，写入消息并清空缓冲区

- 8 处，消息会经过通道 handler 处理，这里是将 String => ByteBuf 发出

- 数据经过网络传输，到达服务器端，服务器端 5 和 6 处的 handler 先后被触发，走完一个流程



#### 4、梳理流程

![](https://v1.mykkto.cn/image/blog/2022/netty/202209171028505.png)

#### 5、详细解读

> - 把 channel 理解为数据的通道
> - 把 msg 理解为流动的数据，最开始输入是 ByteBuf，但经过 pipeline 的加工，会变成其它类型对象，最后输出又变成 ByteBuf
> - 把 handler 理解为数据的处理工序
>   - 工序有多道，合在一起就是 pipeline，pipeline 负责发布事件（读、读取完成...）传播给每个 handler， handler 对自己感兴趣的事件进行处理（重写了相应事件处理方法）
>   - handler 分 Inbound 和 Outbound 两类
> - 把 eventLoop 理解为处理数据的工人
>   - 工人可以管理多个 channel 的 io 操作，并且一旦工人负责了某个 channel，就要负责到底（绑定）
>   - 工人既可以执行 io 操作，也可以进行任务处理，每位工人有任务队列，队列里可以堆放多个 channel 的待处理任务，任务分为普通任务、定时任务
>   - 工人按照 pipeline 顺序，依次按照 handler 的规划（代码）处理数据，可以为每道工序指定不同的工人





### 3、组件

#### 1、EventLoop

事件循环对象

EventLoop 本质是一个单线程执行器（同时维护了一个 Selector），里面有 run 方法处理 Channel 上源源不断的 io 事件。

它的继承关系比较复杂

- 一条线是继承自 j.u.c.ScheduledExecutorService 因此包含了线程池中所有的方法
- 另一条线是继承自 netty 自己的 OrderedEventExecutor，
  - 提供了 boolean inEventLoop(Thread thread) 方法判断一个线程是否属于此 EventLoop
  - 提供了 parent 方法来看看自己属于哪个 EventLoopGroup



事件循环组

EventLoopGroup 是一组 EventLoop，Channel 一般会调用 EventLoopGroup 的 register 方法来绑定其中一个 EventLoop，后续这个 Channel 上的 io 事件都由此 EventLoop 来处理（保证了 io 事件处理时的线程安全）

- 继承自 netty 自己的 EventExecutorGroup
  - 实现了 Iterable 接口提供遍历 EventLoop 的能力
  - 另有 next 方法获取集合中下一个 EventLoop



以一个简单的实现为例：

```java
// 内部创建了两个 EventLoop, 每个 EventLoop 维护一个线程
DefaultEventLoopGroup group = new DefaultEventLoopGroup(2);
System.out.println(group.next());
System.out.println(group.next());
System.out.println(group.next());
```

输出

```
io.netty.channel.DefaultEventLoop@60f82f98
io.netty.channel.DefaultEventLoop@35f983a6
io.netty.channel.DefaultEventLoop@60f82f98
```

也可以使用 for 循环

```java
DefaultEventLoopGroup group = new DefaultEventLoopGroup(2);
for (EventExecutor eventLoop : group) {
    System.out.println(eventLoop);
}
```

输出

```
io.netty.channel.DefaultEventLoop@60f82f98
io.netty.channel.DefaultEventLoop@35f983a6
```



```
io.netty.channel.DefaultEventLoop@60f82f98
io.netty.channel.DefaultEventLoop@35f983a6
```



##### 💡 优雅关闭

优雅关闭 `shutdownGracefully` 方法。该方法会首先切换 `EventLoopGroup` 到关闭状态从而拒绝新的任务的加入，然后在任务队列的任务都处理完成后，停止线程的运行。从而确保整体应用是在正常有序的状态下退出的



##### 2、NioEventLoop 处理普通任务

NioEventLoop 除了可以处理 io 事件，同样可以向它提交普通任务

```java
NioEventLoopGroup nioWorkers = new NioEventLoopGroup(2);

log.debug("server start...");
Thread.sleep(2000);
nioWorkers.execute(()->{
    log.debug("normal task...");
});
```

输出

```
22:30:36 [DEBUG] [main] c.i.o.EventLoopTest2 - server start...
22:30:38 [DEBUG] [nioEventLoopGroup-2-1] c.i.o.EventLoopTest2 - normal task...
```

> 可以用来执行耗时较长的任务



##### 3、NioEventLoop 处理定时任务

```java
NioEventLoopGroup nioWorkers = new NioEventLoopGroup(2);

log.debug("server start...");
Thread.sleep(2000);
nioWorkers.scheduleAtFixedRate(() -> {
    log.debug("running...");
}, 0, 1, TimeUnit.SECONDS);
```

输出

```
22:35:15 [DEBUG] [main] c.i.o.EventLoopTest2 - server start...
22:35:17 [DEBUG] [nioEventLoopGroup-2-1] c.i.o.EventLoopTest2 - running...
22:35:18 [DEBUG] [nioEventLoopGroup-2-1] c.i.o.EventLoopTest2 - running...
22:35:19 [DEBUG] [nioEventLoopGroup-2-1] c.i.o.EventLoopTest2 - running...
22:35:20 [DEBUG] [nioEventLoopGroup-2-1] c.i.o.EventLoopTest2 - running...
...
```

> 可以用来执行定时任务



#### 2、Channel

channel 的主要作用

- close() 可以用来关闭 channel
- closeFuture() 用来处理 channel 的关闭
  - sync 方法作用是同步等待 channel 关闭
  - 而 addListener 方法是异步等待 channel 关闭
- pipeline() 方法添加处理器
- write() 方法将数据写入
- writeAndFlush() 方法将数据写入并刷出



#### 2.1、ChannelFuture

这时刚才的客户端代码

```java
new Bootstrap()
    .group(new NioEventLoopGroup())
    .channel(NioSocketChannel.class)
    .handler(new ChannelInitializer<Channel>() {
        @Override
        protected void initChannel(Channel ch) {
            ch.pipeline().addLast(new StringEncoder());
        }
    })
    .connect("127.0.0.1", 8080)
    .sync()
    .channel()
    .writeAndFlush(new Date() + ": hello world!");
```

现在把它拆开来看

```java
ChannelFuture channelFuture = new Bootstrap()
    .group(new NioEventLoopGroup())
    .channel(NioSocketChannel.class)
    .handler(new ChannelInitializer<Channel>() {
        @Override
        protected void initChannel(Channel ch) {
            ch.pipeline().addLast(new StringEncoder());
        }
    })
    .connect("127.0.0.1", 8080); // 1

channelFuture.sync().channel().writeAndFlush(new Date() + ": hello world!");
```

- 1 处返回的是 ChannelFuture 对象，它的作用是利用 channel() 方法来获取 Channel 对象

**注意** connect 方法是异步的，意味着不等连接建立，方法执行就返回了。因此 channelFuture 对象中不能【立刻】获得到正确的 Channel 对象

实验如下：

```java
ChannelFuture channelFuture = new Bootstrap()
    .group(new NioEventLoopGroup())
    .channel(NioSocketChannel.class)
    .handler(new ChannelInitializer<Channel>() {
        @Override
        protected void initChannel(Channel ch) {
            ch.pipeline().addLast(new StringEncoder());
        }
    })
    .connect("127.0.0.1", 8080);

System.out.println(channelFuture.channel()); // 1
channelFuture.sync(); // 2
System.out.println(channelFuture.channel()); // 3
```

- 执行到 1 时，连接未建立，打印 `[id: 0x2e1884dd]`
- 执行到 2 时，sync 方法是同步等待连接建立完成
- 执行到 3 时，连接肯定建立了，打印 `[id: 0x2e1884dd, L:/127.0.0.1:57191 - R:/127.0.0.1:8080]`

除了用 sync 方法可以让异步操作同步以外，还可以使用回调的方式：

```java
ChannelFuture channelFuture = new Bootstrap()
    .group(new NioEventLoopGroup())
    .channel(NioSocketChannel.class)
    .handler(new ChannelInitializer<Channel>() {
        @Override
        protected void initChannel(Channel ch) {
            ch.pipeline().addLast(new StringEncoder());
        }
    })
    .connect("127.0.0.1", 8080);
System.out.println(channelFuture.channel()); // 1
channelFuture.addListener((ChannelFutureListener) future -> {
    System.out.println(future.channel()); // 2
});
```

- 执行到 1 时，连接未建立，打印 `[id: 0x749124ba]`
- ChannelFutureListener 会在连接建立时被调用（其中 operationComplete 方法），因此执行到 2 时，连接肯定建立了，打印 `[id: 0x749124ba, L:/127.0.0.1:57351 - R:/127.0.0.1:8080]`





#### 2.2、CloseFuture

```java
package com.kk.netty.basice.demo2;

import io.netty.bootstrap.Bootstrap;
import io.netty.channel.Channel;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelFutureListener;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.string.StringEncoder;

import java.util.Date;

public class ChannelFutureTest {
    public static void main(String[] args) throws InterruptedException {
        ChannelFuture channelFuture = new Bootstrap ( )
                .group (new NioEventLoopGroup ( ))
                .channel (NioSocketChannel.class)
                .handler (new ChannelInitializer<Channel> ( ) {
                    @Override
                    protected void initChannel(Channel ch) {
                        ch.pipeline ( ).addLast (new StringEncoder ( ));
                    }
                })
                .connect ("127.0.0.1", 8080);
        //channelFuture.sync ( ).channel ( ).writeAndFlush (new Date ( ) + "hello!");
        channelFuture.channel ( ).writeAndFlush (new Date ( ) + "hello!");

        System.out.println (channelFuture.channel ( ));
//        channelFuture.sync ();
//        System.out.println (channelFuture.channel () );

        channelFuture.addListener ((ChannelFutureListener) future -> {
            System.out.println (future.channel ( ));
        });

    }
}
```



#### 💡 异步提升的是什么

- 看到这里会有疑问：为什么不在一个线程中去执行建立连接、去执行关闭 channel，那样不是也可以吗？非要用这么复杂的异步方式：比如一个线程发起建立连接，另一个线程去真正建立连接
- 还有同学会笼统地回答，因为 netty 异步方式用了多线程、多线程就效率高。其实这些认识都比较片面，多线程和异步所提升的效率并不是所认为的



思考下面的场景，4 个医生给人看病，每个病人花费 20 分钟，而且医生看病的过程中是以病人为单位的，一个病人看完了，才能看下一个病人。假设病人源源不断地来，可以计算一下 4 个医生一天工作 8 小时，处理的病人总数是：`4 * 8 * 3 = 96`

![](https://v1.mykkto.cn/image/blog/2022/netty/202209241222774.png)



经研究发现，看病可以细分为四个步骤，经拆分后每个步骤需要 5 分钟，如下

![](https://v1.mykkto.cn/image/blog/2022/netty/202209241221714.png)



因此可以做如下优化，只有一开始，医生 2、3、4 分别要等待 5、10、15 分钟才能执行工作，但只要后续病人源源不断地来，他们就能够满负荷工作，并且处理病人的能力提高到了 `4 * 8 * 12` 效率几乎是原来的四倍

![](https://v1.mykkto.cn/image/blog/2022/netty/202209241221462.png)

要点

- 单线程没法异步提高效率，必须配合多线程、多核 cpu 才能发挥异步的优势
- 异步并没有缩短响应时间，反而有所增加
- 合理进行任务拆分，也是利用异步的关键



#### 3、Future & Promise



在异步处理时，经常用到这两个接口

首先要说明 netty 中的 Future 与 jdk 中的 Future 同名，但是是两个接口，netty 的 Future 继承自 jdk 的 Future，而 Promise 又对 netty Future 进行了扩展

- jdk Future 只能同步等待任务结束（或成功、或失败）才能得到结果
- netty Future 可以同步等待任务结束得到结果，也可以异步方式得到结果，但都是要等任务结束
- netty Promise 不仅有 netty Future 的功能，而且脱离了任务独立存在，只作为两个线程间传递结果的容器

| 功能/名称    | jdk Future                     | netty Future                                                 | Promise      |
| ------------ | ------------------------------ | ------------------------------------------------------------ | ------------ |
| cancel       | 取消任务                       | -                                                            | -            |
| isCanceled   | 任务是否取消                   | -                                                            | -            |
| isDone       | 任务是否完成，不能区分成功失败 | -                                                            | -            |
| get          | 获取任务结果，阻塞等待         | -                                                            | -            |
| getNow       | -                              | 获取任务结果，非阻塞，还未产生结果时返回 null                | -            |
| await        | -                              | 等待任务结束，如果任务失败，不会抛异常，而是通过 isSuccess 判断 | -            |
| sync         | -                              | 等待任务结束，如果任务失败，抛出异常                         | -            |
| isSuccess    | -                              | 判断任务是否成功                                             | -            |
| cause        | -                              | 获取失败信息，非阻塞，如果没有失败，返回null                 | -            |
| addLinstener | -                              | 添加回调，异步接收结果                                       | -            |
| setSuccess   | -                              | -                                                            | 设置成功结果 |
| setFailure   | -                              | -                                                            | 设置失败结果 |



#### 3-1、JDK-future

```java

    // jdk-future
    public static void test() throws ExecutionException, InterruptedException {
        // 1.线程池
        ExecutorService executorService = Executors.newFixedThreadPool (2);
        // 2.提交任务
        Future<Integer> future = executorService.submit (new Callable<Integer> ( ) {
            @Override
            public Integer call() throws InterruptedException {
                log.info ("执行计算！");
                Thread.sleep (1000);
                return 50;
            }
        });
        // 3、主线程通过 future获取结果
        log.info ("等待结果中......");
        log.info ("结果为：{}", future.get ( ));
    }
```

输出

>10:08:46.201 [main] INFO com.kk.netty.basice.demo2.FutureAndPromise - 等待结果中......
>10:08:46.201 [pool-1-thread-1] INFO com.kk.netty.basice.demo2.FutureAndPromise - 执行计算！
>10:08:47.205 [main] INFO com.kk.netty.basice.demo2.FutureAndPromise - 结果为：50



#### 3-2、netty-future

```java
    // netty-future()
    public static void test11() throws ExecutionException, InterruptedException {
        NioEventLoopGroup group = new NioEventLoopGroup ( );
        EventLoop eventLoop = group.next ( );
        Future<Integer> future = eventLoop.submit (new Callable<Integer> ( ) {
            @Override
            public Integer call() throws InterruptedException {
                log.info ("执行计算！");
                Thread.sleep (1000);
                return 50;
            }
        });
        log.info ("等待结果中......");

        //>>>>>>>>>>>>同步>>>>>>>>>>>>>>>>>
        // 控制台打印是main线程打印
//        log.info ("结果为：{}", future.get ( ));
        //>>>>>>>>>>>>同步>>>>>>>>>>>>>>>>>

        //>>>>>>>>>>>>异步>>>>>>>>>>>>>>>>>
        future.addListener (new GenericFutureListener<Future<? super Integer>> ( ) {
            @Override
            public void operationComplete(Future<? super Integer> future) throws Exception {
                // 控制台打印是 nioEventLoopGroup 线程组打印
                log.info ("结果为：{}", future.getNow ( ));
            }
        });
        //>>>>>>>>>>>>异步>>>>>>>>>>>>>>>>>

    }
```

> 10:40:10.185 [main] INFO com.kk.netty.basice.demo2.FutureAndPromise - 等待结果中......
> 10:40:10.189 [nioEventLoopGroup-2-1] INFO com.kk.netty.basice.demo2.FutureAndPromise - 执行计算！
> 10:40:11.190 [nioEventLoopGroup-2-1] INFO com.kk.netty.basice.demo2.FutureAndPromise - 结果为：50

#### 3-(1-2)小结

无论是juc中的 future 还是 netty 中的 future 都是来自于 juc

![](https://v1.mykkto.cn/image/blog/2022/netty/202209251043399.png)

上面两种获取的 future 方式 都是被动返回得到的，若要主动，此时可以用 netty 中的 Promise



#### 3.1、同步处理任务成功

```java
        DefaultEventLoop eventExecutors = new DefaultEventLoop ( );
        DefaultPromise<Integer> promise = new DefaultPromise<> (eventExecutors);

        eventExecutors.execute (() -> {
            try {
                Thread.sleep (1000);
            } catch (InterruptedException e) {
                e.printStackTrace ( );
            }
            log.debug ("set success, {}", 10);
            promise.setSuccess (10);
        });

        log.debug ("start...");
        log.debug ("getNow:{}", promise.getNow ( )); // 还没有结果
        log.debug ("get:{}", promise.get ( ));
    }
```

输出

>17:15:38.604 [main] DEBUG com.kk.netty.basice.demo2.FutureAndPromise - start...
>17:15:38.604 [main] DEBUG com.kk.netty.basice.demo2.FutureAndPromise - getNow:null
>17:15:39.605 [defaultEventLoop-1-1] DEBUG com.kk.netty.basice.demo2.FutureAndPromise - set success, 10
>17:15:39.605 [main] DEBUG com.kk.netty.basice.demo2.FutureAndPromise - get:10

#### 3.2、异步处理任务成功

```java
        DefaultEventLoop eventExecutors = new DefaultEventLoop ( );
        DefaultPromise<Integer> promise = new DefaultPromise<> (eventExecutors);

        // 设置回调，异步接收结果
        promise.addListener (future -> {
            // 这里的 future 就是上面的 promise
            log.debug ("getNow:{}", future.getNow ( ));
        });

        eventExecutors.execute (() -> {
            try {
                Thread.sleep (1000);
            } catch (InterruptedException e) {
                e.printStackTrace ( );
            }
            log.debug ("set success, {}", 10);
            promise.setSuccess (10);
        });
        log.debug ("start...");
```

输出

>11:49:30 [DEBUG] [main] c.i.o.DefaultPromiseTest2 - start...
>11:49:31 [DEBUG] [defaultEventLoop-1-1] c.i.o.DefaultPromiseTest2 - set success, 10
>11:49:31 [DEBUG] [defaultEventLoop-1-1] c.i.o.DefaultPromiseTest2 - 10

#### 3.3、同步处理任务失败 - sync & get

```java
        DefaultEventLoop eventExecutors = new DefaultEventLoop ( );
        DefaultPromise<Integer> promise = new DefaultPromise<> (eventExecutors);

        eventExecutors.execute (() -> {
            try {
                Thread.sleep (1000);
            } catch (InterruptedException e) {
                e.printStackTrace ( );
            }
            RuntimeException e = new RuntimeException ("error...");
            log.debug ("set failure, {}", e.toString ( ));
            promise.setFailure (e);
        });

        log.debug ("start...");
        log.debug ("getNow:{}", promise.getNow ( ));
        promise.get ( ); // sync() 也会出现异常，只是 get 会再用 ExecutionException 包一层异常
```

输出

>12:11:07 [DEBUG] [main] c.i.o.DefaultPromiseTest2 - start...
>12:11:07 [DEBUG] [main] c.i.o.DefaultPromiseTest2 - null
>12:11:08 [DEBUG] [defaultEventLoop-1-1] c.i.o.DefaultPromiseTest2 - set failure, java.lang.RuntimeException: error...
>Exception in thread "main" java.util.concurrent.ExecutionException: java.lang.RuntimeException: error...
>	at io.netty.util.concurrent.AbstractFuture.get(AbstractFuture.java:41)
>	at com.itcast.oio.DefaultPromiseTest2.main(DefaultPromiseTest2.java:34)
>Caused by: java.lang.RuntimeException: error...
>	at com.itcast.oio.DefaultPromiseTest2.lambda$main$0(DefaultPromiseTest2.java:27)
>	at io.netty.channel.DefaultEventLoop.run(DefaultEventLoop.java:54)
>	at io.netty.util.concurrent.SingleThreadEventExecutor$5.run(SingleThreadEventExecutor.java:918)
>	at io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74)
>	at io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30)
>	at java.lang.Thread.run(Thread.java:745)

#### 3.4、同步处理任务失败 - await

```java
        DefaultEventLoop eventExecutors = new DefaultEventLoop ( );
        DefaultPromise<Integer> promise = new DefaultPromise<> (eventExecutors);

        eventExecutors.execute (() -> {
            try {
                Thread.sleep (1000);
            } catch (InterruptedException e) {
                e.printStackTrace ( );
            }
            RuntimeException e = new RuntimeException ("error...");
            log.debug ("set failure, {}", e.toString ( ));
            promise.setFailure (e);
        });

        log.debug ("start...");
        log.debug ("getNow:{}", promise.getNow ( ));
        promise.await ( ); // 与 sync 和 get 区别在于，不会抛异常
        log.debug ("result {}", (promise.isSuccess ( ) ? promise.getNow ( ) : promise.cause ( )).toString ( ));
```

输出

>12:18:53 [DEBUG] [main] c.i.o.DefaultPromiseTest2 - start...
>12:18:53 [DEBUG] [main] c.i.o.DefaultPromiseTest2 - null
>12:18:54 [DEBUG] [defaultEventLoop-1-1] c.i.o.DefaultPromiseTest2 - set failure, java.lang.RuntimeException: error...
>12:18:54 [DEBUG] [main] c.i.o.DefaultPromiseTest2 - result java.lang.RuntimeException: error...

#### 3.5、异步处理任务失败

```java
       DefaultEventLoop eventExecutors = new DefaultEventLoop ( );
        DefaultPromise<Integer> promise = new DefaultPromise<> (eventExecutors);

        promise.addListener (future -> {
            log.debug ("result {}", (promise.isSuccess ( ) ? promise.getNow ( ) : promise.cause ( )).toString ( ));
        });

        eventExecutors.execute (() -> {
            try {
                Thread.sleep (1000);
            } catch (InterruptedException e) {
                e.printStackTrace ( );
            }
            RuntimeException e = new RuntimeException ("error...");
            log.debug ("set failure, {}", e.toString ( ));
            promise.setFailure (e);
        });

        log.debug ("start...");
```

输出

>12:04:57 [DEBUG] [main] c.i.o.DefaultPromiseTest2 - start...
>12:04:58 [DEBUG] [defaultEventLoop-1-1] c.i.o.DefaultPromiseTest2 - set failure, java.lang.RuntimeException: error...
>12:04:58 [DEBUG] [defaultEventLoop-1-1] c.i.o.DefaultPromiseTest2 - result java.lang.RuntimeException: error...

#### 3.6、await 死锁检查

```java
       DefaultEventLoop eventExecutors = new DefaultEventLoop ( );
        DefaultPromise<Integer> promise = new DefaultPromise<> (eventExecutors);

        eventExecutors.submit (() -> {
            System.out.println ("1");
            try {
                promise.await ( );
                // 注意不能仅捕获 InterruptedException 异常
                // 否则 死锁检查抛出的 BlockingOperationException 会继续向上传播
                // 而提交的任务会被包装为 PromiseTask，它的 run 方法中会 catch 所有异常然后设置为 Promise 的失败结果而不会抛出
            } catch (Exception e) {
                e.printStackTrace ( );
            }
            System.out.println ("2");
        });
        eventExecutors.submit (() -> {
            System.out.println ("3");
            try {
                promise.await ( );
            } catch (Exception e) {
                e.printStackTrace ( );
            }
            System.out.println ("4");
        });
```

输出

>1
>2
>3
>4
>io.netty.util.concurrent.BlockingOperationException: DefaultPromise@47499c2a(incomplete)
>at io.netty.util.concurrent.DefaultPromise.checkDeadLock(DefaultPromise.java:384)



#### 4、Handler & Pipeline

ChannelHandler 用来处理 Channel 上的各种事件，分为入栈、出栈两种。所有 ChannelHandler 被连成一串，就是 Pipeline

- 入站处理器通常是 ChannelInboundHandlerAdapter 的子类，主要用来<font color ='red'>读</font>取客户端数据，写回结果
- 出站处理器通常是 ChannelOutboundHandlerAdapter 的子类，主要对<font color ='red'>写</font>回结果进行加工

打个比喻，每个 Channel 是一个产品的加工车间，Pipeline 是车间中的流水线，ChannelHandler 就是流水线上的各道工序，而后面要讲的 ByteBuf 是原材料，经过很多工序的加工：先经过一道道入站工序，再经过一道道出站工序最终变成产品



先搞清楚顺序，服务端

```java
new ServerBootstrap()
    .group(new NioEventLoopGroup())
    .channel(NioServerSocketChannel.class)
    .childHandler(new ChannelInitializer<NioSocketChannel>() {
        protected void initChannel(NioSocketChannel ch) {
            ch.pipeline().addLast(new ChannelInboundHandlerAdapter(){
                @Override
                public void channelRead(ChannelHandlerContext ctx, Object msg) {
                    System.out.println(1);
                    ctx.fireChannelRead(msg); // 1
                }
            });
            ch.pipeline().addLast(new ChannelInboundHandlerAdapter(){
                @Override
                public void channelRead(ChannelHandlerContext ctx, Object msg) {
                    System.out.println(2);
                    ctx.fireChannelRead(msg); // 2
                }
            });
            ch.pipeline().addLast(new ChannelInboundHandlerAdapter(){
                @Override
                public void channelRead(ChannelHandlerContext ctx, Object msg) {
                    System.out.println(3);
                    ctx.channel().write(msg); // 3
                }
            });
            ch.pipeline().addLast(new ChannelOutboundHandlerAdapter(){
                @Override
                public void write(ChannelHandlerContext ctx, Object msg, 
                                  ChannelPromise promise) {
                    System.out.println(4);
                    ctx.write(msg, promise); // 4
                }
            });
            ch.pipeline().addLast(new ChannelOutboundHandlerAdapter(){
                @Override
                public void write(ChannelHandlerContext ctx, Object msg, 
                                  ChannelPromise promise) {
                    System.out.println(5);
                    ctx.write(msg, promise); // 5
                }
            });
            ch.pipeline().addLast(new ChannelOutboundHandlerAdapter(){
                @Override
                public void write(ChannelHandlerContext ctx, Object msg, 
                                  ChannelPromise promise) {
                    System.out.println(6);
                    ctx.write(msg, promise); // 6
                }
            });
        }
    })
    .bind(8080);
```

客户端

```java
new Bootstrap()
    .group(new NioEventLoopGroup())
    .channel(NioSocketChannel.class)
    .handler(new ChannelInitializer<Channel>() {
        @Override
        protected void initChannel(Channel ch) {
            ch.pipeline().addLast(new StringEncoder());
        }
    })
    .connect("127.0.0.1", 8080)
    .addListener((ChannelFutureListener) future -> {
        future.channel().writeAndFlush("hello,world");
    });
```

服务器端打印：

```
1
2
3
6
5
4
```





## 二、进阶

### 1、

### 2、

### 3、



## 三、源码

### 1、

### 2、

### 3、





## Ⅲ、常识（面试）

##### 1、多线程缺点

- 内存占用高
- 线程上下文切换成本高
- 只适合连接数少的场景



##### 2、线程池缺点

- 阻塞模式下，线程仅能处理一个 socket 连接
- 仅适合短连接场景



##### 3、FileChannel 工作模式

FileChannel 只能工作在阻塞模式下



##### 4、



##### 5、



##### 6、



##### 7、



##### 8、



## 参考文献 ↓



try 1.7 ：https://blog.csdn.net/renvictory/article/details/108844745



