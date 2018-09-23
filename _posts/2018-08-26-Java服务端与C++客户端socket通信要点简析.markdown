# Java服务端与C++客户端socket通信要点简析

    公司项目中有java服务端和C++客户端通信的需求。跨语言情况下有一些比较特殊的点需要处理，在查阅了一些资料并实践之后，收发双方实现了正确通信。

    socket通信可以使用字符流，也可以使用字节流的形式。统一使用字节流可以简化模型，字节对于语言而言是一视同仁的。我总结的几个方面：

1. 大端模式和小端模式
2. 网络字节序和主机字节序，字节对齐和字节填充
3. C++无符号型和Java有符号型数据的相互转化
4. 采用的socket开发框架

## 大端模式和小端模式

    可以直观的举例：

    Big-endian模式下： byte  byValue[] = {0x01, 0x02, 0x03, 0x04};

    Little-endian模式下：byte  byValue[] = {0x04, 0x03, 0x02, 0x01};

## 网络字节序与主机字节序

    网络字节序遵循大端规则。所有语言在通信时均转化成网络字节序传输。

    不同语言有自己的主机字节序。java的主机字节序是大端的，C++的字节序是小端的。所以C++在传输时要对字节序进行转化。

## C++无符号型和Java有符号型数据的相互转化

    C++是无符号的，意味着足够大的数据在java中会产生溢出。反之不会。所以java端需要用高一级别的数据类型来接收C++的数据类型，比如使用8字节有符号long来接收C++的4字节无符号int。

    转化函数的例子：

    有符号long --> 字节数组

```java
public static byte[] unsigned_int_2byte(long length) {
        byte[] targets = new byte[4];
        for (int i = 0; i < 4; i++) {
            int offset = (targets.length - 1 - i) * 8;
            targets[i] = (byte) ((length >>> offset) & 0xff);
        }
        return targets;
    }
```

    字节数组 --> 有符号long

```java
public static long byteToLongForUnsignInt(byte[] b) {
        long s = 0;
        // 最低位
        long s3 = b[0] & 0xff;
        long s2 = b[1] & 0xff;
        long s1 = b[2] & 0xff;
        long s0 = b[3] & 0xff;
        // s0不变
        s1 <<= 8;
        s2 <<= 16;
        s3 <<= 24;
        s = s0 | s1 | s2 | s3;
        return s;
    }
```

## socket开发框架

    项目中java作为服务器，提供一个SocketServer进行持久监听客户端连接。
    注册：客户端请求建立控制连接，之后很多请求通过控制连接进行。
    GPS:客户端在控制连接中请求，服务端应答后再新开一个数据连接。
    socket开发框架的细节将作为下一篇博客的主题。

## 注意点

    字节对齐和字节填充：
    字节对齐后方便对数据进行处理。字节填充可以使字节对齐。比如在1字节类型后填充3个保留字节，可以保证四字节对齐。

## reference：

[1] https://www.cnblogs.com/yuanyq/p/java_unsigned_types.html 无符号和字节序
[2] https://blog.csdn.net/jiangxinyu/article/details/8211612 java和c通讯要点解析