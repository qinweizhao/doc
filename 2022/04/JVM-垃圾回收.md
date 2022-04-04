# JVM-垃圾回收

垃圾是指在运行程序中没有任何指针指向的对象，这个对象就是需要被回收的垃圾。

## 一、概述

- 对于 Java 开发人员而言，自动内存管理就像是一个黑匣子，如果过度依赖于“自动”，那么这将会是一场灾难，最严重的就会弱化 Java 开发人员在程序出现内存溢出时定位问题和解决问题的能力。此时，了解 JVM 的自动内存分配和内存回收原理就显得非常重要，只有在真正了解 JVM 是如何管理内存后，我们才能够在遇见 OutOfMemoryError 时， 快速地根据错误异常日志定位问题和解决问题。当需要排查各种内存溢出、内存泄漏问题时，当垃圾收集成为系统达到更高并发量的瓶颈时，我们就必须对这些“自动化”的技术实施必要的监控和调节。

- 垃圾回收器可以对年轻代回收，也可以对老年代回收，甚至是全堆和方法区的回收。其中，Java堆是垃圾收集器的工作重点。

  - 从次数上讲：
    - 频繁收集Young区
    - 较少收集0ld区
    - 基本不动Perm区(方法区)

  ![image-20210703093933143](/Users/weizhao/Data/imageBed/image-20210703093933143.png)

## 二、相关算法

### 1、标记阶段

**垃圾标记阶段：主要是为了判断对象是否存活。**

- 在堆里存放着几乎所有的 Java 对象实例，在 GC 执行垃圾回收之前，首先需要区分出内存中哪些是存活对象，哪些是已经死亡的对象。只有被标记为己经死亡的对象，GC才会在执行垃圾回收时，释放掉其所占用的内存空间，因此这个过程我们可以称为垃圾标记阶段。
- 当一个对象已经不再被任何的存活对象继续引用时，就可以宣判为已经死亡。判断对象存活一般有两种方式：**引用计数算法**和**可达性分析算法**。

1. **引用计数算法**

   - 概念

     - 引用计数算法（Reference Counting）比较简单，对每个对象保存一个整型的引用计数器属性。用于记录对象被引用的情况。
     - 对于一个对象 A，只要有任何一个对象引用了 A，则 A 的引用计数器就加1；当引用失效时，引用计数器就减1。只要对象A的引用计数器的值为0，即表示对象 A 不可能再被使用，可进行回收。

   - 优点

     实现简单，垃圾对象便于辨识；判定效率高，回收没有延迟性。

   - 缺点

     - 需要单独的字段存储计数器，这样的做法增加了存储空间的开销。

     - 每次赋值都需要更新计数器，伴随着加法和减法操作，这增加了时间开销。

     - 引用计数器有一个严重的问题，即无法处理循环引用的情况。这是一 条致命缺陷，**导致在 Java 的垃圾回收器中没有使用这类算法**。

       ![image-20210703094326043](/Users/weizhao/Data/imageBed/image-20210703094326043.png)

   - 测试 Java 中没有使用引用计数算法

     ```java
     /**
      * 代码测试Java中没有使用引用计数算法来判断对象是否为垃圾
      * VM参数：-XX:+PrintGCDetails
      *
      * @author qinweizhao
      * @since 2022/4/4
      */
     public class RefCountGC {
         
         /**
          * 故意占用空间10M
          */
         byte data[] = new byte[1024 * 1024 * 10];
     
         private Object ref = null;
     
         public static void main(String[] args) {
             RefCountGC refCountGC1 = new RefCountGC();
             RefCountGC refCountGC2 = new RefCountGC();
     
             //循环引用
             refCountGC1.ref = refCountGC2;
             refCountGC2.ref = refCountGC1;
     
             //取消外界对其的引用
             refCountGC1 = null;
             refCountGC2 = null;
     
             //手动GC
           	///把这行代码注释掉
             System.gc();
         }
         
     }
     ```

     - 手动 GC (**System.gc();**)关闭的时候，未执行 GC。

       ```txt
       Heap
        PSYoungGen      total 153088K, used 28375K [0x0000000715580000, 0x0000000720000000, 0x00000007c0000000)
         eden space 131584K, 21% used [0x0000000715580000,0x00000007173c7cb8,0x000000071d600000)
         from space 21504K, 0% used [0x000000071eb00000,0x000000071eb00000,0x0000000720000000)
         to   space 21504K, 0% used [0x000000071d600000,0x000000071d600000,0x000000071eb00000)
        ParOldGen       total 349696K, used 0K [0x00000005c0000000, 0x00000005d5580000, 0x0000000715580000)
         object space 349696K, 0% used [0x00000005c0000000,0x00000005c0000000,0x00000005d5580000)
        Metaspace       used 3165K, capacity 4496K, committed 4864K, reserved 1056768K
         class space    used 346K, capacity 388K, committed 512K, reserved 1048576K
       ```

     - 手动执行GC打开，执行GC。

       ```txt
       [GC (System.gc()) [PSYoungGen: 28375K->464K(153088K)] 28375K->472K(502784K), 0.0016633 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
       [Full GC (System.gc()) [PSYoungGen: 464K->0K(153088K)] [ParOldGen: 8K->351K(349696K)] 472K->351K(502784K), [Metaspace: 3146K->3146K(1056768K)], 0.0023506 secs] [Times: user=0.01 sys=0.00, real=0.00 secs] 
       Heap
        PSYoungGen      total 153088K, used 6579K [0x0000000715580000, 0x0000000720000000, 0x00000007c0000000)
         eden space 131584K, 5% used [0x0000000715580000,0x0000000715bece60,0x000000071d600000)
         from space 21504K, 0% used [0x000000071d600000,0x000000071d600000,0x000000071eb00000)
         to   space 21504K, 0% used [0x000000071eb00000,0x000000071eb00000,0x0000000720000000)
        ParOldGen       total 349696K, used 351K [0x00000005c0000000, 0x00000005d5580000, 0x0000000715580000)
         object space 349696K, 0% used [0x00000005c0000000,0x00000005c0057ff0,0x00000005d5580000)
        Metaspace       used 3165K, capacity 4496K, committed 4864K, reserved 1056768K
         class space    used 346K, capacity 388K, committed 512K, reserved 1048576K
       ```

   - 小结

     1. 引用计数算法，是很多语言的资源回收选择，例如 Python，它更是同时支持引用计数和垃圾收集机制。具体哪种最优是要看场景的，业界有大规模实践中仅保留引用计数机制，以提高吞吐量的尝试。
     2. Java并没有选择引用计数，是因为其存在一个基本的难题，也就是很难处理循环引用关系。
     3. Python如何解决循环引用？
        - 手动解除：很好理解，就是在合适的时机，解除引用关系。
        - 使用弱引用weakref，weakref是Python提供的标准库，旨在解决循环引用。

2. **可达性分析算法**

   **可达性分析算法：也可以称为根搜索算法、追踪性垃圾收集**。相对于引用计数而言，可达性分析算法**解决了循环引用的问题。防止了内存泄露的发生**。

   - **基本思路**

     - 可达性分析算法是以根对象（GCRoots）为起始点，按照从上至下的方式**搜索被根对象集合所连接的目标对象是否可达**。

     - 使用可达性分析算法之后，内存中存活的对象都会被根对象集合直接或者间接连接，搜索走过的路径叫做**引用链**。

     - 如果目标对象没有任何引用链相连，则是不可达的，就意味着该对象己经死亡，可以标记为垃圾对象。

     - 在可达性分析算法中，只有能够被根对象集合直接或者间接连接的对象才是存活对象。

       ![image-20210703102423042](/Users/weizhao/Data/imageBed/image-20210703102423042.png)

   - GCRoots 可以是哪些元素？

     - 虚拟机栈中引用的对象，如：各个线程被调用的方法中使用到的参数、局部变量等。
     - 本地方法栈内 JNI（通常说的本地方法）引用的对象。
     - 方法区中类静态属性引用的对象，如：Java类的引用类型静态变量。
     - 方法区中常量引用的对象，如：字符串常量池（StringTable）里的引用。
     - 所有被同步锁 synchronized 持有的对象。
     - Java 虚拟机内部的引用，基本数据类型对应的Class对象，一些常驻的异常对象（如：NullPointerException、OutofMemoryError），系统类加载器。
     - 反映 Java 虚拟机内部情况的 JMXBean、JVMTI 中注册的回调、本地代码缓存等。

     **小技巧：**由于 Root 采用栈方式存放变量和指针，所以如果一个指针，它保存了堆内存里面的对象，但是自己又不存放在堆内存里面，那它就是一个 Root。

     **总结：**除了堆空间的周边，比如：虚拟机栈、本地方法栈、方法区、字符串常量池等地方对堆空间进行引用的，都可以作为 GC Roots 进行可达性分析。

     ![image-20210703205542925](/Users/weizhao/Data/imageBed/image-20210703205542925.png)

     除了这些固定的GC Roots集合以外，根据用户所选用的垃圾收集器以及当前回收的内存区域不同，还可以有其他对象“临时性”地加入，共同构成完整 GC Roots 集合。比如：分代收集和局部回收（PartialGC）。

     如果只针对堆中的某一块区域进行垃圾回收（比如：典型的只针对新生代），必须考虑到内存区域是虚拟机自己的实现细节，更不是孤立封闭的，这个区域的对象完全有可能被其他区域的对象所引用，这时候就需要一并将关联的区域对象也加入 GC Roots 集合中去考虑，才能保证可达性分析的准确性。

### 2、清除阶段

当成功区分出内存中存活对象和死亡对象之后，GC 接下来的任务就是执行垃圾回收，释放掉无用对象所占用的空间。目前比较常用的算法有三种：

- 标记清除算法
- 复制算法
- 标记压缩算法

1. **标记清除算法（Mark-Sweep）**

   - 背景

     标记清除算法是一种非常基础和常见的垃圾收集算法。 

   - 执行过程

     当堆中的有效内存空间（available memory）被耗尽的时候，就会停止整个程序（也被称为stop the world），然后进行两项工作，第一项则是标记，第二项则是清除：

     - **标记：**Collector从引用根节点开始遍历，标记所有被引用的对象。一般是在对象的Header中记录为可达对象。（标记的是被引用的对象，也就是可达对象，并非标记的是即将被清除的垃圾对象）。

     - **清除：**Collector对堆内存从头到尾进行线性的遍历，如果发现某个对象在其Header中没有标记为可达对象，则将其回收

     ![image-20210704133431320](/Users/weizhao/Data/imageBed/image-20210704133431320.png)

   - 优点

     - 常用，简单。

   - 缺点

     - 标记清除算法的效率不算高。
     - 在进行 GC 的时候，需要停止整个应用程序，用户体验较差。
     - 这种方式清理出来的空闲内存是不连续的，产生内碎片，需要维护一个空闲列表。

     所谓的**清除**并不是真的置空，而是把需要清除的对象地址保存在空闲的地址列表里。下次有新对象需要加载时，判断垃圾的位置空间是否够，如果够，就存放（也就是覆盖原有的地址）。


