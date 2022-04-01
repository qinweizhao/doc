# JVM-类加载子系统

Java 虚拟机将描述类的数据从 class 字节码文件加载到内存，并且对数据进行校验，转化，解析，初始化的工作，最终形成在内存中可以直接使用的数据类型。这个过程叫做虚拟机的类加载机制。

## 一、概述

### 1、图示

![2022-03-31_145732](https://img.qinweizhao.com/2022/03/2022-03-31_145732.png)
### 2、作用

- 类加载子系统负责从文件系统或者网络中加载 class 文件（class 文件在开头有特定标识）。

- l类加载器(Class Loader)只负责 class 文件的加载，至于是否可以运行，由执行引擎（Execution Engine）决定。

- **加载的类信息存放于一块成为方法区的内存空间**。除了类信息之外，方法区还会存放运行时常量池信息，可能还包括字符串字面量和数字常量（这部分常量信息是 class 文件中常量池部分的内存映射）。

### 3、类加载器扮演的角色

![2022-03-31_145753](https://img.qinweizhao.com/2022/03/2022-03-31_145753.png)

- Car.class 存放于本地硬盘中，在运行的时候，JVM 将 Car.class 文件加载到 JVM 中，被称为 DNA 元数据模板（图中 Car Class）。

  存放在 JVM 的方法区中，之后根据元数据模板实例化出相应的对象。

- 在 `.class` -> `JVM` -> `元数据模板` -> `实例对象` 这个过程中，类加载器扮演者快递员的角色。

### 4、 类加载的时机

关于类加载的时机，《Java虚拟机规范》中并没有明确规定。这点可以由虚拟机的具体实现决定。但是类的初始化阶段，规范中明确规定当某个类没有进行初始化，**只有以下6中情况才会触发其初始化过程。**

1. 遇到`new`,`getStatic`，`putStatic`,`invokeStatic`,这四条字节码指令的时候，如果改类型没有进行初始化，则会触发其初始化。也就是如下情况
   1.1. 遇到`new`关键字进行创建对象的时候。
   1.2. 读取或者设置一个类的静态字段的时候（必须被final修饰，也就是在编译器把结果放入常量池中)。
   1.3. 调用一个类的静态方法的时候。
2. 使用 java.lang.reflect 进行反射调用的时候。
3. 当初始化某个类，发现其父类没有初始化的时候。
4. 当虚拟机启动的时候，会触发其主方法所在的类进行初始化。
5. 当使用 JDK1.7 中的动态语言支持时，如果一个`java.lang.invoke.MethidHandle`实例最后的解析结果为`REF_getStatic`,`REF_putStatic`,`REF_invokeStatic`,`REF_newInvokeSpecial`四种类型的方法句柄，并且这个句柄对应的类没有被初始化。
6. 当一个接口实现了 JDK1.8 中的默认方法的时候，如果这个接口的实现类被初始化，则该接口要在其之前进行实例化。

对于以上6中触发类的初始化条件，在 JVM 规范中有一个很强制的词，`if and only if` （有且只有）。这六种行为被称为对`类进行主动引用`，除此之外，其他引用类的方式均不会触发类的初始化。

## 二、类加载过程

![2022-03-31_145941](https://img.qinweizhao.com/2022/03/2022-03-31_145941.png)

类加载的过程主要分为三个阶段 加载，链接，初始化。 而链接阶段又可以细分为验证，准备，解析三个子阶段。

### 1、加载过程

加载过程需要完成以下三个事情:

- 通过一个类的`全限定名`获取定义此类的`二进制字节流`；

- 将这个字节流所代表的的`静态存储结构`转化为方法区的`运行时数据结构`；

- 在内存中生成一个代表这个类的 java.lang.Class 对象，作为方法区这个类的各种`数据的访问入口 `

**加载 class 文件的方式：**

- 从本地系统中直接加载。
- 通过网络获取，典型场景：Web Applet。
- 从zip压缩包中读取，成为日后jar、war格式的基础。
- 运行时计算生成，使用最多的是：动态代理技术。
- 由其他文件生成，典型场景：JSP应用从专有数据库中提取.class文件，比较少见。
- 从加密文件中获取，典型的防Class文件被反编译的保护措施。

### 2、链接过程

![2022-03-31_145818](https://img.qinweizhao.com/2022/03/2022-03-31_145818.png)

#### 1. 验证(Verify)

##### 1. 目的:

在于确保 class 文件的字节流中包含信息符合当前 JVM 规范要求，保证被加载类的正确性，不会危害虚拟机自身安全。

##### 2. 主要包括四种验证

   - `文件格式验证`

     - 字节码是否以十六进制的`CAFEBABE`开头
     - 主，次版本号是否在当前虚拟机可接受的范围之内。
     - 常量池的常量中是否有不被支持的类型
     - Class文件中是否有被添加的其他恶意信息。

     文件格式验证不止以上，上面所列举的只是从HotSpot虚拟机源码中摘抄的一部分。只有通过这个阶段的验证之后，这一段字节流才会进入虚拟机内存中进行存储，
     之后的过程都是基于方法区中的存储结构进行的。不会直接读取字节流了。

   - `源数据验证`

     用于保证字节码中的代码符合《Java语言规范》

     - 此类的父类是否是不可继承的类（Final修饰的）
     - 如果此类不是抽象类，它是否实现了全部需要实现的方法。
     - 类中的字段，方法是否和父类冲突。
     - ……

   - `字节码验证`

     此过程保证代码是符合逻辑的，对代码的流程进行判断，保证不会出现危害虚拟机安全的情况。

     - 保证任意时刻操作数栈中的类型和指令代码序列可以正常工作，比如执行到iadd字节码指令，但是操作数栈顶有一位是Long类型的。
     - 保证代码中的类型转换是有效的。

     如果一个类型中的方法体没有通过次阶段，那它一定是有问题的。但是，不可以认为只要通过此阶段验证，一定没有问题。通过程序去校验程序的逻辑是无法做到绝对准确的。

   - `符号引用验证`。

     此阶段验证符号引用是否合法，主要用于解析阶段的前置任务。

     主要用于判断 该类中是否存在缺少后者被禁止访问它依赖的某些外部类，字段，方法等资源。

#### 2. 准备(Prepare)
   - 为类变量（static）分配内存并且设置初始值。
   
   - 这里不包含用 final 修饰的 static，因为 final 在编译的时候就会分配了，准备阶段会显式初始化。
   - 不会为实例变量分配初始化，类变量会分配在方法去中，而实例变量是会随着对象一起分配到堆中。

#### 3. 解析(Resolve)
- 将常量池内的符号引用转换为直接引用的过程。
  
- 事实上，解析操作往往会伴随着JVM在执行完初始化之后再执行。
  
- 符号引用就是一组符号来描述所引用的目标。符号应用的字面量形式明确定义在《java虚拟机规范》的class文件格式中。直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄。
  
- 解析动作主要针对类或接口、字段、类方法、接口方法、方法类型等。对应常量池中的`CONSTANT_Class_info`/`CONSTANT_Fieldref_info`、`CONSTANT_Methodref_info`等。

### 3、初始化

- 初始化阶段就是执行类构造器方法 clInit() 的过程。 clInit 是 ClassInit 缩写。此方法并不是程序员定义的构造方法。

- clInit() 不需定义，是 javac 编译器自动收集类中的所有类变量(Static)的赋值动作和静态代码块中的语句合并而来。

- 构造器方法中指令按语句在源文件中出现的顺序执行。

- 若该类具有父类，jvm 会保证子类的 clinit() 执行前，父类的 clinit() 已经执行完毕。

  如下代码：

  ```java
  /**
   * 若有父类 ，jvm 会保证子类的 执行前，父类的 clInit() 已经执行完毕
   * @author qinweizhao
   * @since 2021/12/17
   */
  public class ClinitMethod {
      static class Father {
          public static int A = 1;
  
          static {
              A = 2;
          }
      }
  
      static class Son extends Father {
          public static int B = A;
      }
  
      public static void main(String[] args) {
          // 加载 Father 类，其次加载 Son 类。
          // 2
          System.out.println(Son.B);
      }
  }
  ```

  通过执行，发现 Father 类中 A 的值为20 由于是父类的 CInit 方法先执行，也就是说父类的静态代码块中的内容优于子类的赋值操作先执行。

- 虚拟机必须保证一个类的 clinit() 方法在多线程下被同步加锁。

  验证

  ```java
  /**
   * 虚拟机必须保证一个类的 clinit<>() 方法在多线程下被同步加锁。
   * @author qinweizhao
   * @since 2021/12/17
   */
  public class ClinitThread {
  
      /**
       * 只能执行一次初始化的过程，即同步加锁的过程
       * @param args args
       */
      public static void main(String[] args) {
          Runnable r = () -> {
              System.out.println(Thread.currentThread().getName() + "开始");
              DeadThread dead = new DeadThread();
              System.out.println(Thread.currentThread().getName() + "结束");
          };
  
          Thread t1 = new Thread(r,"线程1");
          Thread t2 = new Thread(r,"线程2");
  
          t1.start();
          t2.start();
      }
  }
  
  class DeadThread{
      static{
          if(true){
              System.out.println(Thread.currentThread().getName() + "初始化当前类");
              while(true){
  
              }
          }
      }
  }
  ```
  
  执行结果：当一条线程死循环在CInit处，别的线程也会阻塞。
  
  ```text
  线程1开始
  线程2开始
  线程1初始化当前类
  ```

## 三、类加载器的分类

- JVM严格来讲支持两种类型的类加载器 。分别为引导类加载器（Bootstrap ClassLoader）和自定义类加载器（User-Defined ClassLoader）。
- 从概念上来讲，自定义类加载器一般指的是程序中由开发人员自定义的一类类加载器，但是 Java 虚拟机规范却没有这么定义，而是**将所有派生于抽象类ClassLoader的类加载器都划分为自定义类加载器**。

- 无论类加载器的类型如何划分，在程序中我们最常见的类加载器始终只有三个，如下所示：

  ![2022-03-31_150130](https://img.qinweizhao.com/2022/03/2022-03-31_150130.png)

  四者之间的关系是包含关系，不是上层下层，也不是子父类的继承关系。

### 1、启动类加载器

- 负责加载`JAVA_HOME/lib`目录下的可以被虚拟机识别（通过文件名称，比如`rt.jar``tools.jar`）的字节码文件。
- 与之对应的是`java.lang.ClassLoader`类 

### 2、扩展类加载器

- 负责加载`JAVA_HOME/lib/ext`目录下的的字节码文件。
- 对应`sun.misc.Launcher`类 此类继承于启动类加载器`ClassLoader`

### 3、应用程序类加载器

- 负责加载`ClassPath`路径下的字节码 也就是用户自己写的类。
- 对应于`sun.misc.Launcher.AppClassLoader`类  此类继承于扩展类加载器`Launcher`

### 4、用户自定义加载器

- 需要继承系统类加载器`ClassLoader`，并重写`findClass`方法。
- 负责加载指定位置的字节码文件。通过类中的path变量指定。

**代码：**

```java
/**
 * 类加载器的加载路径
 * @author qinweizhao
 * @since 2021/12/17
 */
public class ClassLoaderPath {

    public static void main(String[] args) {
        System.out.println("**********启动类加载器**************");

        //获取 BootstrapClassLoader 能够加载的 api 的路径
        URL[] urLs = sun.misc.Launcher.getBootstrapClassPath().getURLs();
        for (URL element : urLs) {
            System.out.println(element.toExternalForm());
        }

        //从上面的路径中随意选择一个类,来看看他的类加载器是什么:引导类加载器
        ClassLoader classLoader = Provider.class.getClassLoader();
        System.out.println(classLoader);

        System.out.println("***********扩展类加载器*************");
        String extDirs = System.getProperty("java.ext.dirs");
        for (String path : extDirs.split(";")) {
            System.out.println(path);
        }

        //从上面的路径中随意选择一个类,来看看他的类加载器是什么:扩展类加载器
        ClassLoader classLoader1 = CurveDB.class.getClassLoader();
        //sun.misc.Launcher$ExtClassLoader@1540e19d
        System.out.println(classLoader1);

    }
}
```

## 四、类加载器的常用方法

### 1、常用方法

####  ClassLoader 类，它是一个抽象类，其后所有的类加载器都继承自 ClassLoader（不包括启动类加载器）

| 方法名称                                             | 描述                                                         |
| ---------------------------------------------------- | ------------------------------------------------------------ |
| getParent（）                                        | 返回该类加载器的超类加载器                                   |
| loadClass（String name）                             | 加载名称为name的类，返回结果为java.lang.Class类的实例        |
| findClass（String name）                             | 查找名称为name的类，返回结果为java.lang.Class类的实例        |
| findLoadedClass（String name）                       | 查找名称为name的已经被加载过的类，返回结果为java.lang.Class类的实例 |
| defineClass（String name，byte[] b,int off,int len） | 把字节数组b中的内容转换为一个Java类 ，返回结果为java.lang.Class类的实例 |
| resolveClass（Class<?> c）                           | 连接指定的一个java类                                         |

### 2、ClassLoader 继承关系

![2022-03-31_145852](https://img.qinweizhao.com/2022/03/2022-03-31_145852.png)

### 3、获取 ClassLoader 的途径

![2022-03-31_145918](https://img.qinweizhao.com/2022/03/2022-03-31_145918.png)

```java
/**
 * jvm 支持两种类型的类加载器。分别为引导类加载器和自定义类加载器
 * @author qinweizhao
 * @since 2021/12/17
 */
public class GetClassLoader {

    public static void main(String[] args) {

        // 获取系统类加载器
        ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
        // jdk 1.8 sun.misc.Launcher$AppClassLoader@18b4aac2
        // jdk 11 jdk.internal.loader.ClassLoaders$AppClassLoader@2437c6dc
        System.out.println(systemClassLoader);

        // 获取上的类加载器
        // 扩展类加载器
        ClassLoader extClassLoader = systemClassLoader.getParent();
        // jdk 1.8 sun.misc.Launcher$ExtClassLoader@74a14482
        // jdk 11 jdk.internal.loader.ClassLoaders$PlatformClassLoader@96532d6
        System.out.println(extClassLoader);

        // 获取上的类加载器
        // 引导类加载器(无法获取)
        ClassLoader bootstrapClassLoader = extClassLoader.getParent();
        // null
        System.out.println(bootstrapClassLoader);


        // 获取用户自定义的类加载器
        // 默认使用系统类加载器
        ClassLoader classLoader = GetClassLoader.class.getClassLoader();
        // jdk 1.8 sun.misc.Launcher$AppClassLoader@18b4aac2
        // jdk 11 jdk.internal.loader.ClassLoaders$AppClassLoader@2437c6dc
        System.out.println(classLoader);

        // String 类使用的加载器
        ClassLoader stringClassLoader = String.class.getClassLoader();
        // null (Java的核心类库都是使用引导类加载器进行加载的)
        System.out.println(stringClassLoader);
    }
}
```

## 五、双亲委派机制

### 1、介绍

Java 虚拟机对 class 文件采用的是按需加载的方式，也就是说当需要使用该类时才会将它的class文件加载到内存生成的class对象。而且加载某个类的class文件时，java虚拟机采用的是双亲委派模式。即把请求交由父类处理，它是一种任务委派模式。

### 2、工作原理

![2022-03-31_145842](https://img.qinweizhao.com/2022/03/2022-03-31_145842.png)

1. ##### 如果一个类加载器收到了类加载的请求，它并不会自己加载，而是先把请求委托给父类的加载器执行

2. ##### 如果父类加载器还有父类，则进一步向上委托，依次递归，请求到达最顶层的引导类加载器。

3. ##### 如果顶层类的加载器加载成功，则成功返回。如果失败，则子加载器会尝试加载。直到加载成功。

### 3、代码验证

通过查看最顶层父类 ClassLoader 的 loaderClass 方法，我们可以验证双亲委派机制。

```java
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // 首先检查此类是否被加载过了 
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    // 调用父类的加载器方法
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        // 此时是最顶级的启动类加载器
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // 抛出异常说明父类无法加载
                }

                if (c == null) {
                    //父类无法加载的时候，由子类进行加载。
                    // to find the class.
                    long t1 = System.nanoTime();
                    c = findClass(name);
                    //记录加载时间已经加载耗时
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```

### 4、双亲委派机制优势

- 避免类的重复加载

  当自己程序中定义了一个和Java.lang包同名的类，此时，由于使用的是双亲委派机制，会由启动类加载器去加载`JAVA_HOME/lib`中的类，而不是加载用户自定义的类。此时，程序可以正常编译，但是自己定义的类无法被加载运行。

- 保护程序安全，防止核心API被随意篡改

## 六、沙箱安全机制

- 自定义String类时：在加载自定义String类的时候会率先使用引导类加载器加载，而引导类加载器在加载的过程中会先加载jdk自带的文件（rt.jar包中java.lang.String.class），报错信息说没有main方法，就是因为加载的是 rt.jar 包中的 String 类。

- 这样可以保证对 java 核心源代码的保护，这就是沙箱安全机制。

## 七、其他

### 1、如何判断两个class对象是否相同

如何判断两个 class 对象是否相同？在 JVM 中表示两个 class 对象是否为同一个类存在的两个必要条件：

- 类的完整类名必须一致，包括包名
- 加载这个类的 ClassLoader（指ClassLoader实例对象）必须相同。
- 在 jvm 中，即使这两个类对象（class对象）来源同一个 class 文件，被同一个虚拟机所加载，但只要加载它们的 ClassLoader 实例对象不同，那么这两个类对象也是不相等的。

### 2、对类加载器的引用

- JVM 必须知道一个类型是由启动加载器加载的还是由用户类加载器加载的。

- **如果一个类型是由用户类加载器加载的，那么 JVM 会将这个类加载器的一个引用作为类型信息的一部分保存在方法区中。**

- 当解析一个类型到另一个类型的引用的时候，JVM需要保证这两个类型的类加载器是相同的。

### 3、类的主动使用和被动使用

程序对类的使用方式分为：主动使用和被动使用。主动使用的情况：

1. 创建类的实例
2. 访问某各类或接口的静态变量，或者对静态变量赋值
3. 调用类的静态方法
4. 反射 比如 Class.forName(com.dsh.jvm.xxx)
5. 初始化一个类的子类
6. java 虚拟机启动时被标明为启动类的类
7. JDK 7 开始提供的动态语言支持：java.lang.invoke.MethodHandle 实例的解析结果REF_getStatic、REF_putStatic、REF_invokeStatic句柄对应的类没有初始化，则初始化。
