# JVM-本地方法接口

## 一、定义

- 简单来讲，**一个 Native Method 就是一个 Java 调用非 Java代码的接口**，一个 Native Method 是这样一个 Java 方法：该方法的实现由非 Java 语言实现，比如 C。这个特征并非 Java 特有，很多其他的编程语言都有这一机制，比如在C++ 中，你可以用 extern “C” 告知 C++ 编译器去调用一个 C 的函数。
- 在定义一个 native method 时，并不提供实现体（有些像定义一个Java interface），因为其实现体是由非 Java 语言在外面实现的。
- 本地接口的作用是融合不同的编程语言为 Java 所用，它的初衷是融合 C/C++ 程序。

```java
/**
 * 本地方法
 * 标识符 native 可以与其他所有的 Java 标识符连用，但是 abstract 除外。
 * @author qinweizhao
 * @since 2022/4/1
 */
public  class IHaveNatives {

    //abstract 没有方法体
    // public abstract void abstractMethod(int x);

    //native 和 abstract不能共存，native是有方法体的，由C语言来实现
    public native void Native1(int x);

    /**
     使用native标识为本地方法接口
     */
    native static public long Native2();

    native synchronized private float Native3(Object o);

    native void Native4(int[] array) throws Exception;

}
```

## 二、使用原因

有些层次的任务用 Java 实现起来不容易，或者我们对程序的效率很在意时，问题就来了。

-  **与 Java 环境外交互**

  有时 Java 应用需要与 Java 外面的环境交互，这是本地方法存在的主要原因。 你可以想想java需要与一些底层系统，如擦偶偶系统或某些硬件交换信息时的情况。本地方法正式这样的一种交流机制：它为我们提供了一个非常简洁的接口，而且我们无需去了解 Java 应用之外的繁琐细节。

- **与操作系统交互**

  JVM 支持着 Java 语言本身和运行库，它是 Java 程序赖以生存的平台，它由一个解释器（解释字节码）和一些连接到本地代码的库组成。然而不管怎样，它毕竟不是一个完整的系统，它经常依赖于一些底层系统的支持。这些底层系统常常是强大的操作系统。通过使用本地方法，我们得以用 Java 实现了 jre 的与底层系统的交互，甚至 JVM 的一些部分就是用 C 写的。还有，如果我们要使用一些java 语言本身没有提供封装的操作系统特性时，我们也需要使用本地方法。

- #### Sun’s Java

  Sun 的解释器是用 C 实现的，这使得它能像一些普通的 C 一样与外部交互。jre 大部分是用 Java 实现的，它也通过一些本地方法与外界交互。例如：类 java.lang.Thread 的setPriority() 方法是用 Java 实现的，但是它实现调用的事该类里的本地方法 setPriority0（）。这个本地方法是用 C 实现的，并被植入 JVM 内部，在Windows 95 的平台上，这个本地方法最终将调用 Win32 SetProority() API。这是一个本地方法的具体实现由 JVM 直接提供，更多的情况是本地方法由外部的动态链接库（external dynamic link library）提供，然后被 JVM 调用。

## 三、现状

目前该方法使用的越来越少了，除非是与硬件有关的应用，比如通过 Java 程序驱动打印机或者 Java 系统管理生产设备，在企业级应用中已经比较少见。因为现在的异构领域间的通信很发达，比如可以使用 Socket 通信，也可以使用 Web Service 等等。