2. **复制算法**

   - 背景

     为了解决标记清除算法效率方面的问题，M.L.Minsky于1963年发表了著名的论文，“使用双存储区的Lisp语言垃圾收集器CA LISP Garbage Collector Algorithm Using Serial Secondary Storage）”。M.L.Minsky在该论文中描述的算法被人们称为复制（Copying）算法，它也被M.L.Minsky本人成功地引入到了Lisp语言的一个实现版本中。

   - 核心思想

     将活着的内存空间分为两块，每次只使用其中一块，在垃圾回收时将正在使用的内存中的存活对象复制到未被使用的内存块中，之后清除正在使用的内存块中的所有对象，交换两个内存的角色，最后完成垃圾回收。

     ![image-20210704134558378](/Users/weizhao/Data/imageBed/image-20210704134558378.png)

   - 优点

     - 没有标记和清除过程，实现简单，**运行高效**。
     - 复制过去以后保证**空间连续性**，不会出现“碎片”问题。

   - 缺点

     - 此算法的缺点也是很明显的，就是需要两倍的内存空间。
     - 对于 G1 这种分拆成为大量 region 的 GC，复制而不是移动，意味着 GC 需要维护 region 之间对象引用关系，不管是内存占用或者时间开销也不小。

   - 应用场景

     - 如果系统中的垃圾对象很多，复制算法需要复制的存活对象数量并不会太大，效率较高
     - 老年代大量的对象存活，那么复制的对象将会有很多，效率会很低

   - 在新生代，对常规应用的垃圾回收，一次通常可以回收70% - 99% 的内存空间。回收性价比很高。所以现在的商业虚拟机都是用这种收集算法回收新生代。

3. **标记压缩算法**

   - 背景

     - 复制算法的高效性是建立在存活对象少、垃圾对象多的前提下的。这种情况在新生代经常发生，但是在老年代，更常见的情况是大部分对象都是存活对象。如果依然使用复制算法，由于存活对象较多，复制的成本也将很高。因此，**基于老年代垃圾回收的特性，需要使用其他的算法。**
     - 标记-清除算法的确可以应用在老年代中，但是该算法不仅执行效率低下，而且在执行完内存回收后还会产生内存碎片，所以JVM的设计者需要在此基础之上进行改进。标记-压缩（Mark-Compact）算法由此诞生。
     - 1970年前后，G.L.Steele、C.J.Chene 和 D.s.Wise 等研究者发布标记-压缩算法。在许多现代的垃圾收集器中，人们都使用了标记-压缩算法或其改进版本。

   - 执行过程

     - 第一阶段和标记清除算法一样，从根节点开始标记所有被引用对象

     - 第二阶段将所有的存活对象压缩到内存的一端，按顺序排放。之后，清理边界外所有的空间。

       ![image-20210704140307884](/Users/weizhao/Data/imageBed/image-20210704140307884.png)

   - **标记-压缩算法与标记-清除算法的比较**

     - 标记-压缩算法的最终效果等同于标记-清除算法执行完成后，再进行一次内存碎片整理，因此，也可以把它称为标记-清除-压缩（Mark-Sweep-Compact）算法。
     - 二者的本质差异在于标记-清除算法是一种**非移动式的回收算法**，标记-压缩是**移动式的**。是否移动回收后的存活对象是一项优缺点并存的风险决策。
     - 可以看到，标记的存活对象将会被整理，按照内存地址依次排列，而未被标记的内存会被清理掉。如此一来，当我们需要给新对象分配内存时，JVM 只需要持有一个内存的起始地址即可，这比维护一个空闲列表显然少了许多开销。

   - 优点

     - 消除了标记一清除算法当中，内存区域分散的缺点，我们需要给新对象分配内存时，JVM只 需要持有一个内存的起始地址即可。
     - 消除了复制算法当中，内存减半的高额代价。

   - 缺点

     - 从效率.上来说，标记一整理算法要低于复制算法。
     - 移动对象的同时，如果对象被其他对象引用，则还需要调整引用的地址。
     - 移动过程中，需要全程暂停用户应用程序。即： STW

   - 对比

     | 属性\算法  | 标记清除算法 | 复制算法 | 标记压缩算法 |
     | ---------- | ------------ | -------- | ------------ |
     | 时间复杂度 | 中           | 快       | 满           |
     | 空间复杂度 | 少           | 占用2倍  | 少           |
     | 内存碎片   | 有           | 无       | 无           |
     | 移动对象   | 否           | 是       | 是           |

### 3、分代收集算法

   - **为什么要使用分代收集算法**

     前面所有这些算法中，并没有一种算法可以完全替代其他算法，它们都具有自己独特的优势和特点。分代收集算法应运而生。分代收集算法，是基于这样一个事实：不同的对象的生命周期是不一样的。因此，**不同生命周期的对象可以采取不同的收集方式，以便提高回收效率**。一般是把堆分为新生代和老年代，这样就可以根据各个年代的特点使用不同的回收算法，以提高垃圾回收的效率。

- 目前几乎所有的GC都是采用分代收集（Generational Collecting） 算法执行垃圾回收的。

  在HotSpot中，基于分代的概念，GC所使用的内存回收算法必须结合年轻代和老年代各自的特点。

  - 年轻代（Young Gen）
    - 年轻代特点：区域相对老年代较小，对象生命周期短、存活率低，回收频繁。
    - 这种情况**复制算法**的回收整理，速度是最快的。复制算法的效率只和当前存活对象大小有关，因此很适用于年轻代的回收。而复制算法内存利用率不高的问题，通过 HotSpot 中的两个 survivor 的设计得到缓解。
     - 老年代（Tenured Gen）
       - 老年代特点：区域较大，对象生命周期长、存活率高，回收不及年轻代频繁。
       - 这种情况存在大量存活率高的对象，复制算法明显变得不合适。一般是由标记清除或者是标记整理的混合实现。
         - 标记阶段的开销与存活对象的数量成正比。
         - 清除阶段的开销与所管理区域的大小成正相关。
         - 压缩阶段的开销与存活对象的数据成正比。

- 以 HotSpot 中的 CMS 回收器为例，CMS 是基于标记清除实现的，对于对象的回收效率很高。而对于碎片问题，CMS 采用基于标记压缩算法的 Serial Old 回收器作为补偿措施：当内存回收不佳（碎片导致的执行失败时），将采用 Serial Old 执行 Full GC（标记整理算法）以达到对老年代内存的整理。

- 分代的思想被现有的虚拟机广泛使用。几乎所有的垃圾回收器都区分新生代和老年代。

### 4、增量收集算法

上述现有的算法，在垃圾回收过程中，应用软件将处于一种 Stop the World 的状态。在 Stop the World 状态下，应用程序所有的线程都会挂起，暂停一切正常的工作，等待垃圾回收的完成。如果垃圾回收时间过长，应用程序会被挂起很久，将严重影响用户体验或者系统的稳定性。为了解决这个问题，即对实时垃圾收集算法的研究直接导致了增量收集（Incremental Collecting） 算法的诞生。

1. 基本思想

   - 如果一次性将所有的垃圾进行处理，需要造成系统长时间的停顿，那么就可以让垃圾收集线程和应用程序线程交替执行。每次，**垃圾收集线程只收集一小片区域的内存空间，接着切换到应用程序线程。依次反复，直到垃圾收集完成**。
   - 总的来说，增量收集算法的基础仍是传统的标记清除和复制算法。增量收集算法**通过对线程间冲突的妥善处理，允许垃圾收集线程以分阶段的方式完成标记、清理或复制工作**。

2. 缺点

   使用这种方式，由于在垃圾回收过程中，间断性地还执行了应用程序代码，所以能减少系统的停顿时间。但是，因为线程切换和上下文转换的消耗，会使得垃圾回收的总体成本上升，造成系统吞吐量的下降。

### 5、分区算法

> 主要针对G1收集器来说的

- 一般来说，在相同条件下，堆空间越大，一次GC时所需要的时间就越长，有关GC产生的停顿也越长。为了更好地控制GC产生的停顿时间，将一块大的内存区域分割成多个小块，根据目标的停顿时间，每次合理地回收若干个小区间，而不是整个堆空间，从而减少一次GC所产生的停顿。

- 分代算法将按照对象的生命周期长短划分成两个部分，分区算法将整个堆空间划分成连续的不同小区间。每一个小区间都独立使用，独立回收。这种算法的好处是可以控制一次回收多少个小区间。
  ![16](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/90f361cf2d744ee4abec8beb0085daa0~tplv-k3u1fbpfcp-zoom-1.image)

### 5、最后

**这些只是基本的算法思路，实际GC实现过程要复杂的多，目前还在发展中的前沿GC都是复合算法，并且并行和并发兼备。**

## 三、相关概念

### 1、对象的 finalization 机制

**对象销毁前的回调函数：finalize()**

- Java 语言提供了对象终止（finalization）机制来允许开发人员提供**对象被销毁之前的自定义处理逻辑**。
- 当垃圾回收器发现没有引用指向一个对象，即：垃圾回收此对象之前，总会先调用这个对象的 finalize() 方法。
- finalize() 方法允许在子类中被重写，**用于在对象被回收时进行资源释放**。通常在这个方法中进行一些资源释放和清理的工作，比如关闭文件、套接字和数据库连接等。
- 永远不要主动调用某个对象的 finalize() 方法，应该交给垃圾回收机制调用。理由：**在 finalize() 时可能会导致对象复活**，**finalize() 方法的执行时间是没有保障的；它完全由GC线程决定，极端情况下，若不发生GC，则 finalize() 方法将没有执行机会**；**一个糟糕的 finalize() 会严重影响 GC 的性能**。
- 从功能上来说， finalize() 方法与 C++ 中的析构函数比较相似，但是Java采用的是基于垃圾回收器的自动内存管理机制，所以 finalize() 方法在本质，上不同于 C++ 中的析构函数。
- finalize() 方法对应了一个 finalize 线程，因为优先级比较低，即使主动调用该方法，也不会因此就直接进行回收

1. **对象是否“死亡”**

   如果从所有的根节点都无法访问到某个对象，说明对象己经不再使用了。一般来说，此对象需要被回收。但事实上，也并非是“非死不可”的，这时候它们暂时处于“缓刑”阶段。**一个无法触及的对象有可能在某一个条件下“复活”自己**，如果这样，那么对它的回收就是不合理的，为此，定义虚拟机中的对象可能的三种状态。如下：

   - **可触及的**：从根节点开始，可以到达这个对象。
   - **可复活的**：对象的所有引用都被释放，但是对象有可能在 finalize() 中复活。
   - **不可触及的**：对象的 finalize() 被调用，并且没有复活，那么就会进入不可触及状态。不可触及的对象不可能被复活，因为finalize() 只会被调用一一次。

   以上3种状态中，是由于 finalize() 方法的存在，进行的区分。只有在对象不可触及时才可以被回收。

