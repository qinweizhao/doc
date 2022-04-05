# JVM-性能优化

## 一、概述

### 1、背景说明

1. 生产环境中出现的问题
   - 发生内存泄漏如何处理？
   - 给服务器分配多少内存合适？
   - 如何对垃圾回收器进行性能调优？
   - 生产环境中CPU负载过高如何处理？
   - 生产环境该给应用分配多少线程？
   - 不加log，如何确定请求执行到了某一行代码？
   - 不加log，如何实时查看某个方法的入参与返回值？

2. 为什么要调优？
   - 防止出现OOM
   - 解决OOM
   - 减少Full GC的频率

3. 不同阶段的考虑
   - 上线前
   - 运行阶段
   - 线上出现OOM

### 2、优概述

1.  监控的依据
   - 运行日志
   - 异常堆栈
   - GC日志
   - 线程快照
   - 堆转存快照

2. 调优的大方向
   - 合理编写代码
   - 充分并合理使用硬件资源
   - 合理的进行JVM调优

### 3、性能优化的步骤

1. 发现问题（性能监控）
   - GC频率
   - CPU负载过高
   - OOM
   - 内存泄漏
   - 死锁
   - 响应时间过长

2. 排查问题（性能分析）
   - 打印GC日志，分析GC日志
   - 导出dump文件 进行分析
   - 使用第三方的工具查看JVM状态
   - jstack查看堆栈信息

3. 解决问题（性能调优）
   - 增加内存  根据业务背景选择GC器
   - 优化代码 控制内存使用
   - 增加机器，分散节点压力
   - 合理设置线程数量
   - 使用中间件提高效率，缓存，消息队列等

### 4、性能评价/测试指标

1. 停顿时间

   - 定义：

     ​	提交请求和返回响应之间的时间，（一般关注平均值）

   - 在垃圾回收时，STW的时间。

2. 吞吐量
   - 对单位时间内完成的工作量的量度
   - 在GC中，表示用户代码执行时间占总运行时间的比例。

3. 并发数
   - 同一时刻，对服务器有实际交互的请求数
4.  内存占用
   - 堆所占的内存大小

## 二、监控及诊断工具-命令行

### 1、概述

性能诊断是软件工程师在日常工作中经常要面对和解决的问题，在用户体验至上的今天，解决好应用的i性能问题能带来巨大的收益。Java作为最流行的编程语言之一，其应用性能诊断一直受到业界的广泛关注。可能造成Java应用出现性能问题的因素非常多，比如

- 线程控制

- 磁盘读写

- 数据库访问

- 网络IO

- 垃圾收集

  要想定位这些问题，一款优秀的性能诊断工具必不可少。

### 2、jps （Java Process Status）查看正在运行的Java进程

参数

- -q: 仅显示ID

- -l 输出程序的全限定名

- -m 输出进程启动时传递给main的参数

- -v 列出JVM参数

注意：

​	如果Java进程关闭了默认开启的`UserPerfData`参数，则jps无法探测。

### 3、jstat JVM统计信息

- jstat 用于监视虚拟机各种运行状态信息，比如类装载，内存，GC，JIT编译等。

  > jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]

- option

  - -class  显示ClassLoader的相关信息

  - -gc 显示与GC相关的堆信息（后缀C表示容量 后缀U表示已使用 后缀T表示耗时）

  - -gccapacity: 显示内容与-gc基本相同，但是输出主要关注堆各个区域最大最小空间

  - -gcutil :关注已使用占总空间的比例
  - -gcnew 显示新生代的情况
  - -gcold 显示老年代的情况
  - -gccause 与gcutil输出一样，最后多显示最后一次发生GC的原因

  - -compiler: 显示JIT编译器编译过的方法 耗时等

  - -printcompilation: 输出已经被JIT编译过的方法

- -t 表示在输出信息前加时间戳 表示程序的运行时间 单位 s 

- -h 表示多少行内容之后输入一次表头信息

- interval 指定更新统计数据的周期，单位ms 每隔n毫秒输入一次


- count  与上个参数 配合使用 表示一共输出多少次 空表示一直输出

### 4、jinfo 查看和修改JVM配置参数

> jinfo <option> <pid>

- option

  - 查看

    - jinfo -sysprpos pid ：查看该进程的全部配置信息
    - jinfo -flag 参数名 pid： 查看指定参数值

    - -flag <具体参数> pid： 查看具体参数的值

  - 修改

    - 布尔类型: jinfo -flag +-参数 pid
    - 非布尔类型: jinfo -flag 参数名=参数值 pid


### 5、jmap 导出内存映像文件&内存使用情况

### 6、jhat JDK自带的堆分析工具

### 7、jstack 打印JVM中的线程快照

### 8、 jcmd 多功能命令行

>  在JDK1.7之后，新增了一个命令行工具jcmd ，它是一个多功能的工具，实现之前的所有功能。

语法

