# JVM-堆

## 一、概述

一个进程对应一个 JVM 实例，同时包含多个线程，这些**线程共享方法区和堆**，每个**线程独有程序计数器、本地方法栈和虚拟机栈**。

- 一个 JVM 实例只存在一个堆内存，堆也是 Java 内存管理的核心区域。
- Java 堆区在 JVM 启动的时候即被创建，其空间大小也就确定了。是 JVM 管理的最大一块内存空间（堆内存的大小是可以调节的）。
- 《Java虚拟机规范》规定，堆可以处于**物理上不连续**的内存空间中，但在**逻辑上它应该被视为连续的**。
- 所有的线程共享 Java 堆，在这里还可以划分**线程私有的缓冲区**（TLAB:Thread Local Allocation Buffer）。（面试问题：堆空间一定是所有线程共享的么？不是，TLAB线程在堆中独有的）。
- 《Java虚拟机规范》中对 Java 堆的描述是：所有的对象实例以及数组都应当在运行时分配在堆上；从实际使用角度看：**几乎**所有的对象实例都在堆分配内存，但并非全部。因为还有一些对象是在栈上分配的（逃逸分析，标量替换）。
- 数组或对象永远不会存储在栈上，因为栈帧中保存引用，这个引用指向对象或者数组在堆中的位置。
- 在方法结束后，堆中的对象不会马上被移除，仅仅在垃圾收集的时候才会被移除。
- 堆，是 GC(Garbage Collection，垃圾收集器)执行垃圾回收的重点区域。

> 随着 JVM 的迭代升级，原来一些绝对的事情，在后续版本中也开始有了特例，变的不再那么绝对。

## 二、内存

### 1、内存结构

- 存储在 JVM 中的 Java 对象可以被划分为两类：
  - 一类是生命周期较短的瞬时对象，这类对象的创建和消亡都非常迅速（存入新生代）。
  - 另外一类对象时生命周期非常长，在某些情况下还能与JVM的生命周期保持一致 (存入老年代)。
- Java堆区进一步细分可以分为年轻代（YoungGen）和老年代（OldGen） 。
  - 其中年轻代可以分为伊甸园区（Eden）、新生区1（from）和新生区2（to）。