2. **具体过程**

   判定一个对象 objA 是否可回收，至少要经历两次标记过程：

   - 如果对象 objA 到 GC Roots 没有引用链，则进行第一次标记。
   - 进行筛选，判断此对象是否有必要执行finalize()方法
     1. 如果对象 objA 没有重写 finalize() 方法，或者 finalize() 方法已经被虚拟机调用过，则虚拟机视为“没有必要执行”，objA 被判定为不可触及的。
     2. 如果对象 objA 重写了 finalize()方法，且还未执行过，那么 objA 会被插入到 F-Queue 队列中，由一个虚拟机自动创建的、低优先级的 Finalizer 线程触发其finalize()方法执行。
     3. finalize()方法是对象逃脱死亡的最后机会，稍后 GC 会对 F-Queue 队列中的对象进行第二次标记。如果 objA 在 finalize() 方法中与引用链上的任何一个对象建立了联系，那么在第二次标记时，objA 会被移出“即将回收”集合。之后，对象会再次出现没有引用存在的情况。在这个情况下，finalize() 方法不会被再次调用，对象会直接变成不可触及的状态，也就是说，一个对象的 finalize() 方法只会被调用一次。

3. 代码演示对象复活

   ```java
   /**
    * @author qinweizhao
    * @since 2022/4/4
    */
   public class CanReliveObject {
       
       /**
        * 类变量，属于 GC Root
        */
       public static CanReliveObject obj;
   
   
       /**
        * 此方法只能被调用一次
        * @throws Throwable t
        */
       @Override
       protected void finalize() throws Throwable {
           super.finalize();
           System.out.println("调用当前类重写的finalize()方法");
           //当前待回收的对象在finalize()方法中与引用链上的一个对象obj建立了联系
           obj = this;
       }
   
   
       public static void main(String[] args) {
           try {
               obj = new CanReliveObject();
               // 对象第一次成功拯救自己
               obj = null;
               System.gc();//调用垃圾回收器
               System.out.println("第1次 gc");
               // 因为Finalizer线程优先级很低，暂停2秒，以等待它
               Thread.sleep(2000);
               if (obj == null) {
                   System.out.println("obj is dead");
               } else {
                   System.out.println("obj is still alive");
               }
               System.out.println("第2次 gc");
               // 下面这段代码与上面的完全相同，但是这次自救却失败了
               obj = null;
               System.gc();
               // 因为Finalizer线程优先级很低，暂停2秒，以等待它
               Thread.sleep(2000);
               if (obj == null) {
                   System.out.println("obj is dead");
               } else {
                   System.out.println("obj is still alive");
               }
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
       }
   }
   ```

   **如果注释掉 finalize() 方法，结果：**

   >第1次 gc
   >obj is dead
   >第2次 gc
   >obj is dead

   **放开finalize()方法，输出结果：**

   > 第1次 gc
   > 调用当前类重写的finalize()方法
   > obj is still alive
   > 第2次 gc
   > obj is dead

   第一次自救成功，但由于 finalize() 方法只会执行一次，所以第二次自救失败。

### 2、System.gc 的理解

- 在默认情况下，通过 System.gc() 或者 Runtime.getRuntime().gc() 的调用，**会显式触发 Full GC**，同时对老年代和新生代进行回收，尝试释放被丢弃对象占用的内存。

- 然而 System.gc() 调用附带一个免责声明，无法保证对垃圾收集器的调用(不能确保立即生效)

- JVM实现者可以通过 System.gc() 调用来决定 JVM 的 GC 行为。而一般情况下，垃圾回收应该是自动进行的，**无须手动触发，否则就太过于麻烦了。**在一些特殊情况下，如我们正在编写一个性能基准，我们可以在运行之间调用System.gc()。 


**代码示例：手动执行 GC 操作**

  ```java
public class SystemGC {
    public static void main(String[] args) {
        new SystemGC();
        //提醒jvm的垃圾回收器执行gc,但是不确定是否马上执行gc 
        System.gc();
        //强制调用失去引用的对象的finalize()方法
        System.runFinalization();
    }

    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("SystemGCTest 重写了finalize()");
    }
}
  ```

输出结果不确定：有时候会调用 finalize() 方法，有时候并不会调用。

> SystemGCTest 重写了finalize()
> 或
> 空

**手动 GC 理解不可达对象的回收行为**

```java
public class LocalVarGC {
    public void localvarGC1() {

        byte[] buffer = new byte[10 * 1024 * 1024];//10MB
        System.gc();
        //输出: 不会被回收, FullGC时被放入老年代
        //[GC (System.gc()) [PSYoungGen: 14174K->10736K(76288K)] 14174K->10788K(251392K), 0.0089741 secs] [Times: user=0.01 sys=0.00, real=0.01 secs]
        //[Full GC (System.gc()) [PSYoungGen: 10736K->0K(76288K)] [ParOldGen: 52K->10649K(175104K)] 10788K->10649K(251392K), [Metaspace: 3253K->3253K(1056768K)], 0.0074098 secs] [Times: user=0.01 sys=0.02, real=0.01 secs]
    }

    public void localvarGC2() {
        byte[] buffer = new byte[10 * 1024 * 1024];
        buffer = null;
        System.gc();
        //输出: 正常被回收
        //[GC (System.gc()) [PSYoungGen: 14174K->544K(76288K)] 14174K->552K(251392K), 0.0011742 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
        //[Full GC (System.gc()) [PSYoungGen: 544K->0K(76288K)] [ParOldGen: 8K->410K(175104K)] 552K->410K(251392K), [Metaspace: 3277K->3277K(1056768K)], 0.0054702 secs] [Times: user=0.01 sys=0.00, real=0.01 secs]

    }

    public void localvarGC3() {
        {
            byte[] buffer = new byte[10 * 1024 * 1024];
        }
        System.gc();
        //输出: 不会被回收, FullGC时被放入老年代 局部变量表中索引为1的位置依然在 所以不会被回收
        //[GC (System.gc()) [PSYoungGen: 14174K->10736K(76288K)] 14174K->10784K(251392K), 0.0076032 secs] [Times: user=0.02 sys=0.00, real=0.01 secs]
        //[Full GC (System.gc()) [PSYoungGen: 10736K->0K(76288K)] [ParOldGen: 48K->10649K(175104K)] 10784K->10649K(251392K), [Metaspace: 3252K->3252K(1056768K)], 0.0096328 secs] [Times: user=0.01 sys=0.01, real=0.01 secs]
    }

    public void localvarGC4() {
        {
            byte[] buffer = new byte[10 * 1024 * 1024];
        }
        int value = 10;
        System.gc();
        //输出: 正常被回收 局部变量表中索引1的位置被value占用 
        //[GC (System.gc()) [PSYoungGen: 14174K->496K(76288K)] 14174K->504K(251392K), 0.0016517 secs] [Times: user=0.01 sys=0.00, real=0.00 secs]
        //[Full GC (System.gc()) [PSYoungGen: 496K->0K(76288K)] [ParOldGen: 8K->410K(175104K)] 504K->410K(251392K), [Metaspace: 3279K->3279K(1056768K)], 0.0055183 secs] [Times: user=0.00 sys=0.00, real=0.01 secs]
    }

    public void localvarGC5() {
        localvarGC1();
        System.gc();
        //输出: 正常被回收
        //[GC (System.gc()) [PSYoungGen: 14174K->10720K(76288K)] 14174K->10744K(251392K), 0.0121568 secs] [Times: user=0.02 sys=0.00, real=0.02 secs]
        //[Full GC (System.gc()) [PSYoungGen: 10720K->0K(76288K)] [ParOldGen: 24K->10650K(175104K)] 10744K->10650K(251392K), [Metaspace: 3279K->3279K(1056768K)], 0.0101068 secs] [Times: user=0.01 sys=0.02, real=0.01 secs]
        //[GC (System.gc()) [PSYoungGen: 0K->0K(76288K)] 10650K->10650K(251392K), 0.0005717 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
        //[Full GC (System.gc()) [PSYoungGen: 0K->0K(76288K)] [ParOldGen: 10650K->410K(175104K)] 10650K->410K(251392K), [Metaspace: 3279K->3279K(1056768K)], 0.0045963 secs] [Times: user=0.01 sys=0.00, real=0.00 secs]
    }

    public static void main(String[] args) {
        LocalVarGC local = new LocalVarGC();
        local.localvarGC5();
    }
}
```

### 3、内存溢出

1. 内存溢出相对于内存泄漏来说，尽管更容易被理解，但是同样的，内存溢出也是引发程序崩溃的罪魁祸首之一。
2. 由于GC一直在发展，所有一般情况下，除非应用程序占用的内存增长速度非常快，造成垃圾回收已经跟不上内存消耗的速度，否则不太容易出现OOM的情况。
3. 大多数情况下，GC会进行各种年龄段的垃圾回收，实在不行了就放大招，来一次独占式的Full GC操作，这时候会回收大量的内存，供应用程序继续使用。
4. Javadoc中对OutofMemoryError的解释是，没有空闲内存，并且垃圾收集器也无法提供更多内存。

**内存溢出（OOM）原因分析**

首先说没有空闲内存的情况：说明Java虚拟机的堆内存不够。原因有二：

1. Java虚拟机的堆内存设置不够。
   - 比如：可能存在内存泄漏问题；也很有可能就是堆的大小不合理，比如我们要处理比较可观的数据量，但是没有显式指定JVM堆大小或者指定数值偏小。我们可以通过参数-Xms 、-Xmx来调整。
2. 代码中创建了大量大对象，并且长时间不能被垃圾收集器收集（存在被引用）
   - 对于老版本的Oracle JDK，因为永久代的大小是有限的，并且JVM对永久代垃圾回收（如，常量池回收、卸载不再需要的类型）非常不积极，所以当我们不断添加新类型的时候，永久代出现OutOfMemoryError也非常多见。尤其是在运行时存在大量动态类型生成的场合；类似intern字符串缓存占用太多空间，也会导致OOM问题。对应的异常信息，会标记出来和永久代相关：“java.lang.OutOfMemoryError:PermGen space”。
   - 随着元数据区的引入，方法区内存已经不再那么窘迫，所以相应的OOM有所改观，出现OOM，异常信息则变成了：“java.lang.OutofMemoryError:Metaspace”。直接内存不足，也会导致OOM。

3. 这里面隐含着一层意思是，在抛出OutofMemoryError之前，通常垃圾收集器会被触发，尽其所能去清理出空间。
   - 例如：在引用机制分析中，涉及到JVM会去尝试**回收软引用指向的对象**等。
   - 在java.nio.Bits.reserveMemory()方法中，我们能清楚的看到，System.gc()会被调用，以清理空间。
4. 当然，也不是在任何情况下垃圾收集器都会被触发的
   - 比如，我们去分配一个超大对象，类似一个超大数组超过堆的最大值，JVM可以判断出垃圾收集并不能解决这个问题，所以直接抛出OutofMemoryError。

### 4、内存泄漏

1. 也称作“存储渗漏”。严格来说，**只有对象不会再被程序用到了，但是GC又不能回收他们的情况，才叫内存泄漏。**
2. 但实际情况很多时候一些不太好的实践（或疏忽）会导致对象的生命周期变得很长甚至导致OOM，也可以叫做宽泛意义上的“内存泄漏”。
3. 尽管内存泄漏并不会立刻引起程序崩溃，但是一旦发生内存泄漏，程序中的可用内存就会被逐步蚕食，直至耗尽所有内存，最终出现OutofMemory异常，导致程序崩溃。
4. 注意，这里的存储空间并不是指物理内存，而是指虚拟内存大小，这个虚拟内存大小取决于磁盘交换区设定的大小。