- jcmd -l 列出所有的Java进程

- jcmd pid help : 列出所有可用的指令

- jcmd pid 命令（上图中可选的命令均可）


### 9、jstatd 远程主机信息收集

![image-20210711133144394](https://img.qinweizhao.com/2022/04/2022-04-05_115230.png)
> 通过jstatd 可以建立本地计算机与远程监控工具的通信。jstatd将本机的Java应用程序信息传递给远程计算机。

## 三、监控及诊断工具-GUI

### 1、工具概述

JDK自带的GUI工具

	- JConsole
	- VisualVM
	- JMC

第三方工具

- MAT

- Jprofiler

- Arthas

- Btrace

  ......

### 1、JConsole

基本概述

从 Java5开始，JDK自带的Java监控和管理控制台。

启动

cmd jconsole或者在jdk bin目录下执行即可

### 3、Visual VM

> 一个功能强大的集合-故障诊断和性能监控的可视化工具

启动

cmd jconsole或者在jdk bin目录下执行即可

### 4、eclipse MAT

> 需要JDK11之后，由于我的JDK版本不支持，此处略

### 5、Jprofiler

> 软件下载地址: [JProfiler11免费版下载](https://www.jb51.net/softs/608640.html#downintro2)
>
> IDEA插件: [下载地址](https://plugins.jetbrains.com/files/253/122553/idea-jprofiler.zip?updateId=122553&pluginId=253&family=INTELLIJ)

 特点

- 使用方便，界面操作友好

- 对被分析的应用影响小

- CPU，Thread，Memory分析功能强大

- 支持对JDBC，jsp,servlet,socket等分析

- 支持多种模式

- 支持远程JVM

- 跨平台，支持多种系统


数据采集方式

- 在class文件加载之前，将相关的统计功能代码写入到class中，对正在运行的JVM有影响。

  - 优点：调用堆栈信息准确
  - 缺点：如果分析的类较多，则CPU开销较高。此模式配合Filter使用。

- Sampling抽样模式

  一定时间将每个线程栈中方法栈的信息统计出来

  - 优点：对应用影响小
  - 缺点： 一些数据无法提供

### 6、Arthas（阿里巴巴）

>  阿尔萨斯，是Alibaba开源的Java诊断工具，在线排查问题，无需重启，动态跟踪Java代码，实时监控JVM状态。

[官方地址](https://arthas.aliyun.com/zh-cn/)

### 7、关于内存泄漏

可达性分析算法来判断对象是否是不再使用的对象，本质是判断一个对象是否还被引用，由于代码的实现不同就会出现很多种内存泄漏问题。

内存泄露的理解

- 严格来说，只有对象不再被程序使用，但是GC又不能回收的情况叫做内存泄露
- 实际上，很多时候一些不好的编程习惯会使得某些对象的生命周期变的很长，比如局部变量定义为类变量，类变量定义为静态变量，此时，可能会导致OOM，这种也可以叫做宽泛意义上的`内存泄露`
- 比如

![image-20210711170026972](https://img.qinweizhao.com/2022/04/2022-04-05_115310.png)

- 内存泄漏过多，就会导致内存溢出



演示内存泄漏并排查问题

代码

```java
/**
演示内存泄漏
*/
public class Stack {
    private Object[] elements;
    private int size = 0;
    /**
     * 默认容量
     */
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    /**
     * 入栈
     */
    public void push(Object e) { //入栈
        ensureCapacity();
        elements[size++] = e;
    }

    /**
     * 出栈
     */
    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];

        //不加此行代码 容易造成内存泄漏
        //elements[size] = null;
        return result;
    }

    /**
     * 扩容
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

代码中出栈操作只是将当前位置下移，；出栈的对象并未置Null,此时，指向的对象无法被GC回收

##### 解决方案

在返回出栈对象之前，将该对象置空 即加入 即可

```java
public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null;
        return result;
    }
```

## 四、JVM运行时参数

### 1、JVM参数选项类型

- 标准参数选项

  - 比较稳定 以 - 开头

    ```java
        -d32          使用 32 位数据模型 (如果可用)
        -d64          使用 64 位数据模型 (如果可用)
        -server       选择 "server" VM
                      默认 VM 是 server.
    
        -cp <目录和 zip/jar 文件的类搜索路径>
        -classpath <目录和 zip/jar 文件的类搜索路径>
                      用 ; 分隔的目录, JAR 档案
                      和 ZIP 档案列表, 用于搜索类文件。
        -D<名称>=<值>
                      设置系统属性
        -verbose:[class|gc|jni]
                      启用详细输出
        -version      输出产品版本并退出
        -version:<值>
                      警告: 此功能已过时, 将在
                      未来发行版中删除。
                      需要指定的版本才能运行
        -showversion  输出产品版本并继续
        -jre-restrict-search | -no-jre-restrict-search
                      警告: 此功能已过时, 将在
                      未来发行版中删除。
                      在版本搜索中包括/排除用户专用 JRE
        -? -help      输出此帮助消息
        -X            输出非标准选项的帮助
        -ea[:<packagename>...|:<classname>]
        -enableassertions[:<packagename>...|:<classname>]
                      按指定的粒度启用断言
        -da[:<packagename>...|:<classname>]
        -disableassertions[:<packagename>...|:<classname>]
                      禁用具有指定粒度的断言
        -esa | -enablesystemassertions
                      启用系统断言
        -dsa | -disablesystemassertions
                      禁用系统断言
        -agentlib:<libname>[=<选项>]
                      加载本机代理库 <libname>, 例如 -agentlib:hprof
                      另请参阅 -agentlib:jdwp=help 和 -agentlib:hprof=help
        -agentpath:<pathname>[=<选项>]
                      按完整路径名加载本机代理库
        -javaagent:<jarpath>[=<选项>]
                      加载 Java 编程语言代理, 请参阅 java.lang.instrument
        -splash:<imagepath>
                      使用指定的图像显示启动屏幕
    ```

    有关详细信息, 请参阅 http://www.oracle.com/technetwork/java/javase/documentation/index.html。

-  -X参数选项

  ```java
      -Xmixed           混合模式执行（默认）
      -Xint             仅解释模式执行
      -Xbootclasspath:<用 ; 分隔的目录和 zip/jar 文件>
                        设置引导类和资源的搜索路径
      -Xbootclasspath/a:<用 ; 分隔的目录和 zip/jar 文件>
                        附加在引导类路径末尾
      -Xbootclasspath/p:<用 ; 分隔的目录和 zip/jar 文件>
                        置于引导类路径之前
      -Xdiag            显示附加诊断消息
      -Xnoclassgc        禁用类垃圾收集
      -Xincgc           启用增量垃圾收集
      -Xloggc:<file>    将 GC 状态记录在文件中（带时间戳）
      -Xbatch           禁用后台编译
      -Xms<size>        设置初始 Java 堆大小
      -Xmx<size>        设置最大 Java 堆大小
      -Xss<size>        设置 Java 线程堆栈大小
      -Xprof            输出 cpu 分析数据
      -Xfuture          启用最严格的检查，预计会成为将来的默认值
      -Xrs              减少 Java/VM 对操作系统信号的使用（请参阅文档）
      -Xcheck:jni       对 JNI 函数执行其他检查
      -Xshare:off       不尝试使用共享类数据
      -Xshare:auto      在可能的情况下使用共享类数据（默认）
      -Xshare:on        要求使用共享类数据，否则将失败。
      -XshowSettings    显示所有设置并继续
      -XshowSettings:system
                        （仅限 Linux）显示系统或容器
                        配置并继续
      -XshowSettings:all
                        显示所有设置并继续
      -XshowSettings:vm 显示所有与 vm 相关的设置并继续
      -XshowSettings:properties
                        显示所有属性设置并继续
      -XshowSettings:locale
                        显示所有与区域设置相关的设置并继续
  ```

  -X 选项是非标准选项。如有更改，恕不另行通知。

-  -XX参数选项

  分类

  - 布尔类型 : -XX +<option> 启用     -XX -<option> 停用
  - 非布尔类型：-XX:name=value

### 2、添加JVM参数选项

-  IDEA：

  - 点击配置运行环境，打开VM选项即可输入

- ### 运行jar包： 

  - java <参数> -jar demo.jar

- ### 通过Tomcat运行：    

  - catalina.bat中添加：set "JAVA_OPTS=-Xms100m -Xmx100m"

### 3、常用的JVM参数选项

打印设置的参数

- -XX:+PrintCommandLineFlags 表示程序运行前打印出JVM参数
- -XX:+PrintFlagsInitial 表示打印出所有参数的默认值
- -XX:+PrintFlagsFinal 打印出最终的参数值
- -XX:+PrintVMOptions 打印JVM的参数

栈

- -Xss128k

 堆

- -Xms600m    设置堆的初始大小
- -Xmx600m   设置堆的最大大小
- -XX:NewSize=1024m  设置年轻代的初始大小
- -XX:MaxNewSize=1024m  设置年轻代的最大值
- -XX:SurvivorRatio=8 伊甸园和幸存者的比例
- -XX:NewRatio=4 设置老年代和新生代的比例
- -XX:MaxTenuringThreshold=15 设置晋升老年代的年龄条件

方法区

- 永久代
  - -XX:PermSize=256m 设置永久代初始大小
  - -XX:MaxPernSize=256m 设置永久代的最大大小
- 元空间
  - -XX:MetasapceSize=256m 设置初始元空间大小
  - -XX:MaxMatespaceSize=256m 设置最大元空间大小 默认无限制

直接内存

- -XX:MaxDirectMemorySize  设置直接内存的容量，默认与堆最大值一样。