- JDK7 及以前：

  - 逻辑：新生区（伊甸园区+新幸存者1区+幸存者2区）+ 养老区+永久区（方法区在1.7的实现）。

    ![2022-03-31_215004](https://img.qinweizhao.com/2022/03/2022-03-31_215004.png)

- JDK8 及以后

  - 逻辑：新生区（伊甸园区+新幸存者1区+幸存者2区）+养老区+元空间(直接内存)（方法区在1.8的实现）

### 2、参数设置

- **-Xms** 用于表示堆区的起始内存，（默认为物理内存的1/64）等价于 **-XX:InitialHeapSize**。
- **-Xmx** 则用于表示堆区的最大内存，默认为物理内存的1/4）等价于 **-XX:MaxHeapSize**。
- **-XX:NewRatio**: 配置新生代与老年代在堆结构的占比，**-XX:NewRatio=2**   表示老年代/新生代 默认为 2 。
- **-XX:SurvivorRatio** ：设置新生代中伊甸园区/幸存者区。默认值是 8。
- **-XX:-UseAdaptiveSizePolicy** ：关闭自适应的内存分配策略  （暂时用不到）。
- **-Xmn**：设置新生代的空间的大小。 （一般不设置）。
- **-XX:PrintFlagsInitial**: 查看所有参数的默认初始值。
 - **-XX:PrintFlagsFinal**：查看所有的参数的最终值（可能会存在修改，不再是初始值）。
 - **-Xmn:** 设置新生代大小（初始值及最大值）。
 - **-XX:MaxTenuringThreshold**：设置新生代垃圾的最大年龄(默认15)。
 - **-XX:+PrintGCDetails**：输出详细的GC处理日志。
 - 打印gc简要信息：
   - -XX:+PrintGC
   -  -verbose:gc
- **-XX:HandlePromotionFailure**：是否设置空间分配担保。

具体查看某个参数的指令：

- jps：查看当前运行中的进程
- jinfo -flag SurvivorRatio 进程id： 查看新生代中Eden和S0/S1空间的比例

**通常会将 -Xms 和 -Xmx 两个参数配置相同的值**，其目的就是为了能够在 Java 垃圾回收机制清理完堆区后不需要重新分隔计算堆区的大小，从而提高性能。当堆区中的内存大小超过 -Xmx所指定的最大内存时，将会抛出OOM异常；查看堆信息：-XX:+PrintGCDetails。

### 3、查看大小

```Java
/**
 *
 * 1. 设置堆空间大小的参数
 * -Xms 用来设置堆空间（年轻代+老年代）的初始内存大小
 *      -X 是jvm的运行参数
 *      ms 是memory start
 * -Xmx 用来设置堆空间（年轻代+老年代）的最大内存大小
 *
 * 2. 默认堆空间的大小
 *    初始内存大小：物理电脑内存大小 / 64
 *             最大内存大小：物理电脑内存大小 / 4
 * 3. 手动设置：-Xms600m -Xmx600m
 *     开发中建议将初始堆内存和最大的堆内存设置成相同的值。
 *
 * 4. 查看设置的参数：方式一： jps   /  jstat -gc 进程id
 *                  方式二：-XX:+PrintGCDetails
 * @author qinweizhao
 * @since 2022/4/3
 */
public class HeapSpaceInitial {
    public static void main(String[] args) {

        //返回Java虚拟机中的堆内存总量
        long initialMemory = Runtime.getRuntime().totalMemory() / 1024 / 1024;
        //返回Java虚拟机试图使用的最大堆内存量
        long maxMemory = Runtime.getRuntime().maxMemory() / 1024 / 1024;

        //-Xms : 243M
        System.out.println("-Xms : " + initialMemory + "M");

        //-Xmx : 3612M
        System.out.println("-Xmx : " + maxMemory + "M");

        //系统内存大小为：15.1875G
        System.out.println("系统内存大小为：" + initialMemory * 64.0 / 1024 + "G");
        //系统内存大小为：14.109375G
        System.out.println("系统内存大小为：" + maxMemory * 4.0 / 1024 + "G");

        try {
            Thread.sleep(1000000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

设置参数：`-Xms 600m -Xmx600m`。

终端输入: `jps` 显示当前进程:

```sh
jps
```

终端打印 7112 进程的 gc 情况：


```sh
jstat -gc 7112
```

![2022-03-31_215024](https://img.qinweizhao.com/2022/03/2022-03-31_215024.png)

打印出的结果将前七项相加除以1024，得到的为600m。

输出结果：

因为幸存者区 S0 和 S1 各占据了25m，但是他们始终有一个是空的，存放对象的是伊甸园区和某一个幸存者区。

```text
-Xms : 575M
-Xmx : 575M
```

**或：**设置参数 **-XX:+PrintGCDetails**。

## 三、对象分配

> 为新对象分配内存是件非常严谨和复杂的任务，JVM 的设计者们不仅需要考虑内存如何分配、在哪里分配的问题，并且由于内存分配算法与内存回收算法密切相关，所以还需要考虑 GC 执行完内存回收后是否会在内存空间中产生内存碎片。

### 1、分配过程

**一般情况：**

1. new 的对象先放伊甸园区。此区有大小限制。
2. 当 Eden 的空间填满时，程序又需要创建对象，JVM 的垃圾回收器将对伊甸园区进行垃圾回收（MinorGC），将伊甸园区中的不再被其他对象所引用的对象进行销毁。再加载新的对象放到伊甸园区。
3. 然后将伊甸园中的剩余对象移动到幸存者0区。
4. 如果再次触发垃圾回收，此时上次幸存下来的放到幸存者0区的，如果没有回收，就会放到幸存者1区。
5. 如果再次经历垃圾回收，此时会重新放回幸存者0区，接着再去幸存者1区。
6. 啥时候能去养老区呢？可以设置次数。默认是15次。可以设置新生区进入养老区的年龄限制，设置 JVM 参数：**-XX:MaxTenuringThreshold**=N 进行设置。
7. 在养老区，相对悠闲。当养老区内存不足时，再次触发GC：Major GC，进行养老区的内存清理。
8. 若养老区执行了Major GC之后，发现依然无法进行对象的保存，就会产生OOM异常。

**总结**：

**针对幸存者 s0,s1区：复制之后有交换，谁空谁是 to；关于垃圾回收：频繁在新生区收集，很少在养老区收集，几乎不再永久区/元空间收集。**

**特殊情况：**

1. 如果来了一个新对象，先看看 Eden 是否放的下？
   - 如果 Eden 放得下，则直接放到 Eden 区
   - 如果 Eden 放不下，则触发 YGC ，执行垃圾回收，看看还能不能放下？
2. 将对象放到老年区又有两种情况：
   - 如果 Eden 执行了 YGC 还是无法放不下该对象，那没得办法，只能说明是超大对象，只能直接放到老年代
   - 那万一老年代都放不下，则先触发FullGC ，再看看能不能放下，放得下最好，但如果还是放不下，那只能报 OOM
3. 如果 Eden 区满了，将对象往幸存区拷贝时，发现幸存区放不下啦，那只能便宜了某些新对象，让他们直接晋升至老年区

![2022-03-31_215156](https://img.qinweizhao.com/2022/03/2022-03-31_215156.png)

**代码举例：**

```Java
/**
 * -Xms600m -Xmx600m
 */
public class HeapInstanceTest {
    byte[] buffer = new byte[new Random().nextInt(1024 * 200)];

    public static void main(String[] args) {
        ArrayList<HeapInstanceTest> list = new ArrayList<HeapInstanceTest>();
        while (true) {
            list.add(new HeapInstanceTest());
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

![2022-03-31_215206](https://img.qinweizhao.com/2022/03/2022-03-31_215206.png)

说明：

伊甸园区的分配如图所示，当分配的对象到达150M，则进行一次垃圾回收，将非垃圾放入幸存者1区，继续分配放入伊甸园区，到达150之后，将对象放入0区，并且将之前1区中的对象放入老年区，以此类推。直到老年区满400M，爆出 OOM(OutOfMemory)异常。

### 2、分配策略

1. 如果对象在 Eden 出生并经过第一次Minor GC 后仍然存活，并且能被 Survivor 容纳的话，将被移动到 Survivor 空间中，并将对象年龄设为1。
2. 对象在 Survivor 区中每熬过一次 MinorGC，年龄就增加1岁，当它的年龄增加到一定程度（默认为15岁，其实每个 JVM、每个 GC 都有所不同）时，就会被晋升到老年代
3. 对象晋升老年代的年龄阀值，可以通过选项 **-XX:MaxTenuringThreshold **来设置。

**不同年龄段的对象分配原则：**

1. 如果对象在 Eden 出生并经过第一次 Minor GC 后依然存活，并且能被 Survivor 容纳的话，将被移动到 Survivor 空间中，把那个将对象年龄设为1。对象在 Survivor 区中每熬过一次 MinorGC ，年龄就增加一岁，当它的年龄增加到一定程度（默认15岁，其实每个 JVM、每个GC都有所不同）时，就会被晋升到老年代中。对象晋升老年代的年龄阈值，可以通过选项 -XX：MaxTenuringThreshold来设置。

2. **优先分配到Eden**：开发中比较长的字符串或者数组，会直接存在老年代，但是因为新创建的对象都是朝生夕死的，所以这个大对象可能也很快被回收，但是因为老年代触发Major GC的次数比 Minor GC要更少，因此可能回收起来就会比较慢

3. **大对象直接分配到老年代**：尽量避免程序中出现过多的大对象

4. **长期存活的对象分配到老年代**

5. **动态对象年龄判断**：如果Survivor区中相同年龄的所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象可以直接进入老年代，无须等到MaxTenuringThreshold中要求的年龄。

6. **空间分配担保**： -XX:HandlePromotionFailure 。

   **测试：**

   分配 60m 堆空间，新生代 20m ，Eden 16m， s0 2m， s1 2m，buffer对象20m，Eden 区无法存放buffer， 直接晋升老年代。

   ```Java
   /**
    * 测试：大对象直接进入老年代
    * -Xms60m -Xmx60m -XX:NewRatio=2 -XX:SurvivorRatio=8 -XX:+PrintGCDetails
    *
    * @author qinweizhao
    * @since 2022/4/3
    */
   public class YoungOldArea {
   
       /**
        * 新生代 20m ，Eden 16m， s0 2m， s1 2m 老年代 40m
        *
        * @param args args
        */
       public static void main(String[] args) {
           //Eden 区无法存放buffer  晋升老年代
           //20m
           byte[] buffer = new byte[1024 * 1024 * 20];
       }
       
   }
   ```

   **结果：**

   20m的buffer存放于老年代中

   ![2022-03-31_215215](https://img.qinweizhao.com/2022/03/2022-03-31_215215.png)

## 四、GC 分类

JVM 的调优的一个环节，也就是垃圾收集，需要尽量避免垃圾回收，因为在垃圾回收的过程中，容易出现 STW（Stop the World）的问题，**而 Major GC 和 Full GC 出现 STW 的时间，是 Minor GC 的10倍以上**。JVM 在进行 GC 时，并非每次都针对上面三个内存区域（新生代、老年代、方法区）一起回收的，大部分时候回收都是指新生代。针对 Hotspot VM的实现，它里面的 GC 按照回收区域又分为两大种类型：一种是部分收集（Partial GC），一种是整堆收集（Full GC）。

- 整堆收集（Full GC）：收集整个Java堆和方法区的垃圾收集。

- 部分收集：
  - 新生代收集（Minor GC/Young GC）：只是新生代的垃圾收集。
  
  - 老年代收集（Major GC/Old GC）：只是老年代的垃圾收集。
  
  - 目前，只有CMS GC会有单独收集老年代的行为。
  
    注意，**很多时候Major GC 会和 Full GC混淆使用，需要具体分辨是老年代回收还是整堆回收。**
  
  - 混合收集（Mixed GC）：收集整个新生代以及部分老年代的垃圾收集。
    - 目前，只有G1 GC会有这种行为。

### 1、Minor GC

**触发机制：**

1. 当年轻代空间不足时，就会触发Minor GC，这里的年轻代满指的是Eden代满。Survivor满不会主动引发GC，在Eden区满的时候，会顺带触发s0区的GC，也就是被动触发GC（每次Minor GC会清理年轻代的内存）
2. 因为Java对象大多都具备朝生夕灭的特性，所以Minor GC非常频繁，一般回收速度也比较快。这一定义既清晰又易于理解。
3. Minor GC会引发STW（Stop The World），暂停其它用户的线程，等垃圾回收结束，用户线程才恢复运行

### 2、Major

**触发机制：**

1. 指发生在老年代的GC，对象从老年代消失时，我们说  Major G C 或 Full GC 发生了。
2. 出现了 Major GC，经常会伴随至少一次的 Minor GC。（但非绝对的，在Parallel Scavenge收集器的收集策略里就有直接进行MajorGC的策略选择过程）。
   - 也就是在老年代空间不足时，会先尝试触发Minor GC（哈？我有点迷？），如果之后空间还不足，则触发Major GC。
3. Major GC的速度一般会比Minor GC慢10倍以上，STW的时间更长。
4. 如果Major GC后，内存还不足，就报OOM了。

### 3、Full GC

**触发机制：**

1. 调用 System.gc() 时，系统建议执行 FullGC，但是不必然执行。
2. 老年代空间不足。
3. 方法区空间不足。
4. 通过 Minor GC后进入老年代的平均大小大于老年代的可用内存。
5. 由 Eden 区、survivor space0（From Space）区向survivor space1（To Space）区复制时，对象大小大于To Space可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小。

说明：Full GC 是开发或调优中尽量要避免的。这样 STW 时间会短一些。

## 五、其他

### 1、堆空间分代思想

为什么要把 Java 堆分代？不分代就不能正常工作了么？经研究，不同对象的生命周期不同。70%-99% 的对象都是临时对象。

- 新生代：有Eden、Survivor构成（s0,s1 又称为from,to），to总为空。
- 老年代：存放新生代中经历多次依然存活的对象。

其实不分代完全可以，分代的唯一理由就是**优化 GC性能**。如果没有分代，那所有的对象都在一块，就如同把一个学校的人都关在一个教室。GC 的时候要找到哪些对象没用，这样就会对堆的所有区域进行扫描，而很多对象都是朝生夕死的。如果分代的话，把新创建的对象放到某一地方，当 GC 的时候先把这块存储“朝生夕死”对象的区域进行回收，这样就会腾出很大的空间出来。

### 2、TLAB

1. 为什么要有 TLAB ？

   - 堆区是线程共享区域，任何线程都可以访问到堆区中的共享数据
   - 由于对象实例的创建在JVM中非常频繁，因此在并发环境下从堆区中划分内存空间是线程不安全的
   - 为避免多个线程操作同一地址，需要使用加锁等机制，进而影响分配速度

2. 什么是 TLAB ？

   - 从内存模型而不是垃圾收集的角度，对Eden区域继续进行划分，JVM为每个线程分配了一个私有缓存区域，它包含在Eden空间内

   ![2022-03-31_215226](https://img.qinweizhao.com/2022/03/2022-03-31_215226.png)

   - 多线程同时分配内存时，使用TLAB可以避免一系列的非线程安全问题，同时还能够提升内存分配的吞吐量，因此我们可以将这种内存分配方式称之为**快速分配策略**
   - 所有OpenJDK衍生出来的JVM都提供了TLAB的设计

3. 说明

   - 尽管不是所有的对象实例都能够在TLAB中成功分配内存，单JV明确是是将 TLAB 作为内存分配的首选。
   - 在程序中，开发人员可以通过选项“-XX:UseTLAB“ 设置是够开启 TLAB 空间。
   - 默认情况下，TLAB 空间的内存非常小，仅占有整个EDen空间的1%，当然我们可以通过选项 ”-XX:TLABWasteTargetPercent“ 设置 TLAB 空间所占用 Eden 空间的百分比大小。
   - 一旦对象在TLAB空间分配内存失败时，JVM 就会尝试着通过使用加锁机制确保数据操作的原子性，从而直接在 Eden 空间中分配了内存。

4.  TLAB对象分配过程

   ![2022-03-31_215243](https://img.qinweizhao.com/2022/03/2022-03-31_215243.png)

### 3、逃逸分析

1. 概念

   在《深入理解Java虚拟机》中关于 Java 堆内存有这样一段描述：随着 JIT 编译期的发展与逃逸分析技术逐渐成熟，**栈上分配、标量替换优化技术**将会导致一些微妙的变化，所有的对象都分配到堆上也渐渐变得不那么**绝对**了。
   在 Java 虚拟机中，对象是在 Java 堆中分配内存的，这是一个普遍的常识。但是，有一种特殊情况，那就是如果经过**逃逸分析（Escape Analysis)后发现，一个对象并没有逃逸出方法的话，那么就可能被优化成栈上分配**。这样就无需在堆上分配内存，也无须进行垃圾回收了。这也是最常见的堆外存储技术。

   - 如何将堆上的对象分配到栈，需要使用逃逸分析手段。
   - 这是一种可以有效减少Java程序中同步负载和内存堆分配压力的跨函数全局数据流分析算法。
   - 通过逃逸分析，Java Hotspot 编译器能够分析出一个新的对象的引用的使用范围从而决定是否要将这个对象分配到堆上。
   - 逃逸分析的基本行为就是分析对象动态作用域：
     - 当一个对象在方法中被定义后，**对象只在方法内部使用，则认为没有发生逃逸**。
     - 当一个对象在方法中被定义后，它被外部方法所引用，则认为发生逃逸。例如作为调用参数传递到其他地方中。
   - **如何快速的判断是否发生了逃逸分析，就看 new 的对象实体是否有可能在方法外被调用**。

2.  代码分析

```Java
public static StringBuffer createStringBuffer(String s1,String s2){
    StringBuffer sb = new StringBuffer();
    sb.append(s1);
    sb.append(s2);
    //发生逃逸 因为外部使用了
    return sb;
    
    //没有发生逃逸 外部没有使用
    return sb.toString();
}
```

3. 结论

   开发中能使用局部变量的，就不要使用在方法外定义。

### 4、逃逸分析之代码优化

- **栈上分配**：将堆分配转化为栈分配。如果一个对象在子程序中被分配，要使指向该对象的指针永远不会发生逃逸，对象可能是栈上分配的候选，而不是堆上分配

- **同步省略**：如果一个对象被发现只有一个线程被访问到，那么对于这个对象的操作可以不考虑同步。

- **分离对象或标量替换**：有的对象可能不需要作为一个连续的内存结构存在也可以被访问到，那么对象的部分（或全部）可以不存储在内存，而是存储在CPU寄存器中。

1. **栈上分配**

   - JIT编译器在编译期间根据逃逸分析的结果，发现如果一个对象并没有逃逸出方法的话，就可能被优化成栈上分配。分配完成后，继续在调用栈内执行，最后线程结束，栈空间被回收，局部变量对象也被回收。这样就无须进行垃圾回收了。
   - 常见的栈上分配的场景：在逃逸分析中，已经说明了，分别是给成员变量赋值、方法返回值、实例引用传递。

   代码分析：

   ```java
   /**
    * 分配100000个对象
    * -Xmx1G -Xms1G
    */
   public class StackAllocation {
       
       public static void main(String[] args) throws InterruptedException {
           
           long start = System.currentTimeMillis();
   
           for (int i = 0; i < 10000000; i++) {
               //创建对象
               alloc();
           }
           // 查看执行时间
           long end = System.currentTimeMillis();
           System.out.println("花费的时间为： " + (end - start) + " ms");
           
           // 为了方便查看堆内存中对象个数，线程sleep
           Thread.sleep(100000);
       }
   
       private static void alloc() {
           User user = new User();//未发生逃逸
       }
       static class User {
       }
   
       private static void alloc() {
           User user = new User();//未发生逃逸
       }
       static class User {
       }
   }
   ```

   结果：

   关闭逃逸分析 -XX:-DoEscapeAnalysis。维护10000个对象，花费的时间为： 104 ms。![2022-03-31_215251](https://img.qinweizhao.com/2022/03/2022-03-31_215251.png)

   开启逃逸分析(默认开启) ，维护少量对象，花费的时间为： 6 ms。

   ![20210625205306816](https://img.qinweizhao.com/2022/03/2022-03-31_220040.png)

2. **同步省略（同步消除）**

   - 如果一个对象被发现只能从一个线程被访问到，那么对于这个对象的操作可以不考虑同步。
   - 线程同步的代价是相当高的，同步的后果是降低并发性和性能。
   - 在动态编译同步块的时候，JIT编译器可以借助逃逸分析来**判断同步块所使用的锁对象是否只能够被一个线程访问而没有被发布到其他线程**。
   - 如果没有，那么JIT编译器在编译这个同步块的时候就会取消对这部分代码的同步。这样就能大大提高并发性和性能。这个**取消同步的过程就叫同步省略，也叫锁消除**。
   
   如下代码：
   
   ```Java
   public void f() {
           Object hollis = new Object();
           synchronized(hollis) {
               System.out.println(hollis);
   }
   ```
   
   代码中对 hollis 这个对象进行加锁，但是hollis对象的生命周期只在 f() 方法中    并不会被其他线程所访问控制，所以在 JIT 编译阶段就会被优化掉。
   
   ```Java
   public void f2() {
           Object hollis = new Object();
           System.out.println(hollis);
       }
   ```
   
   字节码分析：
   
   ```java
    0 new #2 <java/lang/Object>
    3 dup
    4 invokespecial #1 <java/lang/Object.<init>>
    7 astore_1
    8 aload_1
    9 dup
   10 astore_2
   11 monitorenter
   12 getstatic #3 <java/lang/System.out>
   15 aload_1
   16 invokevirtual #4 <java/io/PrintStream.println>
   19 aload_2
   20 monitorexit
   21 goto 29 (+8)
   24 astore_3
   25 aload_2
   26 monitorexit
   27 aload_3
   28 athrow
   29 return
   ```
   
   注意：字节码文件中并没有进行优化，可以看到加锁和释放锁的操作依然存在，**同步省略操作是在解释运行时发生的**。

3. **标量替换**

   分离对象或标量替换

   - 有的对象可能不需要作为一个连续的内存结构存在也可以被访问到，那么对象的部分（或全部）可以不存储在内存，而是存储在CPU寄存器中。
   - **标量Scalar**是指一个无法在分解成更小的数据的数据。Java中的原始数据类型就是标量
   - 相对的，那些还可以分解的数据叫做**聚合量(Aggregate)**，Java中对象就是聚合量，因为它可以分解成其他聚合量和标量
   - 在JIT阶段，如果经过逃逸分析，发现一个对象不会被外界访问的话，那么经过JIT优化，就会把这个对象拆解成若干个其中包含的若干个成员变量来替代。这个过程就是标量替换。

   ```java
   public class ScalarTest {
       public static void main(String[] args) {
           alloc();   
       }
       public static void alloc(){
           Point point = new Point(1,2);
       }
   }
   class Point{
       private int x;
       private int y;
       public Point(int x,int y){
           this.x = x;
           this.y = y;
       }
   }
   
   
   ```

   以上代码会被优化为：

   ```java
   public static void alloc(){
       int x = 1;
       int y = 2;
   }
   ```

   可以看到，Point这个聚合量经过逃逸分析后，发现他并没有逃逸，就被替换成两个标量了。那么标量替换有什么好处呢？就是可以**大大减少堆内存的占用**。因为一旦不需要创建对象了，那么就不再需要分配堆内存了。标量替换为栈上分配提供了很好的基础。

### 5、逃逸分析小结（技术并不成熟）

- 关于逃逸分析的论文在1999年就已经发表了，但直到 JDK1.6才有实现，而且这项技术到如今也并不是十分成熟的。
- 其根本原因就是无法保证逃逸分析的性能消耗一定能高于他的消耗。虽然经过逃逸分析可以做标量替换、栈上分配、和锁消除。但是逃逸分析自身也是需要进行一系列复杂的分析的，这其实也是一个相对耗时的过程。
- 一个极端的例子，就是经过逃逸分析之后，发现没有一个对象是不逃逸的。那这个逃逸分析的过程就白白浪费掉了。
- 虽然这项技术并不十分成熟，但是它也是即时编译器优化技术中一个十分重要的手段。
- 注意到有一些观点，认为通过逃逸分析，JVM 会在栈上分配那些不会逃逸的对象，这在理论上是可行的，但是取决于 JVM 设计者的选择。据我所知，Oracle Hotspot VM中并未这么做，这一点在逃逸分析相关的文档里已经说明，所以可以明确所有的对象实例都是创建在堆上。
- 目前很多书籍还是基于 JDK7 以前的版本，JDK 已经发生了很大变化，intern 字符串的缓存和静态变量曾经都被分配在永久代上，而永久代已经被元数据区取代。但是，intern 字符串缓存和静态变量并不是被转移到元数据区，而是直接在堆上分配，所以这一点同样符合前面一点的结论：**对象实例都是分配在堆上**。
- **年轻代是对象的诞生、省长、消亡的区域，一个对象在这里产生、应用、最后被垃圾回收器收集、结束生命**
- **老年代防止长生命周期对象，通常都是从Survivor区域筛选拷贝过来的Java对象。当然，也有特殊情况，我们知道普通的对象会被分配在TLAB上，如果对象较大，JVM会试图直接分配在Eden其他位置上；如果对象他打，完全无法在新生代找到足够长的连续空闲空间，JVM就会直接分配到老年代**
- **当GC只发生在年轻代中，回收年轻对象的行为被称为MinorGC。当GC发生在老年代时则被称为MajorGC或者FullGC。一般的，MinorGC的发生频率要比MajorGC高很多，即老年代中垃圾回收发生的频率大大低于年轻代**

### 6、常用调优工具

- JDK 命令行
- Eclipse：Memory Analyzer Tool
- Jconsole
- VisualVM
- Jprofiler
- Java Flight Recorder
- GCViewer
- GC Easy