**常见例子**

1. 单例模式
   - 单例的生命周期和应用程序是一样长的，所以在单例程序中，如果持有对外部对象的引用的话，那么这个外部对象是不能被回收的，则会导致内存泄漏的产生。
2. 一些提供close()的资源未关闭导致内存泄漏
   - 数据库连接 dataSourse.getConnection()，网络连接socket和io连接必须手动close，否则是不能被回收的。

### 5、STW

1. Stop-the-World，简称STW，指的是GC事件发生过程中，会产生应用程序的停顿。**停顿产生时整个应用程序线程都会被暂停，没有任何响应**，有点像卡死的感觉，这个停顿称为STW。
2. 可达性分析算法中枚举根节点（GC Roots）会导致所有Java执行线程停顿，为什么需要停顿所有 Java 执行线程呢？
   - 分析工作必须在一个能确保一致性的快照中进行
   - 一致性指整个分析期间整个执行系统看起来像被冻结在某个时间点上
   - **如果出现分析过程中对象引用关系还在不断变化，则分析结果的准确性无法保证**
3. 被STW中断的应用程序线程会在完成GC之后恢复，频繁中断会让用户感觉像是网速不快造成电影卡带一样，所以我们需要减少STW的发生。

4. STW事件和采用哪款GC无关，所有的GC都有这个事件。
5. 哪怕是G1也不能完全避免Stop-the-world情况发生，只能说垃圾回收器越来越优秀，回收效率越来越高，尽可能地缩短了暂停时间。
6. STW是JVM在**后台自动发起和自动完成**的。在用户不可见的情况下，把用户正常的工作线程全部停掉。
7. 开发中不要用System.gc() ，这会导致Stop-the-World的发生。

**代码感受 Stop the World**

```
/**
 * @author qinweizhao
 * @since 2022/4/4
 */
public class StopTheWorld {
    public static class WorkThread extends Thread {
        List<byte[]> list = new ArrayList<byte[]>();

        @Override
        public void run() {
            try {
                while (true) {
                    for (int i = 0; i < 1000; i++) {
                        byte[] buffer = new byte[1024];
                        list.add(buffer);
                    }

                    if (list.size() > 10000) {
                        list.clear();
                        System.gc();//会触发full gc，进而会出现STW事件

                    }
                }
            } catch (Exception ex) {
                ex.printStackTrace();
            }
        }
    }

    public static class PrintThread extends Thread {
        public final long startTime = System.currentTimeMillis();

        @Override
        public void run() {
            try {
                while (true) {
                    // 每秒打印时间信息
                    long t = System.currentTimeMillis() - startTime;
                    System.out.println(t / 1000 + "." + t % 1000);
                    Thread.sleep(1000);
                }
            } catch (Exception ex) {
                ex.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        WorkThread w = new WorkThread();
        PrintThread p = new PrintThread();
        w.start();
        p.start();
    }
}
```

关闭工作线程 w ，观察输出：当前时间间隔与上次时间间隔**基本**是每隔1秒打印一次。

> 0.1
> 1.1
> 2.2
> 3.2
> 4.3
> 5.3
> 6.3
> 7.3

开启工作线程 w ，观察打印输出：当前时间间隔与上次时间间隔相差 1.3s ，可以明显感受到 Stop the World 的存在。

> 0.1
> 1.4
> 2.7
> 3.8
> 4.12
> 5.13

### 6、垃圾回收的并行与并发

**并发(Concurrent)**

- 在操作系统中，是指**一个时间段**中有几个程序都处于已启动运行到运行完毕之间，且这几个程序都是在同一个处理器上运行
- 并发不是真正意义上的“同时进行”，只是CPU把一个时间段划分成几个时间片段（时间区间），然后在这几个时间区间之间来回切换。由于CPU处理的速度非常快，只要时间间隔处理得当，即可让用户感觉是多个应用程序同时在进行

![image-20210704170519397](/Users/weizhao/Data/imageBed/image-20210704170519397.png)

**并行(Parallel)**

- 当系统有一个以上 CPU 时，当一个 CPU 执行一个进程时，另一个 CPU 可以执行另一个进程，两个进程互不抢占 CPU 资源，可以**同时**进行，我们称之为并行（Parallel）。
- 其实决定并行的因素不是 CPU 的数量，而是 CPU 的核心数量，比如一个 CPU 多个核也可以并行。
- 适合科学计算，后台处理等弱交互场景。

![](/Users/weizhao/Data/imageBed/image-20210704170543875.png)

**并发与并行的对比**

1. 并发，指的是多个事情，在同一时间段内同时发生了。
2. 并行，指的是多个事情，在同一时间点上（或者说同一时刻）同时发生了。
3. 并发的多个任务之间是互相抢占资源的。并行的多个任务之间是不互相抢占资源的。
4. 只有在多CPU或者一个CPU多核的情况中，才会发生并行。否则，看似同时发生的事情，其实都是并发执行的。

**垃圾回收的并发与并行**

- 并行（Parallel） ：指多条垃圾收集线程并行工作，但此时用户线程仍处于等待状态。

  - 如ParNew、 Parallel Scavenge、 Parallel 0ld；

- 串行（Serial）

  - 相较于并行的概念，单线程执行。
  - 如果内存不够，则程序暂停，启动JVM垃圾回收器进行垃圾回收。回收完，再启动程序的线程。

  ![image-20210704171216140](/Users/weizhao/Data/imageBed/image-20210704171216140.png)

- 并发（Concurrent） ：指用户线程与垃圾收集线程同时执行（但不一定是并行的，可能会交替执行），垃圾回收线程在执行时不会停顿用户程序的运行。

- - 用户程序在继续运行，而垃圾收集程序线程运行于另一个CPU上；
  - 如： CMS、G1

  ![5](/Users/weizhao/Data/imageBed/172daaae0010db3e)

### 7、安全点与安全区域

**安全点(SafePoint)**

- 程序执行时并非在所有地方都能停顿下来开始GC，只有在特定的位置才能停顿下来开始GC，这些位置称为“安全点（Safepoint）”。
- Safe Point的选择很重要，**如果太少可能导致GC等待的时间太长，如果太频繁可能导致运行时的性能问题**。大部分指令的执行时间都非常短暂，通常会根据“**是否具有让程序长时间执行的特征**”为标准。比如：选择一些执行时间较长的指令作为Safe Point，**如方法调用、循环跳转和异常跳转等**。

**如何在GC发生时，检查所有线程都跑到最近的安全点停顿下来呢？**

- 抢先式中断： （目前没有虚拟机采用） 首先中断所有线程。如果还有线程不在安全点，就恢复线程，让线程跑到安全点。
- 主动式中断： 设置一个中断标志，各个线程运行到Safe Point的时候主动轮询这个标志，如果中断标志为真，则将自己进行中断挂起。

**安全区域(Safe Region)**

- Safepoint 机制保证了程序执行时，在不太长的时间内就会遇到可进入GC的Safepoint。但是，程序“不执行”的时候呢？
- 例如线程处于Sleep状态或Blocked 状态，这时候线程无法响应JVM的中断请求，“走”到安全点去中断挂起，JVM也不太可能等待线程被唤醒。对于这种情况，就需要安全区域（Safe Region）来解决。
- **安全区域是指在一段代码片段中，对象的引用关系不会发生变化，在这个区域中的任何位置开始GC都是安全的**。我们也可以把Safe Region看做是被扩展了的Safepoint。

**实际执行时:**

- 当线程运行到Safe Region的代码时，首先标识已经进入了Safe Region，如果这段时间内发生GC，JVM会忽略标识为Safe Region状态的线程
- 当线程即将离开Safe Region时，会检查JVM是否已经完成根节点枚举（即GC Roots的枚举），如果完成了，则继续运行，否则线程必须等待直到收到可以安全离开Safe Region的信号为止；

### 6、Java中的引用

| 引用       | 引用存在 是否回收 | 应用场景     |
| ---------- | ----------------- | ------------ |
| 强引用     | 死也不回收        | 大部分       |
| 软引用     | 内存不足时回收    | 缓存         |
| 弱引用     | GC即回收          | 缓存         |
| 虚引用     |                   | GC时对象跟踪 |
| 终结器引用 |                   |              |

1. **强引用（StrongReference）**

   - 定义

     最传统的“引用”的定义，是指在程序代码之中普遍存在的引用赋值，**无论任何情况下，只要强引用关系还存在，垃圾收集器就永远不会回收掉被引用的对象**。

   - 特征

     - 在Java程序中，最常见的引用类型是强引用（普通系统99%以上都是强引用），也就是我们最常见的普通对象引用，也是默认的引用类型。
     - 当在Java语言中使用new操作符创建一个新的对象， 并将其赋值给一个变量的时候，这个变量就成为指向该对象的一个强引用。
     - 强引用的对象是可触及的，垃圾收集器就永远不会回收掉被引用的对象。
     - 对于一一个普通的对象，如果没有其他的引用关系，只要超过了引用的作用域或者显式地将相应（强）引用赋值为null，就是可以当做垃圾被收集了，当然具体回收时机还是要看垃圾收集策略。
     - 相对的，软引用、 弱引用和虚引用的对象是软可触及、弱可触及和虛可触及的，在一定条件下，都是可以被回收的。所以，强引用是造成Java内存泄漏的主要原因之一。

2. **软引用（SoftReference）**

   - 定义

     在系统将要发生内存溢出之前，将会把这些对象列入回收范围之中进行第二次回收。如果这次回收后还没有足够的内存，才会抛出内存溢出异常。

   - 特征

     - 软引用是用来描述一 些还有用，但非必需的对象。只被软引用关联着的对象，在系统将要发生内存溢出异常前，会把这些对象列进回收范围之中进行第二次回收，如果这次回收还没有足够的内存，才会抛出内存溢出异常。
     - **软引用通常用来实现内存敏感的缓存**。比如：高速缓存就有用到软引用。如果还有空闲内存，就可以暂时保留缓存，当内存不足时清理掉，这样就保证了使用缓存的同时，不会耗尽内存。
     - 垃圾回收器在某个时刻决定回收软可达的对象的时候，会清理软引用，并可选地把引用存放到一个引用队列（ Reference Queue）。
     - 类似弱引用，只不过Java虚拟机会尽量让软引用的存活时间长一些，迫不得.已才清理。
     - 在JDK 1. 2版之后提供了java.lang.ref.SoftReference类来实现软引用。

   - 代码测试软引用

     ```java
     /**
      * 软引用的测试：内存不足即回收
      * -Xms10m -Xmx10m -XX:+PrintGCDetails
      */
     public class SoftReferenceTest {
         public static class User {
             public User(int id, String name) {
                 this.id = id;
                 this.name = name;
             }
     
             public int id;
             public String name;
     
             @Override
             public String toString() {
                 return "[id=" + id + ", name=" + name + "] ";
             }
         }
     
         public static void main(String[] args) {
             //创建对象，建立软引用
     //        SoftReference<User> userSoftRef = new SoftReference<User>(new User(1, "songhk"));
             //上面的一行代码，等价于如下的三行代码
             User u1 = new User(1,"songhk");
             SoftReference<User> userSoftRef = new SoftReference<User>(u1);
             u1 = null;//取消强引用
     
     
             //从软引用中重新获得强引用对象
             System.out.println("GC之前软引用获取的对象" + userSoftRef.get());
     
             System.gc();
     
             //垃圾回收之后获得软引用中的对象
             System.out.println("GC之后软引用获取的对象" + userSoftRef.get());//由于堆空间内存足够，所有不会回收软引用的可达对象。
     
             try {
                 //让系统认为内存资源紧张、不够
     //            byte[] b = new byte[1024 * 1024 * 7];
                 byte[] b = new byte[1024 * 7168 - 399 * 1024];//恰好能放下数组又放不下u1的内存分配大小 不会报OOM
             } catch (Throwable e) {
                 e.printStackTrace();
             } finally {
                 //再次从软引用中获取数据
                 System.out.println("资源不足之后GC：软引用获取的对象" + userSoftRef.get());//在报OOM之前，垃圾回收器会回收软引用的可达对象。
             }
         }
     }
     ```

