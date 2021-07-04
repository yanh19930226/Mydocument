# ========     JVM 体系架构    ==========

JVM是运行在操作系统之上的，它与硬件没有直接的交互

## JVM整体结构

![image](https://gitee.com/heguangchuan/rainmeter/raw/master/img/jvm/jiagou2.png)

## JVM生命周期

### 虚拟机的启动

**jvm**的启动是通过引导类加载器 **bootstrap class loader**创建的一个初始化类 **initial class**来完成的，这个类是由虚拟机的具体实现指定的。

### 虚拟机的执行

执行一个java程序时，真正在执行的是一个叫做**JVM**的进程，程序开始执行时，他才执行

### 虚拟机的退出

程序正常结束、抛出异常、操作系统错误、System.exit()、Runtime.halt()等

## JVM发展历程

### Sun Classic VM

第一代商用的JVM，JDK1.4时被完全淘汰，其内部只有解释器，无编译器

### Exact VM

英雄气短，最终被 Hotspot 取代

### Hotspot VM

目前占有绝对的市场，称霸武林

![image](https://gitee.com/heguangchuan/rainmeter/raw/master/img/jvm/version.png)

### JRockit VM

只针对服务端的服务器,因此没有解释器，全世界最快的JVM

### J9  VM

与HotSpot定位一样，服务器端、桌面应用、嵌入式等多用途

### 其他虚拟机

略