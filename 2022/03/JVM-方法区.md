# JVM-方法区

## 一、栈，堆，方法区的交互关系

![2022-03-31_194457](https://img.qinweizhao.com/2022/03/2022-03-31_194457.png)



![2022-03-31_194501](https://img.qinweizhao.com/2022/03/2022-03-31_194501.png)

- Person 类的 .class 信息存放在方法区中。
- person 变量存放在 Java 栈的局部变量表中。
- 真正的 person 对象存放在 Java 堆中。
- 在 person 对象中，有个指针指向方法区中的 person 类型数据，表明这个 person 对象是用方法区中的 Person 类 new 出来的。

## 二、方法区的理解

方法区是一种规范，方法区在 JDK1.7 及之前，用**永久代**实现，使用虚拟机的内存；JDK1.8 及以后改用与 JRockit、J9（不存在永久代的概念） 一样在本地内存中实现的元空间（Metaspace）来代替，使用本地内存。**方法区主要存放的是 Class，而堆中主要存放的是实例化的对象。**

![2022-03-31_194512](https://img.qinweizhao.com/2022/03/2022-03-31_194512.png)

- 《Java虚拟机规范》中明确说明：尽管所有的方法区在逻辑上是属于堆的一部分，但一些简单的实现可能不会选择去进行垃圾收集或者进行压缩。但对于 HotSpot JVM 而言，方法区还有一个别名叫做 Non-Heap（非堆），目的就是要和堆分开。所以，方法区可以看作是一块独立于 Java 堆的内存空间。
  
- 方法区（Method Area）与 Java 堆一样，是各个线程共享的内存区域。

- 方法区在 JVM 启动时就会被创建，并且它的实际的物理内存空间中和 Java 堆区一样都可以是不连续的。

- 方法区的大小，跟堆空间一样，可以选择固定大小或者可拓展。

- 方法区的大小决定了系统可以保存多少个类，如果系统定义了太多的类，导致方法区溢出，虚拟机同样会抛出内存溢出错误：`java.lang.OutofMemoryError:PermGen space` 或者`java.lang.OutOfMemoryError:Metaspace`。

  - 加载大量的第三方 jar 包。
  - Tomcat 部署的工程过多。
  - 大量动态生成反射类。
  
- 关闭 JVM 就会释放这个区域的内存。


代码举例：

```java
/**
 * @author qinweizhao
 * @since 2022/4/3
 */
public class MethodArea {

    public static void main(String[] args) {
        System.out.println("start...");
        try {
            Thread.sleep(1000000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("end...");
    }

}
```

JVisualVM 查看加载类的个数：

![2022-03-31_194522](https://img.qinweizhao.com/2022/03/2022-03-31_194522.png)

## 三、设置方法区大小

- JDK7 及以前：
  
  - 永久代初始分配大小:	-XX:PermSize 来设置。默认值是20.75M。
  - 永久代最大空间：-XX :MaxPermSize 来设定。32位机器默认是64M，64位机器模式是82M。
  - 当 JVM 加载的类信息容量超过了这个值，会报异常 `OutOfMemoryError:PermGen space`。
  
- JDK8 及以后：

  -  元空间初始分配大小：-XX:MetaspaceSize 。Windows 下默认为21M。
  -  元空间最大：-XX:MaxMetaspaceSize。Windows 默认为-1 即无限制。
  -  与永久代不同，如果不指定大小，默认情况下，虚拟机会耗尽所有的可用系统内存。 如果元数据区发生溢出，虚拟机一样会拋出异常 `OutOfMemoryError:Metaspace`。
  -  初始分配大小就是初始的高水位线，一旦触及这个水位线，Full GC 将会被触发并卸载没用的类（即这些类对应的类加载器不再存活），然后这个高水位线将会重置。新的高水位线的值取决于 GC 后释放了多少元空间。如果释放的空间不足，那么在不超过MaxMetaspaceSize 时，适当提高该值。如果释放空间过多，则适当降低该值。
  -  如果初始化的高水位线设置过低，上述高水位线调整情况会发生很多次。通过垃圾回收器的日志可以观察到 Full GC 多次调用。为了避免频繁地 GC，建议将 `-XX:MetaspaceSize`设置为一个相对较高的值。

  举例：

  ```shell
   *  JDK7及以前：
   *  查询 jps  -> jinfo -flag PermSize [进程id]
   *  -XX:PermSize=20m -XX:MaxPermSize=82m
   *
   *  JDK8及以后：
   *  查询 jps  -> jinfo -flag MetaspaceSize [进程id]
   *  -XX:MetaspaceSize=20.79m  -XX:MaxMetaspaceSize=-1
  ```

## 四、方法区异常

  代码举例

  ```java
  public class OOMTestMetaSpace extends ClassLoader {
      public static void main(String[] args) {
          int count = 0;
          try {
              OOMTestMetaSpace test = new OOMTestMetaSpace();
              for (int i = 0; i < 10000000; i++) {
                  //创建ClassWriter对象，用于生成类的二进制字节码
                  ClassWriter classWriter = new ClassWriter(0);
                  //指明版本号，修饰符，类名，包名，父类，接口
                  classWriter.visit(Opcodes.V1_6, Opcodes.ACC_PUBLIC, "Class" + i, null, "java/lang/Object", null);
                  //返回byte[]
                  byte[] code = classWriter.toByteArray();
                  //类的加载
                  test.defineClass("Class" + i, code, 0, code.length);//Class对象
                  count++;
              }
          } finally {
              System.out.println(count);
          }
      }
  }
  ```

  结果爆出 OOM   ![2022-03-31_194543](https://img.qinweizhao.com/2022/03/2022-03-31_194543.png)

  **如何解决OOM**

  1、要解决 00M 异常或 heap space 的异常，一般的手段是首先通过内存映像分析工具（如Eclipse Memory Analyzer） 对 dump 出来的堆转储快照进行分析，重点是确认内存中的对象是否是必要的，也就是要先分清楚到底是出现了内存泄漏（Memory Leak）还是内存溢出（Memory 0verflow） 。

  2、如果是内存泄漏，可进一步通过工具查看泄漏对象到 GC Roots 的引用链。于是就能找到泄漏对象是通过怎样的路径与 GCRoots 相关联并导致垃圾收集器无法自动回收它们的。掌握了泄漏对象的类型信息，以及 GC Roots 引用链的信息，就可以比较准确地定位出泄漏代码的位置。

  3、如果不存在内存泄漏，换句话说就是内存中的对象确实都还必须存活着，那就应当检查虚拟机的堆参数（-Xmx与-Xms），与机器物理内存对比看是否还可以调大，从代码上检查是否存在某些对象生命周期过长、持有状态时间过长的情况，尝试减少程序运行期的内存消耗。

## 五、方法区的内部结构

![2022-03-31_194611](https://img.qinweizhao.com/2022/03/2022-03-31_194611.png)

方法区用于存储已被虚拟机加载的**类型信息、运行时常量池、静态变量、域（属性）信息、方法信息、JIT 编译后的代码缓存**等。

![2022-03-31_194634](https://img.qinweizhao.com/2022/03/2022-03-31_194634.png)

1.  **类型信息**

   对每个加载的类型（ 类 class、接口 interface、枚举 enum、注解 annotation），JVM必须在方法区中存储以下类型信息：

   - 这个类型的完整有效名称（全名=包名.类名）。
   - 这个类型直接父类的完整有效名（对于interface或是java. lang.Object，都没有父类）。
   - 这个类型的修饰符（public， abstract， final的某个子集）。
   - 这个类型直接实现接口的一个有序列表。

2.  **域信息（成员变量/属性）**

   - JVM 必须在方法区中保存类型的所有域的相关信息以及域的声明顺序。
   - 域的相关信息包括：域名称、 域类型、域修饰符（public， private， protected， static， final， volatile， transient的某个子集）。

3.  **方法信息**

   JVM 必须保存所有方法的以下信息，同域信息一样包括声明顺序：

   - 方法名称
   - 方法的返回类型（或void），void 在 Java 中对应的为 void.class。
   - 方法参数的数量和类型（按顺序）。
   - 方法的修饰符（public， private， protected， static， final， synchronized， native ， abstract的一个子集）。
   - 方法的字节码（bytecodes）、操作数栈、局部变量表及大小（ abstract和native 方法除外）。
   - 异常表（ abstract和native方法除外），每个异常处理的开始位置、结束位置、代码处理在程序计数器中的偏移地址、被捕获的异常类的常量池索引。

4. **non-final的类变量**

   - 静态变量和类关联在一起，随着类的加载而加载，他们成为类数据在逻辑上的一部分。
   - 类变量被类的所有实例所共享，即使没有类实例你也可以访问它。

5.  **全局常量 static final** 

   在编译的时候就被分配赋值了，对应的值在编译上的时候已经写死在字节码文件中了。

6. **运行时常量池**

   方法区，内部包含了运行时常量池。字节码文件，内部包含了常量池。（之前的字节码文件中已经看到了很多Constant pool的东西，这个就是常量池）。要弄清楚方法区，需要理解清楚 ClassFile，因为加载类的信息都在方法区。要弄清楚方法区的运行时常量池，需要理解清楚ClassFile中的常量池。

   **常量池**

   一个有效的字节码文件中除了包含类的版本信息、字段、方法以及接口等描述符信息外。还包含一项信息就是**常量池表**（**Constant Pool Table**），包括各种字面量和对类型、域和方法的符号引用。

   ![2022-03-31_194642](https://img.qinweizhao.com/2022/03/2022-03-31_194642.png)

   常量池中有**数量值**、**字符串值**、**类引用**、**字段引用**、**方法引用**。常量池、可以看做是一张表，虚拟机指令根据这张常量表找到要执行的类名、方法名、参数类型、字面量等类型。

   **运行时常量池**

   - 运行时常量池（Runtime Constant Pool）是方法区的一部分。
   - 常量池表（Constant Pool Table）是Class字节码文件的一部分，用于存放编译期生成的各种字面量与符号引用，**这部分内容将在类加载后存放到方法区的运行时常量池中**。（运行时常量池就是常量池在程序运行时的称呼）
   - 运行时常量池，在加载类和接口到虚拟机后，就会创建对应的运行时常量池。
   - JVM 为每个已加载的类型（类或接口）都维护一个常量池。池中的数据项像数组项一样，是通过索引访问的。
   - 运行时常量池中包含多种不同的常量，包括编译期就已经明确的数值字面量，也包括到运行期解析后才能够获得的方法或者字段引用。**此时不再是常量池中的符号地址了，这里换为真实地址**。
   - 运行时常量池，相对于 Class 文件常量池的另一重要特征是：具备动态性。
   - 运行时常量池类似于传统编程语言中的符号表（symbol table），但是它所包含的数据却比符号表要更加丰富一些。
   - 当创建类或接口的运行时常量池时，如果构造运行时常量池所需的内存空间超过了方法区所能提供的最大值，则JVM会抛OutofMemoryError异常。
   
   **代码验证：**
   
   ```java
   /**
    * @author qinweizhao
    * @since 2022/4/3
    */
   public class MethodInnerStructure extends Object implements Comparable<String>, Serializable {
   
       /**
        * 属性
        */
       public int num = 10;
   
       private static String str = "测试方法的内部结构";
   
       /**
        * 方法
        */
       public void test1(){
           int count = 20;
           System.out.println("count = " + count);
       }
       public static int test2(int cal){
           int result = 0;
           try {
               int value = 30;
               result = value / cal;
           } catch (Exception e) {
               e.printStackTrace();
           }
           return result;
       }
   
       @Override
       public int compareTo(String o) {
           return 0;
       }
   
   }
   ```
   
   **反编译并输出到文件：**
   
   ```sh
   javap -v -p MethodInnerStructure.class > test.txt
   ```
   
   参数 -p 确保能查看 private 权限类型的字段或方法。
   
   ```txt
   // 输出太多了！！！截取部分吧。
   ......
   ```
   
   ```txt
   // 类型信息      
   public class com.qinweizhao.jvm.ma.MethodInnerStructure extends java.lang.Object implements java.lang.Comparable<java.lang.String>, java.io.Serializable
     minor version: 0
     major version: 52
     flags: ACC_PUBLIC, ACC_SUPER
     
     
     // 域信息
     public int num;
     	// I 表示字段类型为 Integer
       descriptor: I
       // 表示字段权限修饰符为 public
       flags: ACC_PUBLIC
   
     private static java.lang.String str;
       descriptor: Ljava/lang/String;
       flags: ACC_PRIVATE, ACC_STATIC
       
       
     // 方法信息    
     public void test1();
       // 方法返回值类型为 void
       descriptor: ()V
       flags: ACC_PUBLIC
       Code:
         // 操作数栈深度为 3,局部变量个数为 2 个（实力方法包含 this）, 方法虽然没有参数，但是其 args_size=1 ，这时因为将 this 作为了参数
         stack=3, locals=2, args_size=1
            0: bipush        20
            2: istore_1
            3: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
            6: new           #4                  // class java/lang/StringBuilder
            9: dup
           10: invokespecial #5                  // Method java/lang/StringBuilder."<init>":()V
           13: ldc           #6                  // String count =
           15: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
           18: iload_1
           19: invokevirtual #8                  // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
           22: invokevirtual #9                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
           25: invokevirtual #10                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
           28: return
         LineNumberTable:
           line 22: 0
           line 23: 3
           line 24: 28
         LocalVariableTable:
           Start  Length  Slot  Name   Signature
               0      29     0  this   Lcom/qinweizhao/jvm/ma/MethodInnerStructure;
               3      26     1 count   I
   ```
   

## 六、方法区使用举例

```java
public class MethodAreaDemo {
    public static void main(String[] args) {
        int x = 500;
        int y = 100;
        int a = x / y;
        int b = 50;
        System.out.println(a + b);
    }
}
```

字节码指令

```shell
 0 sipush 500 // 将500放入操作数栈中
 3 istore_1 // 将变量操作数栈顶中的500存储到LV中
 4 bipush 100 // 将100放入操作数栈中
 6 istore_2 //将操作数栈顶中的100放入LV中
 7 iload_1 // 读取LV1 500
 8 iload_2 // 读取LV2 100
 9 idiv // 将LV1读取的值除以LV2读取的值
10 istore_3 // 结果5存入LV3中
11 bipush 50 //压入50到操作数栈顶
13 istore 4 //将栈顶的50存入LV中
15 getstatic #2 <java/lang/System.out>
18 iload_3
19 iload 4
21 iadd
22 invokevirtual #3 <java/io/PrintStream.println>
25 return
```

![2022-03-31_194712](https://img.qinweizhao.com/2022/03/2022-03-31_194712.png)

## 七、方法区的演进细节

首先明确：只有 HotSpot 才有永久代。 BEA JRockit、IBM J9 等来说，是不存在永久代的概念的。原则上如何实现方法区属于虛拟机实现细节，不受《Java虚拟机规范》管束，并不要求统一。

- 针对HotSpot

  | 版本         | 方法区实现                                                   |
  | ------------ | ------------------------------------------------------------ |
  | JDK1.6及之前 | 静态变量及字符串常量池存放在永久代（方法区1.6的实现）上      |
  | JDK1.7       | 有永久代，但已经逐步“去永久代”，字符串常量池、静态变量移除，保存在堆中 |
  | JDK1.8及之后 | 无永久代，类型信息、字段、方法、常量保存在本地内存的元空间，但字符串常量池、静态变量仍在堆 |

  ![2022-03-31_194741](https://img.qinweizhao.com/2022/03/2022-03-31_194741.png)

  ![2022-03-31_194755](https://img.qinweizhao.com/2022/03/2022-03-31_194755.png)

  ![2022-03-31_194809](https://img.qinweizhao.com/2022/03/2022-03-31_194809.png)

**永久代为什么要被元空间替换？**

- 随着 Java8 的到来，HotSpot VM 中再也见不到永久代了。但是这并不意味着类.的元数据信息也消失了。这些数据被移到了一个与堆不相连的本地内存区域，这个区域叫做元空间（ Metaspace ）。
- 由于类的元数据分配在本地内存中，元空间的最大可分配空间就是系统可用内存空间。
- 这项改动是很有必要的，原因有：
  - 为永久代设置空间大小是很难确定的。 在某些场景下，如果动态加载类过多，容易产生Perm区的 OOM 。比如某个实际 Web 工程中，因为功能点比较多，在运行过程中，要不断动态加载很多类，经常出现致命错误。 `"Exception in thread' dubbo client x.x connector’java.lang.OutOfMemoryError： PermGenspace"` 。而元空间和永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存。因此，默认情况下，元空间的大小仅受本地内存限制。
  - 对永久代进行调优是很困难的。方法区的垃圾收集主要回收两部分内容：常量池中废弃的常量和不再用的类型，方法区的调优主要是为了降低 **Full GC**。有些人认为方法区（如 HotSpot 虚拟机中的元空间或者永久代）是没有垃圾收集行为的，其实不然。《Java 虚拟机规范》对方法区的约束是非常宽松的，提到过可以不要求虚拟机在方法区中实现垃圾收集。事实上也确实有未实现或未能完整实现方法区类型卸载的收集器存在（如 JDK11时期的 ZGC 收集器就不支持类卸载）。一般来说这个区域的回收效果比较难令人满意，尤其是类型的卸载，条件相当苛刻**。但是这部分区域的回收有时又确实是必要的。以前 Sun 公司的 Bug 列表中，曾出现过的若干个严重的 Bug 就是由于低版本的 HotSpot 虚拟机对此区域未完全回收而导致内存泄漏。

**字符串常量池 StringTable 为什么要调整位置？**

JDK7 中将 StringTable 放到了堆空间中。因为永久代的回收效率很低，在 Full GC 的时候才会触发。而 Full GC 是老年代的空间不足、永久代不足时才会触发。这就导致了 StringTable 回收效率不高。而我们开发中会有大量的字符串被创建，回收效率低，导致永久代内存不足。放到堆里，能及时回收内存。

## 八、方法区的垃圾回收

- 有些人认为方法区（如Hotspot，虚拟机中的元空间或者永久代）是没有垃圾收集行为的，其实不然。《Java 虚拟机规范》对方法区的约束是非常宽松的，提到过可以不要求虚拟机在方法区中实现垃圾收集。事实上也确实有未实现或未能完整实现方法区类型卸载的收集器存在（如 JDK11 时期的 2GC 收集器就不支持类卸载）。
- 一般来说这个区域的回收效果比较难令人满意，尤其是类型的卸载，条件相当苛刻。但是这部分区域的回收有时又确实是必要的。以前 Sun 公司的 Bug 列表中，曾出现过的若干个严重的 Bug 就是由于低版本的 Hotspot 虚拟机对此区域未完全回收而导致内存泄漏。
- 方法区的垃圾收集主要回收两部分内容：常量池中废奔的常量和不再使用的类型。

- 先来说说方法区内常量池之中主要存放的两大类常量：字面量和符号引用。 字面量比较接近 Java 语言层次的常量概念，如文本字符串、被声明为 final 的常量值等。而符号引用则属于编译原理方面的概念，包括下面三类常量：
  - 类和接口的全限定名
  - 字段的名称和描述符
  - 方法的名称和描述符
  
- HotSpot 虚拟机对常量池的回收策略是很明确的，只要常量池中的常量没有被任何地方引用，就可以被回收。

- 回收废弃常量与回收 Java 堆中的对象非常类似。

  下面也称作**类卸载**

- 判定一个常量是否“废弃”还是相对简单，而要判定一个类型是否属于“不再被使用的类”的条件就比较苛刻了。需要同时满足下面三个条件：
  - 该类所有的实例都已经被回收，也就是Java堆中不存在该类及其任何派生子类的实例。
  - 加载该类的类加载器已经被回收，这个条件除非是经过精心设计的可替换类加载器的场景，如OSGi、JSP的重加载等，否则通常是很难达成的。
  - 该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。
  
- Java 虚拟机被允许对满足上述三个条件的无用类进行回收，这里说的仅仅是“被允许”，而并不是和对象一样，没有引用了就必然会回收。关于是否要对类型进行回收，HotSpot虚拟机提供了`-Xnoclassgc`参数进行控制，还可以使用`-verbose:class` 以及 `-XX：+TraceClass-Loading`、`-XX：+TraceClassUnLoading`查看类加载和卸载信息。

- 在大量使用反射、动态代理、CGLib 等字节码框架，动态生成 JSP 以及 OSGi 这类频繁自定义类加载器的场景中，通常都需要 Java 虚拟机具备类型卸载的能力，以保证不会对方法区造成过大的内存压力。

## 九、总结

![2022-03-31_194815](https://img.qinweizhao.com/2022/03/2022-03-31_194815.png)