3. **弱引用（WeakReference）**

   - 定义

     被弱引用关联的对象只能生存到下一次垃圾收集之前。当垃圾收集器工作时，无论内存空间是否足够，都会回收掉被弱引用关联的对象。

   - 特征

     - 弱引用也是用来描述那些非必需对象，被弱引用关联的对象只能生存到下一次垃圾收集发生为止。在系统GC时，只要发现弱引用，不管系统堆空间使用是否充足，都会回收掉只被弱引用关联的对象。
     - 但是，由于垃圾回收器的线程通常优先级很低，因此，并不一 定能很快地发现持有弱引用的对象。在这种情况下，弱引用对象可以存在较长的时间。
     - 弱引用和软引用一样，在构造弱引用时，也可以指定一个引用队列，当弱引用对象被回收时，就会加入指定的引用队列，通过这个队列可以跟踪对象的回收情况。
     - 软引用、弱引用都非常适合来保存那些可有可无的缓存数据。
       - 如果这么做，当系统内存不足时，这些缓存数据会被回收，不会导致内存溢出。
       - 而当内存资源充足时，这些缓存数据又可以存在相当长的时间，从而起到加速系统的作用。
     - 在JDK1.2版之后提后了java.lang.ref.WeakReference类来实现弱引用
     - 弱引用对象与软引用对象的最大不同就在于，当GC在进行回收时，需要通过算法检查是否回收软引用对象，而对于弱引用对象，GC总是进行回收。弱引用对象更容易、更快被GC回收。

   - 代码测试

     ```java
     public class WeakReferenceTest {
     
         public static void main(String[] args) {
             //构造了弱引用
             WeakReference<User> userWeakRef = new WeakReference<User>(new User(1, "songhk"));
             //从弱引用中重新获取对象
             System.out.println(userWeakRef.get());
     
             System.gc();
             // 不管当前内存空间足够与否，都会回收它的内存
             System.out.println("After GC:");
             //重新尝试从弱引用中获取对象
             System.out.println(userWeakRef.get());
         }
     
         public static class User {
             public User(int id, String name) {
                 this.id = id;
                 this.name = name;
             }
     
             public int id;
             public String name;
     
             @Override
             public String toString() {
                 return "[id=" + id + ", name=" + name + "] ";
             }
         }
     }
     ```

4. **虚引用（PhantomReference）**

   - 定义

     一个对象是否有虛引用的存在，完全不会对其生存时 间构成影响，也无法通过虚引用来获得一个对象的实例。**为一个对象设置虛引用关联的唯一目的就是能在这个对象被收集器回收时收到一个系统通知(回收跟踪)**。

   - 特征

     - 虚引用(Phantom Reference),也称为“幽灵引用”或者“幻影引用”，是所有引用类型中最弱的一个。
     - 一个对象是否有虚引用的存在，完全不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它和没有引用几乎是一样的，随时都可能被垃圾回收器回收。
     - 它不能单独使用，也无法通过虚引用来获取被引用的对象。当试图通过虚引用的get（）方法取得对象时，总是null。
     - **为一个对象设置虚引用关联的唯一目的在于跟踪垃圾回收过程**。比如：能在这个对象被收集器回收时收到一个系统通知。
     - 虚引用必须和引用队列一起使用。虚引用在创建时必须提供一个引用队列作为参数。当垃圾回收器准备回收一个对象时，如果发现它还有虛引用，就会在回收对象后，将这个虚引用加入引用队列，以通知应用程序对象的回收情况。
     - 由于虚引用可以跟踪对象的回收时间，因此，也可以将一些资源释放操作放置在虛引用中执行和记录。
     - 在JDK 1. 2版之后提供了PhantomReference类来实现虚引用。

   - 代码测试

     ```java
     public class PhantomReferenceTest {
         public static PhantomReferenceTest obj;//当前类对象的声明
         static ReferenceQueue<PhantomReferenceTest> phantomQueue = null;//引用队列
     
         public static class CheckRefQueue extends Thread {
             @Override
             public void run() {
                 while (true) {
                     if (phantomQueue != null) {
                         PhantomReference<PhantomReferenceTest> objt = null;
                         try {
                             objt = (PhantomReference<PhantomReferenceTest>) phantomQueue.remove();
                         } catch (InterruptedException e) {
                             e.printStackTrace();
                         }
                         if (objt != null) {
                             System.out.println("追踪垃圾回收过程：PhantomReferenceTest实例被GC了");
                         }
                     }
                 }
             }
         }
     
         @Override
         protected void finalize() throws Throwable { //finalize()方法只能被调用一次！
             super.finalize();
             System.out.println("调用当前类的finalize()方法");
             obj = this;
         }
     
         public static void main(String[] args) {
             Thread t = new CheckRefQueue();
             t.setDaemon(true);//设置为守护线程：当程序中没有非守护线程时，守护线程也就执行结束。
             t.start();
     
             phantomQueue = new ReferenceQueue<PhantomReferenceTest>();
             obj = new PhantomReferenceTest();
             //构造了 PhantomReferenceTest 对象的虚引用，并指定了引用队列
             PhantomReference<PhantomReferenceTest> phantomRef = new PhantomReference<PhantomReferenceTest>(obj, phantomQueue);
     
             try {
                 //不可获取虚引用中的对象
                 System.out.println(phantomRef.get());
     
                 //将强引用去除
                 obj = null;
                 //第一次进行GC,由于对象可复活，GC无法回收该对象
                 System.gc();
                 Thread.sleep(1000);
                 if (obj ** null) {
                     System.out.println("obj 是 null");
                 } else {
                     System.out.println("obj 可用");
                 }
                 System.out.println("第 2 次 gc");
                 obj = null;
                 System.gc(); //一旦将obj对象回收，就会将此虚引用存放到引用队列中。
                 Thread.sleep(1000);
                 if (obj ** null) {
                     System.out.println("obj 是 null");
                 } else {
                     System.out.println("obj 可用");
                 }
             } catch (InterruptedException e) {
                 e.printStackTrace();
             }
         }
     }
     
     ```

     输出：

     > null
     > 调用当前类的finalize()方法
     > obj 可用
     > 第 2 次 gc
     > 追踪垃圾回收过程：PhantomReferenceTest实例被GC了
     > obj 是 null

5. **终结器引用**

   - 定义
     - 它用以实现对象的finalize（）方法，也可以称为终结器引用。
     - 无需手动编码， 其内部配合引用队列使用。
     - 在GC时， 终结器引用入队。由Finalizer线程通过终结器引用找到被引用对象并调用它的finalize（）方法，第二次GC时才能回收被引用对象。

## 四、垃圾回收器

垃圾收集器没有在规范中进行过多的规定，可以由不同的厂商、不同版本的JVM来实现。由于 JDK 的版本处于高速迭代过程中，因此 Java 发展至今已经衍生了众多的 GC 版本。从不同角度分析垃圾收集器，可以将 GC 分为不同的类型。

### 1、分类

1. **按线程数分**

   - **串行垃圾回收器**：单 CPU，配置较低，只有一条 GC 线程 。

   - **并行垃圾回收器**：并发较强的 CPU 多条GC 线程 。

     ![image-20210705091315353](/Users/weizhao/Data/imageBed/image-20210705091315353.png)

2. 按照工作模式

   - **并发式垃圾回收器**

     与应用程序线程交替工作，以尽可能减少应用程序的停顿时间。

   - **独占式垃圾回收器**

     一旦运行，就停止应用程序中的所有用户线程，直到垃圾回收过程完全结束。

3. 按碎片处理方式

   - **压缩式垃圾回收器**

     ​	回收完成后，对存活对象进行压缩整理，消除回收后的碎片。

   - **非压缩式垃圾回收器**

     ​	非回收碎片，再创建对象使用空闲列表。

4. 按工作的内存区间分

   - **年轻代垃圾回收器**
   - **老年代垃圾回收器**

### 2、性能指标

- **吞吐量：运行用户代码的时间占总运行时间的比例**

  - 吞吐量就是 CPU 用于运行用户代码的时间与 CPU 总消耗时间的比值，即吞吐量=运行用户代码时间/ （运行用户代码时间+垃圾收集时间）。比如：虚拟机总共运行了100分钟，其中垃圾收集花掉1分钟，那吞吐量就是99%。
  - 这种情况下，应用程序能容忍较高的暂停时间，因此，高吞吐量的应用程序有更长的时间基准，快速响应是不必考虑的。
  - 吞吐量优先，意味着在单位时间内，STW 的时间最短： 0.2 + 0.2 = 0.4


  ![image-20210705095526753](/Users/weizhao/Data/imageBed/image-20210705095526753.png)

- 垃圾收集开销：吞吐量的补数，垃圾收集所用时间与总运行时间的比例。

- 暂停时间：执行垃圾收集时，程序的工作线程被暂停的时间。

  - “暂停时间”是指一个时间段内应用程序线程暂停，让GC线程执行的状态。例如，GC期间100毫秒的暂停时间意味着在这100毫秒期间内没有应用程序线程是活动的。.

  - 暂停时间优先，意味着尽可能让单次STW的时间最短： 0.1 + 0.1 + 0.1 + 0.1+0.1=0.5

    ![image-20210705095558002](/Users/weizhao/Data/imageBed/image-20210705095558002.png)

- 收集频率：相对于应用程序的执行，收集操作发生的频率。

- 内存占用： Java堆区所占的内存大小。

- 快速：一个对象从诞生到被回收所经历的时间。

吞吐量、暂停时间、内存占用这三者共同构成一个“不可能三角”。三者总体的表现会随着技术进步而越来越好。一款优秀的收集器通常最多同时满足其中的两项。这三项里，暂停时间的重要性日益凸显。因为随着硬件发展，内存占用多些越来越能容忍，硬件性能的提升也有助于降低收集器运行时对应用程序的影响，即提高了吞吐量。而内存的扩大，对延迟反而带来负面效果。简单来说，主要抓住两点：吞吐量、暂停时间。

**吞吐量 vs 暂停时间**

1. **高吞吐量较好**因为这会让应用程序的最终用户感觉只有应用程序线程在做“生产性”工作。直觉上，吞吐量越高程序运行越快。
2. 低暂停时间（低延迟）较好，是从最终用户的角度来看，不管是GC还是其他原因导致一个应用被挂起始终是不好的。这取决于应用程序的类型，有时候甚至短暂的200毫秒暂停都可能打断终端用户体验。因此，具有较低的暂停时间是非常重要的，特别是对于一个交互式应用程序（就是和用户交互比较多的场景）。
3. 不幸的是”高吞吐量”和”低暂停时间”是一对相互竞争的目标（矛盾）。因为如果选择以吞吐量优先，那么**必然需要降低内存回收的执行频率**，但是这样会导致GC需要更长的暂停时间来执行内存回收。相反的，如果选择以低延迟优先为原则，那么为了降低每次执行内存回收时的暂停时间，也只能频繁地执行内存回收，但这又引起了年轻代内存的缩减和导致程序吞吐量的下降。
4. 在设计（或使用）GC算法时，我们必须确定我们的目标：一个GC算法只可能针对两个目标之一（即只专注于较大吞吐量或最小暂停时间），或尝试找到一个二者的折衷。
5. 现在标准：**在最大吞吐量优先的情况下，降低停顿时间**。

### 3、概述

1.  发展史

- 1999年随JDK1.3.1一 起来的是串行方式的Serial GC，它是第一款GC。ParNew垃圾收集器是Serial收集器的多线程版本

- 2002年2月26日，Parallel GC和Concurrent Mark Sweep GC跟随JDK1.4.2一起发布

- Parallel GC在JDK6之后成为HotSpot默认GC。

- 2012年，在JDK1.7u4版本中，G1可用。

- 2017年，JDK9中G1变成默认的垃圾收集器，以替代CMS。

- 2018年3月，JDK10中G1垃圾回收器的并行完整垃圾回收，实现并行性来改善最坏情况下的延迟。

----------------- 分水岭 ---------------------

- 2018年9月，JDK11发布。引入Epsilon垃圾回收器，又被称为"No一0p （无操作） "回收器。同时，引入ZGC：可伸缩的低延迟垃圾回收器（Experimental）。

- 2019年3月，JDK12发布。 增强G1，自动返回未用堆内存给操作系统。同时，引入Shenandoah GC：低停顿时间的GC （Experimental）。

- 2019年9月，JDK13发布。增强ZGC，自动返回未用堆内存给操作系统。

- 2020年3月，JDK14发布。删除CMS垃圾回收器。扩展ZGC在macOS和Windows.上的应用

2.  七款经典的垃圾回收器

- 串行回收器：Serial. Serial Old 
- 并行回收器：ParNew. Parallel Scavenge. Parallel Old
- 并发回收器：CMS. G1
- ![image-20210705101934780](/Users/weizhao/Data/imageBed/image-20210705101934780.png)

3.  七款经典的垃圾收集器与垃圾分代之间的关系

- 新生代收集器： Serial、 ParNeW、Parallel Scavenge
- 老年代收集器： Serial 0ld、 Parallel 0ld、 CMS
- 整堆收集器： G1

![image-20210705102026012](/Users/weizhao/Data/imageBed/image-20210705102026012.png)

4. 垃圾收集器的组合关系

![image-20210705102141253](/Users/weizhao/Data/imageBed/image-20210705102141253.png)

**注意：**

- JDK1.3到1.5默认使用SerialGC + SerialOldGC
- JDK1.6到1.8默认是ParallerlGC + parallerlOldGC
- JDK9及以后默认使用G1
- CMS在9之后过期，14之后被移除

**说明：**

1. 两个收集器间有连线，表明它们可以搭配使用。
   - Serial/Serial old
   - Serial/CMS （JDK9废弃）
   - ParNew/Serial Old （JDK9废弃）
   - ParNew/CMS
   - Parallel Scavenge/Serial Old （预计废弃）
   - Parallel Scavenge/Parallel Old
   - G1
2. 其中Serial 0ld作为CMS 出现"Concurrent Mode Failure"失败的后 备预案。 
3. （红色虚线）由于维护和兼容性测试的成本，JDK8过时，JDK 9中移除。
4. （绿色虚线）JDK 14中：过时 未来会移除
5. （青色虚线）JDK 14中：删除CMS垃圾回收器 

- 为什么要有很多收集器个不够吗？ 因为Java的使用场景很多， 移动端，服务器等。所以就需要针对不同的场景，提供不同的垃圾收集器，提高垃圾收集的性能。
- 虽然我们会对各个收集器进行比较，但并非为了挑选一个最好的收集器出来。没有一种放之四海皆准、任何场景下都适用的完美收集器存在，更加没有万能的收集器。所以我们选择的只是对具体应用最合适的收集器

5. 查看默认的垃圾收集器
   - -XX：+PrintCommandLineFlags： 查看命令行相关参数（包含使用的垃圾收集器）
     - JDK1.8结果:  +UseParallelGC  +UseParallelOldGC
     - JDK11结果：+UseG1GC
   - 使用命令行指令： jinfo -flag 相关垃圾回收器参数进程 ID

### 4、Serial 回收器

**Serial 回收器：串行回收**

1. **SerialGC**

   Serial 收集器是最基本、历史最悠久的垃圾收集器了。JDK1.3之前回收新生代唯一的选择。作为HotSpot中Client模式下的默认新生代垃圾收集器。**Serial收集器采用复制算法、串行回收和"Stop一 the一World"机制的方式执行内存回收。** 

2. **SerialOldGC**

   除了年轻代之外，Serial收集器还提供用于执行老年代垃圾收集的Serial 0ld收集器。 Serial Old收集器同样也采用了串行回收 和"Stop the World"机制，只不过内存回收算法使用的是**标记压缩算法**。Serial 0ld是运行在Client模式下默认的老年代的垃圾回收器。Serial 0ld在服务端（Linux）下主要有两个用途：与新生代的ParallelScavenge配合使用、作为老年代CMS收集器的后备垃圾收集方案。

3. **特点**

   这个收集器是一个单线程的收集器，但它的“单线程”的意义并不仅仅会使用一个CPU或一条收集线程去完成垃圾收集工作，更重要的是在它进行垃圾收集时，必须暂停其他所有的工作线程，直到它收集结束（Stop The World ）。

   ![image-20210705201814064](/Users/weizhao/Data/imageBed/image-20210705201814064.png)

4. **优势**
   
   简单而高效（与其他收集器的单线程比），对于限定单个CPU的环境来说，Serial 收集器由于没有线程交互的开销，专心做垃圾收集自然可以获得最高的单线程收集效率。（在用户的桌面应用场景中，可用内存一般不大（几十MB至一两百MB）， 可以在较短时间内完成垃圾收集（几十ms至一百多ms） ，只要不频繁发生，使用串行回收器是可以接受的）。
   
5. 总结
   
   这种垃圾收集器了解即可，现在已经不用串行的了。而且在限定单核 cpu 才可以用。现在都不是单核的了。对于交互较强的应用而言，这种垃圾收集器是不能接受的。一般在 Java Web应用程序中是不会采用串行垃圾收集器的。在HotSpot虛拟机中，使用`-XX：+UseSerialGC `表示年轻代使用 SerialGC 和老年代使用 SerialOld GC 收集器。

### 5、ParNew回收器

> ParNew是上一个 Serial 的多线程版本**（并行回收）**

ParNew 收集器除了采用并行回收的方式执行内存回收外，两款垃圾收集器之间几乎没有任何区别。ParNew 收集器在年轻代中同样也是采用**复制算法**、老年代同样采用**标记压缩算法**。"Stop一 the一World"机制。ParNew 是很多 JVM 运行在 Server 模式下新生代的默认垃圾收集器。

![image-20210705203727535](/Users/weizhao/Data/imageBed/image-20210705203727535.png)

对于新生代，回收次数频繁，使用并行方式高效。对于老年代，回收次数少，使用串行方式节省资源（CPU并行 需要切换线程，串行可以省去切换线程的资源）。

1. ParNew 收集器的回收效率在任何场景下都会比 Serial 收集器更高效？

ParNew 收集器运行在多CPU的环境下，由于可以充分利用多CPU、 多核心等物理硬件资源优势，可以更快速地完成垃圾收集，提升程序的吞吐量。但是在单个CPU的环境下，ParNew收 集器不比 Serial 收集器更高效。虽然 Serial 收集器是基于串行回收，但是由于CPU不需要频繁地做任务切换，因此可以有效避免多线程交互过程中产生的一些额外开销。

2. 设置 ParNew 垃圾回收器

在程序中，开发人员可以通过选项`-XX： +UseParNewGC`手动指定使用 ParNew 收集器执行内存回收任务。它表示年轻代使用并行收集器，不影响老年代。`-XX：ParallelGCThreads`限制线程数量，默认开启和 CPU 数据相同的线程数。

###  6、Parallel回收器

Parallel Scavenge 收集器的目标是**达到一个可控制的吞吐量**（Throughput），它也被称为吞吐量优先的垃圾收集器。高吞吐量则可以高效率地利用CPU 时间，尽快完成程序的运算任务，主 要适合在后台运算而不需要太多交互的任务。因此，常见在服务器环境中使用。**自适应调节策略**也是Parallel Scavenge 与 ParNew 一个重要区别。

Parallel 收集器在 JDK1.6 时提供了用于执行老年代垃圾收集的 Parallel 0ld收集器，用来代替老年代的 Serial 0ld 收集器。Parallel 0ld 收集器采用了**标记一压缩**算法，但同样也是基于并行回收和”Stop一the一World"机制。在程序吞吐量优先的应用场景中，Parallel 收集器和Parallel 0ld收集器的组合，在Server模式下的内存回收性能很不错。**在Java8中，默认是此垃圾收集器**

![image-20210705204714966](/Users/weizhao/Data/imageBed/image-20210705204714966.png)

1. **参数设置**

- -XX:+UseParallelGC 手动指定年轻代使用 Parallel 并行收集器执行内存回收任务。

- -XX:+UseParallelOldGC：手动指定老年代都是使用并行回收收集器。

  上面两个参数分别适用于新生代和老年代。默认jdk8是开启的。默认开启一个，另一个也会被开启。（互相激活）

- -XX:ParallelGCThreads：设置年轻代并行收集器的线程数。一般地，最好与CPU数量相等，以避免过多的线程数影响垃圾收集性能。在默认情况下，当CPU数量小于8个， ParallelGCThreads 的值等于 CPU 数量。当 CPU 数量大于8个，ParallelGCThreads的值等于3+[5*CPU_Count]/8]

- -XX:MaxGCPauseMillis 设置垃圾收集器最大停顿时间（即STW的时间）。单位是毫秒。为了尽可能地把停顿时间控制在XX:MaxGCPauseMillis 以内，收集器在工作时会调整Java堆大小或者其他一些参数。对于用户来讲，停顿时间越短体验越好。但是在服务器端，我们注重高并发，整体的吞吐量。所以服务器端适合Parallel，进行控制。**该参数使用需谨慎。**

- -XX:GCTimeRatio垃圾收集时间占总时间的比例，即等于 1 / (N+1) ，用于衡量吞吐量的大小。取值范围(0, 100)。默认值99，也就是垃圾回收时间占比不超过1。与前一个-XX:MaxGCPauseMillis 参数有一定矛盾性，STW 暂停时间越长，Radio 参数就容易超过设定的比例。

- -XX:+UseAdaptiveSizePolicy 设置Parallel Scavenge收集器具有**自适应调节策略**。在这种模式下，年轻代的大小、Eden和Survivor的比例、晋升老年代的对象年龄等参数会被自动调整，已达到在堆大小、吞吐量和停顿时间之间的平衡点。在手动调优比较困难的场合，可以直接使用这种自适应的方式，仅指定虚拟机的最大堆、目标的吞吐量（GCTimeRatio）和停顿时间（MaxGCPauseMillis），让虚拟机自己完成调优工作。

### 7、CMS回收器

CMS收集器的关注点是`尽可能缩短垃圾收集时用户线程的停顿时间`。停顿时间越短（**低延迟**）就越适合与用户交互的程序，良好的响应速度能提升用户体验。在 JDK1.5时期， HotSpot推出了一款在强交互应用中几乎可认为有划 时代意义的垃圾收集器： CMS - （Concurrent -Mark -Sweep）收集器，这款收集器是HotSpot虚拟机中第一款真正意义上的并发收集器，它第一次实现了让垃圾收集线程与用户线程同时工作。目前很大一部分的Java应用集中在互联网站或者B/S系统的服务端上，这类应用尤其重视服务的响应速度，希望系统停顿时间最短，以给用户带来较好的体验。CMS收集器就非常符合这类应用的需求。不幸的是，CMS 作为老年代的收集器，却无法与 JDK 1.4.0 中已经存在的新生代收集器Parallel Scavenge配合工作，所以在JDK 1. 5中使用CMS来收集老年代的时候，新生代只能选择ParNew或者Serial收集器中的一个。在G1出现之前，CMS使用还是非常广泛的。一直到今天，仍然有很多系统使用CMS GC。

1. 工作原理（过程）

   CMS整个过程比之前的收集器要复杂，整个过程分为4个主要阶段，即初始标记阶段、并发标记阶段、重新标记阶段和并发清除阶段。(涉及STW的阶段主要是：初始标记 和 重新标记)

   ![image-20210706093959717](/Users/weizhao/Data/imageBed/image-20210706093959717.png)

- 初始标记（Initial一Mark） 阶段：在这个阶段中，程序中所有的工作线程都将会因为. “Stop一the一World"机制而出现短暂的暂停，这个阶段的主要任务仅仅只是**标记出GCRoots能直接关联到的对象**。
  一旦标记完成之后就会恢复之前被暂停的所有应用.线程。由于直接关联对象比较小，所以这里的速度非常快。
- 并发标记（Concurrent一Mark）阶段：`从GC Roots的 直接关联对象开始遍历整个对象图`的过程，这个过程耗时较长但是不需要停顿用户线程，可以与垃圾收集线程一起并发运行。
- 重新标记（Remark） 阶段：由于在并发标记阶段中，程序的工作线程会和垃圾收集线程同时运行或者交叉运行，因此为了`修正并发标记期间，因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录`，这个阶段的停顿时间通常会比初始标记阶段稍长一些，
  但也远比并发标记阶段的时间短。
- 并发清除（ Concurrent一Sweep）阶段：此阶段**清理删除掉标记阶段判断的已经死亡的对象，释放内存空间。**由于不需要移动存活对象，所以这个阶段也是可以与用户线程同时并发的

2. 分析

   尽管CMS收集器采用的是并发回收（非独占式），**但是在其初始化标记和再次标记这两个阶段中仍然需要执行“Stop-the-World”机制**暂停程序中的工作线程，不过暂停时间并不会太长，因此可以说明目前所有的垃圾收集器都做不到完全不需要“Stop-the-World”，只是尽可能地缩短暂停时间。**由于最耗费时间的并发标记与并发清除阶段都不需要暂停工作，所以整体的回收是低停顿的**。另外，由于在垃圾收集阶段用户线程没有中断，所以在CMS回收过程中，还应该确保应用程序用户线程有足够的内存可用。因此，CMS收集器不能像其他收集器那样等到老年代几乎完全被填满了再进行收集，**而是当堆内存使用率达到某一阈值时，便开始进行回收**，以确保应用程序在CMS工作过程中依然有足够的空间支持应用程序运行。要是CMS运行期间预留的内存无法满足程序需要，就会出现一次**“Concurrent Mode Failure”** 失败，这时虚拟机将启动后备预案：临时启用Serial old收集器来重新进行老年代的垃圾收集，这样停顿时间就很长了。CMS收集器的垃圾收集算法采用的是**标记清除算法**，这意味着每次执行完内存回收后，由于被执行内存回收的无用对象所占用的内存空间极有可能是不连续的一些内存块，**不可避免地将会产生一些内存碎片**。那么CMS在为新对象分配内存空间时，将无法使用指针碰撞（Bump the Pointer）技术，而只能够选择空闲列表（Free List）执行内存分配。

3. **为什么 CMS 不采用标记-压缩算法呢？**

   当并发清除的时候，用 Compact整理内存的话，原来的用户线程使用的内存还怎么用呢？要保证用户线程能继续执行，前提的它运行的资源不受影响嘛。Mark Compact更适合“stop the world”这种场景下使用。

4. 优点：

   - 并发收集
   - 低延迟

5. 弊端：

   - **会产生内存碎片**，导致并发清除后，用户线程可用的空间不足。在无法分配大对象的情况下，不得不提前触发Full GC。
   - **CMS收集器对CPU资源非常敏感**。在并发阶段，它虽然不会导致用户停顿，但是会因为占用了一部分线程而导致应用程序变慢，总吞吐量会降低。
   - **CMS收集器无法处理浮动垃圾**。可能出现“Concurrent Mode Failure”失败而导致另一次Full GC的产生。在并发标记阶段由于程序的工作线程和垃圾收集线程是同时运行或者交叉运行的，**那么在并发标记阶段如果产生新的垃圾对象，CMS将无法对这些垃圾对象进行标记，最终会导致这些新产生的垃圾对象没有被及时回收，**从而只能在下一次执行GC时释放这些之前未被回收的内存空间。

6. 参数设置

   - -XX:+UseConcMarkSweepGC：手动指定使用CMS收集器执行内存回收任务。开启该参数后会自动将-XX:+UseParNewGC打开。即：ParNew（Young区）+CMS（Old区）+Serial Old（Old区备选方案）的组合。

   - -XX:CMSInitiatingOccupanyFraction：设置堆内存使用率的阈值，一旦达到该阈值，便开始进行回收。JDK5 及以前版本的默认值为68，即当老年代的空间使用率达到68%时，会执行一次CMS回收。JDK6 及以上版本默认值为92%。如果内存增长缓慢，则可以设置一个稍大的值，大的阀值可以有效降低 CMS 的触发频率，减少老年代回收的次数可以较为明显地改善应用程序性能。反之，如果应用程序内存使用率增长很快，则应该降低这个阈值，以避免频繁触发老年代串行收集器。因此通过该选项便可以有效降低Full GC的执行次数。

   - -XX:+UseCMSCompactAtFullCollection：用于指定在执行完Full GC后对内存空间进行压缩整理，以此避免内存碎片的产生。不过由于内存压缩整理过程无法并发执行，所带来的问题就是停顿时间变得更长了。

   - -XX:CMSFullGCsBeforeCompaction：设置在执行多少次Full GC后对内存空间进行压缩整理。

   - -XX:ParallelCMSThreads：设置CMS的线程数量。CMS默认启动的线程数是 (ParallelGCThreads + 3) / 4，ParallelGCThreads是年轻代并行收集器的线程数，可以当做是 CPU 最大支持的线程数。当CPU资源比较紧张时，受到CMS收集器线程的影响，应用程序的性能在垃圾回收阶段可能会非常糟糕。

### 8、G1回收器：分区/垃圾优先

G1是一个并行回收器，它把堆内存分割为很多不相关的区域（Region） （物理上 不连续的）。使用不同的Region来表示Eden、幸存者0区，幸存者1区，老年代等。G1 GC有计划地避免在整个Java 堆中进行全区域的垃圾收集。G1跟踪各个Region 里面的垃圾堆积的价值大小（回收所获得的空间大小以及回收所需时间），在后台维护一个优先列表，每次**根据允许的收集时间，优先回收价值最大的Region**。由于这种方式的侧重点在于区间中垃圾数目，所以我们给G1一个名字：垃圾优先（Garbage First） G1 （Garbage一First） 是一款面向服务端应用的垃圾收集器，主要针对配备多核CPU及大容量内存的机器，**以极高概率满足GC停顿时间的同时，还兼具高吞吐量的性能特征。**在JDK1. 9版本正式启用，被Oracle官方称为“全功能的垃圾收集器” 。与此同时，CMS已经在JDK 9中被标记为废弃（deprecated） 。在jdk8中还不是默认的垃圾回收器，需要使用一XX： +UseG1GC来启用。

1. 优势
   - 兼具并行与并发
     - 并行性： G1在回收期间，可以有多个GC线程同时工作，有效利用多核计算能力。此时用户线程STW
     - 并发性： G1拥有与应用程序交替执行的能力，部分工作可以和应用程序同时执行，因此，一般来说，不会在整个回收阶段发生完全阻塞应用程序的情况
   - 分代收集
     - 从分代上看，**G1依然属于分代型垃圾回收器**，它会区分年轻代和老年代，年轻代依然有Eden区和Survivor区。但从堆的结构，上看，它不要求整个Eden区、年轻代或者老年代都是连续的，也不再坚持固定大小和固定数量。
     - 将堆空间分为若干个区域（Region） ，这些区域中包含了逻辑上的年轻代和老年代。
     - 和之前的各类回收器不同，**它同时兼顾年轻代和老年代。**
   - 空间整合
     - CMS： “标记一清除”算法、内存碎片、若干次GC后进行一次碎片整理
     - G1将内存划分为一个个的region。 内存的回收是以region作为基本单位的.Region之间是复制算法，但整体上实际可看作是标记-压缩（Mark一Compact）算法，两种算法都可以避免内存碎片。这种特性有利于程序长时间运行，分配大对象时不会因为无法找到连续内存空间而提前触发下一次GC。尤其是当Java堆非常大的时候，G1的优势更加明显。
   - **可预测的停顿时间模型**
     - 这是G1相对于CMS的另一大优势，G1除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为M毫秒的时间片段内，消耗在垃圾收集上的时间不得超过N毫秒。
     - 由于分区的原因，G1可以只选取部分区域进行内存回收，这样缩小了回收的范围，因此对于全局停顿情况的发生也能得到较好的控制。
     - G1跟踪各个Region里面的垃圾堆积的价值大小（回收所获得的空间大小以 及回收所需时间的经验值），在后台维护一个优先列表，**每次根据允许的收集时间，优先回收价值最大的Region。保证了G1 收集器在有限的时间内可以获取尽可能高的收集效率。**
     - 相比于CMSGC，G1未必能做到CMS在最好情况下的延时停顿，但是最差情况要.好很多。

2. 缺点

   - 相较于CMS，G1还不具备全方位、压倒性优势。比如在用户程序运行过程中，G1无论是为了垃圾收集产生的内存占用（Footprint） 还是程序运行时的额外执行负载（overload） 都要比CMS要高。
   - 从经验上来说，在小内存应用上CMS的表现大概率会优于G1，而G1在大内存应用，上则发挥其优势。平衡点在6-8GB之间。

   G1回收器的过程

   - 年轻代GC （Young GC ）

   - 老年代并发标记过程（ Concurrent Marking）

   - 混合回收（Mixed GC ）

   - （如果需要，单线程、独占式、高强度的Full GC还是继续存在的。它针对GC的评估失败提供了一种失败保护机制，即强力回收。）

     ![image-20210818090636748](/Users/weizhao/Data/imageBed/image-20210818090636748.png)

3. 参数设置
   - -XX:+UseG1GC 手动指定使用G1收集器执行内存回收任务。
   - -XX:G1HeapRegionSize 设置每个Region的大小。值是2的幂，范围是1MB 到32MB之间，目标是根据最小的Java堆大小划分出约2048个区域。默认是堆内存的1/2000。
   - -XX:MaxGCPauseMillis 设置期望达到的最大Gc停顿时间指标（JVM会尽力实现，但不保证达到）。默认值是200ms
   - -XX:ParallelGCThread 设置sTw.工作线程数的值。最多设置为8
   - -XX:ConcGCThreads 设置并发标记的线程数。将n设置为并行垃圾回收线程数（ParallelGCThreads）的1/4左右。
   - -XX:InitiatingHeapOccupancyPercent 设置触发并发GC周期的Java堆占用率阈值。超过此值，就触发GC。默认值是45。

4. G1回收器的常见操作步骤

   G1的设计原则就是简化JVM性能调优，开发人员只需要简单的三步即可完成调优：

   - 第一步：开启G1垃圾收集器
   - 第二步：设置堆的最大内存
   - 第三步：设置最大的停顿时间

   G1中提供了三种垃圾回收模式： YoungGC、 Mixed GC和Full GC， 在不同的条件下被触发。

5. 适用场景
   - 面向服务端应用，针对具有大内存、多处理器的机器。
   - 最主要的应用是**需要低GC延迟，并具有大堆的应用程序提供解决方案；**
   - 如：在堆大小约6GB或更大时，可预测的暂停时间可以低于0.5秒； （ G1通过每次只清理一部分而不是全部的Region的增量式清理来保证每次GC停顿时间不会过长）。
   - 用来替换掉JDK1.5中的CMS收集器； 在下面的情况时，使用G1可能比CMS好：
     ①超过50%的Java堆被活动数据占用；
     ②对象分配频率或年代提升频率变化很大；
     ③GC停顿时间过长（长于0. 5至1秒）。
   - HotSpot垃圾收集器里，除了G1以外，其他的垃圾收集器使用内置的JVM线程执行 GC的多线程操作，而G1 GC可以采用应用线程承担后台运行的GC工作，即当JVM的GC线程处理速度慢时，系统会调用应用程序线程帮助加速垃圾回收过程。

6. 分区region,化整为零

   使用G1收集器时，它将整个Java堆划分成约2048个大小相同的独立Region块，每个Region块大小根据堆空间的实际大小而定，整体被控制在1MB到32MB之间，且为2的N次幂，即1MB， 2MB， 4MB， 8MB， 1 6MB， 32MB。可以通过一 XX：G1HeapRegionSize设定。所有的Region大小相同，且在JVM生命周期内不会被改变。虽然还保留有新生代和老年代的概念，但新生代和老年代不再是物理隔离的了，它们都是一部分Region （不需要连续）的集合。通过Region的动态分配方式实现逻辑_上的连续。


   ![image-20210706162729369](/Users/weizhao/Data/imageBed/image-20210706162729369.png)

   一个region 有可能属于Eden， Survivor 或者0ld/Tenured 内存区域。但是一个region只可能属于一个角色。图中的E表示该region属于Eden内存区域，s表示属于Survivor内存区域，0表示属于0ld内存区域。图中空白的表示未使用的内存空间。

   G1垃圾收集器还增加了一种新的内存区域，叫做Humongous内存区域，如图中的H块。主要用于存储大对象，如果超过1. 5个region，就放到H。设置H的原因：对于堆中的大对象，默认直接会被分配到老年代，但是如果它是一个短期存在的大对象，就会对垃圾收集器造成负面影响。为了解决这个问题，G1 划分了一个Humongous区，它用来专门存放大对象。如果一个H区装不下一个大对象，那么G1会寻找连续的H区来存储。为了能找到连续的H区，有时候不得不启动Full GC。G1 的大多数行为都把H区作为老年代的一部分来看待。


7. G1回收器垃圾回收过程

   G1 GC的垃圾回收过程主要包括如下三个环节：

   - 年轻代GC （Young GC ）

   - 老年代并发标记过程（ Concurrent Marking）

   - 混合回收（Mixed GC ）

     （如果需要，单线程、独占式、高强度的Full GC还是继续存在的。它针对GC的评估失败提供了一种失败保护机制，即强力回收。）

     ![image-20210706162819150](/Users/weizhao/Data/imageBed/image-20210706162819150.png)

     应用程序分配内存，**当年轻代的Eden区用尽时开始年轻代回收过程**； G1的年轻代收集阶段是一个并行的**独占式**收集器。在年轻代回收期，G1 GC暂停所有应用程序线程，启动多线程执行年轻代回收。然后从年轻代区间移动存活对象到Survivor区间或者老年区间，也有可能是两个区间都会涉及。当堆内存使用达到一定值（默认45%）时，开始老年代并发标记过程。标记完成马上开始混合回收过程。对于一个混合回收期，G1 GC从老年区间移动存活对象到空闲区间，这些空闲区间也就成为了老年代的一部分。和年轻代不同，老年代的G1回收器和其他GC不同，G1的老年代回收器不需要整个老年代被回收，一次只需要扫描/回收一小部分老年代的Region就可以了。同时，这个老年代Region是和年轻代一起 被回收的。举个例子：一个web服务器，Java进程最大堆内存为4G，每分钟响应1500个请求，每45秒钟会新分配大约2G的内存。G1会每45秒钟进行一次年轻代回收，每31 个小时整个堆的使用率会达到45%，会开始老年代并发标记过程，标记完成后开始四到五次的混合回收。

### 9、总结

截止JDK 1.8，一共有7款不同的垃圾收集器。

1.对比

![image-20210706164337791](/Users/weizhao/Data/imageBed/image-20210706164337791.png)

2.组合

![image-20210706164353493](/Users/weizhao/Data/imageBed/image-20210706164353493.png)

3. 如何选择

- **优先调整堆的大小让 JVM 自适应完成。**
- **如果内存小于100M，使用串行收集器。**
- **如果是单核、单机程序，并且没有停顿时间的要求，串行收集器。**
- **如果是多 CPU、需要高吞吐量、允许停顿时间超过1秒，选择并行或者 JVM 自己选择**
- **如果是多 CPU、追求低停顿时间，需快速响应（比如延迟不能超过1秒，如互联网应用），使用并发收集器。**
- **官方推荐 G1，性能高。现在互联网的项目，基本都是使用G1。**

4. GC 日志分析

   通过阅读GC日志，我们可以了解Java虛拟机内存分配与回收策略。内存分配与垃圾回收的参数列表

   -  -XX:+PrintGC 输出GC日志。 显示总的GC堆的变化

     - 打开GC日志：-verbose:gc

     - 这个只会显示总的GC堆的变化， 如下：

     - 输出

       ```java
       [GC （Allocation Failure） 80832K一>19298K（227840K），0.0084018 secs]
       [GC （Metadata GC Threshold） 109499K一>21465K （228352K），0.0184066 secs]
       [Full GC （Metadata GC Threshold） 21 465K一>16716K （201728K），0.0619261 secs ]
       ```

     - 解析

       ```java
       GC、Full GC： GC的类型，GC只在新生代上进行，Full GC包括永生代，新生代， 老年代。
       Allocation Failure： GC发生的原因。
       80832K一> 19298K：堆在GC前的大小和GC后的大小。
       228840k：现在的堆大小。
       0.0084018 secs： GC持续的时间。
       ```

   -  -XX： +PrintGCDetails 输出GC的详细日志

     - 输出

     ```java
     [GC （Allocation Failure） [ PSYoungGen： 70640K一> 10116K（141312K） ] 80541K一>20017K （227328K），0.0172573 secs] [Times： user=0.03 sys=0.00， real=0.02 secs ]
     [GC （Metadata GC Threshold） [PSYoungGen：98859K一>8154K（142336K） ] 108760K一>21261K （228352K），
     0.0151573 secs] [Times： user=0.00 sys=0.01， real=0.02 secs]
     [Full GC （Metadata GC Threshold） [PSYoungGen： 8154K一>0K（142336K） ] [ParOldGen： 13107K一>16809K（62464K） ] 21261K一>16809K （204800K），[Metaspace： 20599K一>20599K （1067008K） ]，0.0639732 secs]
     [Times： user=0.14 sys=0.00， real=0.06 secs]
     ```

     - 解析：

     ```java
     PSYoungGen：使用了Parallel Scavenge并行垃圾收集器的新生代GC前后大小的变化
     ParOldGen：使用了Parallel Old并行垃圾收集器的老年代Gc前后大小的变化
     Metaspace： 元数据区GC前后大小的变化，JDK1.8中引入了 元数据区以替代永久代
     xxx secs ： 指Gc花费的时间
     Times： user： 指的是垃圾收集器花费的所有CPU时间，sys： 花费在等待系统调用或系统事件的时间， real ：GC从开始到结束的时间，包括其他进程占用时间片的实际时间。
     ```

   -  -XX： +PrintGCTimeStamps 输出GC的时间戳（以基准时间的形式）

     - 打开GC日志： `一verbose：gc 一XX： +PrintGCDetails 一XX：+PrintGCTimeStamps 一 XX： +PrintGCDateStamps`
     - 输入信息如下：

     ```java
     2019一09一24T22：15：24.518+0800：3.287： [GC（Allocation Failure） [ PSYoungGen： 1361 62K一>5113K（136192K） ] 141425K一>17632K （222208K） ，0.0248249 secs] [Times： user=0.05sys=0.00， real=0.03 secs ]
     2019一09一24T22：15：25.559+0800：4.329： [ GC（Metadata GC Threshold）[PSYoungGen：97578K一>10068K（274944K） ] 110096K一>22658K （360960K），0.0094071 secs]
     [Times： user=0. 00sys=0.00， real=0. 01 secs]
     2019一09一24T22：15：25.569+0800：4.338： [Full GC （Metadata GC Threshold）[ PSYoungGen：10068K一>0K（274944K） ] [ ParoldGen： 12590K一>13564K （56320K） ] 22658K一>13564K （331264K） ，
     [Metaspace： 20590K一>20590K（1067008K）]， 0. 0494875 secs]
     [Times： user=0.17 sys=0. 02，real=0.05 secs ]     
     ```

   -  -XX： +PrintGCDateStamps输出GC的时间戳（以日期的形式，如2013一05一04T21 ： 53：59.234+0800 ）

   - -XX： +PrintHeapAtGC 在进行GC的前后打印出堆的信息

   -  -Xloggc：. . /logs/gc. log日志文件的输出路径
