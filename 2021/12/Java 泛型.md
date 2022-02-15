

# Java 泛型

>Java 泛型（generics）是 JDK 5 中引入的一个新特性, 泛型提供了编译时类型安全检测机制，该机制允许开发者在编译时检测到非法的类型。
>
>泛型的本质是参数化类型，也就是说所操作的数据类型被指定为一个参数。

## 一、泛型带来的好处

在没有泛型的情况的下，通过对类型 Object 的引用来实现参数的“任意化”，“任意化”带来的缺点是要做显式的强制类型转换，而这种转换是要求开发者对实际参数类型可以预知的情况下进行的。对于强制类型转换错误的情况，编译器可能不提示错误，在运行的时候才出现异常，这是本身就是一个安全隐患。那么泛型的好处就是在编译的时候能够检查类型安全，并且所有的强制转换都是自动和隐式的。

```java
public class GlmapperGeneric<T> {
  private T t;
    public void set(T t) { this.t = t; }
    public T get() { return t; }

    public static void main(String[] args) {
        // do nothing
    }

  /**
    * 不指定类型
    */
  public void noSpecifyType(){
    GlmapperGeneric glmapperGeneric = new GlmapperGeneric();
    glmapperGeneric.set("test");
    // 需要强制类型转换
    String test = (String) glmapperGeneric.get();
    System.out.println(test);
  }

  /**
    * 指定类型
    */
  public void specifyType(){
    GlmapperGeneric<String> glmapperGeneric = new GlmapperGeneric();
    glmapperGeneric.set("test");
    // 不需要强制类型转换
    String test = glmapperGeneric.get();
    System.out.println(test);
  }
}
```

上面这段代码中的 specifyType 方法中省去了强制转换，可以在编译时候检查类型安全，可以用在类，方法，接口上。

## 二、泛型的原理

Java 语言的泛型采用的是**擦除法**实现的**伪泛型**，泛型信息（类型变量、参数化类型）编译之后通通被除掉了。

**类型擦除**

泛型信息只存在于编译前，编译后的字节码中是不包含泛型中的类型信息的。因此，编译器在编译时去掉类型参数，叫做类型擦除。

例如 List<Integer> 和 List<String> 等类型在编译之后都会变成 List。JVM 看到的只是 List，而泛型信息对 JVM 来说是不可见的。

```
Class class1= new ArrayList<String>().getClass();
Class class2= new ArrayList<Integer>().getClass();
System.out.println(class1 == class2);  // true
```

两个 ArrayList 对象相等，既 JVM 认为这是同一类型。

## 三、泛型类

>泛型类的类型参数声明部分也包含一个或多个类型参数，参数间用逗号隔开。一个泛型参数，也被称为一个类型变量，是用于指定一个泛型类型名称的标识符。因为他们接受一个或多个参数，这些类被称为参数化的类或参数化的类型。

```java
package com.qinweizhao.generics;

/**
 * 泛型类
 * @author qinweizhao
 * @since 2021/12/29
 */
public class BoxGeneric<T> {

    private T t;

    public void add(T t) {
        this.t = t;
    }

    public T get() {
        return t;
    }

    public static void main(String[] args) {
        BoxGeneric<Integer> integerBoxGeneric = new BoxGeneric<Integer>();
        BoxGeneric<String> stringBoxGeneric = new BoxGeneric<String>();

        integerBoxGeneric.add(10);
        stringBoxGeneric.add("Java 泛型");

        System.out.printf("整型值为 :%d/n/n", integerBoxGeneric.get());
        System.out.printf("字符串为 :%s/n", stringBoxGeneric.get());
    }
}
```

运行结果：

![2021-12-29_164719](https://img.qinweizhao.com/2021/12/2021-12-29_164719.png)

## 四、泛型方法

- 所有泛型方法声明都有一个类型参数声明部分（由尖括号分隔），该类型参数声明部分在方法返回类型之前（在下面例子中的 **<E>**）。
- 每一个类型参数声明部分包含一个或多个类型参数，参数间用逗号隔开。一个泛型参数，也被称为一个类型变量，是用于指定一个泛型类型名称的标识符。
- 类型参数能被用来声明返回值类型，并且能作为泛型方法得到的实际参数类型的占位符。
- 泛型方法体的声明和其他方法一样。注意类型参数只能代表引用型类型，不能是原始类型（像 **int、double、char** 等）。

```java
package com.qinweizhao.generics;

/**
 * 泛型方法
 * @author qinweizhao
 * @since 2021/12/29
 */
public class MethodGeneric {

    /**
     * 泛型方法 printArray
     *
     * @param inputArray inputArray
     * @param <E>        E
     */
    public static <E> void printArray(E[] inputArray) {
        // 输出数组元素
        for (E element : inputArray) {
            System.out.printf("%s ", element);
        }
        System.out.println();
    }

    public static void main(String[] args) {
        // 创建不同类型数组： Integer, Double 和 Character
        Integer[] intArray = {1, 2, 3, 4, 5};
        Double[] doubleArray = {1.1, 2.2, 3.3, 4.4};
        Character[] charArray = {'H', 'E', 'L', 'L', 'O'};

        System.out.println("整型数组元素为:");
        // 传递一个整型数组
        printArray(intArray);

        System.out.println("/n双精度型数组元素为:");
        // 传递一个双精度型数组
        printArray(doubleArray);

        System.out.println("/n字符型数组元素为:");
        // 传递一个字符型数组
        printArray(charArray);
    }
}

```

运行结果：

![2021-12-29_165147](https://img.qinweizhao.com/2021/12/2021-12-29_165147.png)

## 五、泛型标识符

Java 中泛型标记符：

- **E** - Element (在集合中使用，因为集合中存放的是元素)
- **T** - Type（Java 类）
- **K** - Key（键）
- **V** - Value（值）
- **N** - Number（数值类型）
- **？** - 表示不确定的 Java 类型

### 1、？无界通配符

一个父类 Animal 和几个子类，如狗、猫等，现在我需要一个动物的列表：

```java
List<? extends Animal> listAnimals
```

通配符其实在声明局部变量时是没有什么意义的，但是当为一个方法声明一个参数时，是非常重要的。

```java
package com.qinweizhao.generics;

import com.qinweizhao.generics.base.Animal;
import com.qinweizhao.generics.base.Dog;

import java.util.ArrayList;
import java.util.List;

/**
 * @author qinweizhao
 * @since 2021/12/29
 */
public class Wildcard {

    static int countLegs(List<? extends Animal> animals) {
        int retVal = 0;
        for (Animal animal : animals) {
            retVal += animal.countLegs();
        }
        return retVal;
    }

    static int countLegs1(List<Animal> animals) {
        int retVal = 0;
        for (Animal animal : animals) {
            retVal += animal.countLegs();
        }
        return retVal;
    }

    public static void main(String[] args) {
        List<Dog> dogs = new ArrayList<>();
        // 不会报错
        countLegs(dogs);
        // 报错
        // countLegs1(dogs);
    }
}
https://img.qinweizhao.com/2021/12/
```

当调用 countLegs1 时，就会飘红，提示的错误信息如下：

![2021-12-29_171743](https://img.qinweizhao.com/2021/12/2021-12-29_171743.png)

所以，对于不确定或者不关心实际要操作的类型，可以使用无限制通配符（尖括号里一个问号，即 <?> ），表示可以持有任何类型。像 countLegs 方法中，限定了上界，但是不关心具体类型是什么，所以对于传入的 Animal 的所有子类都可以支持，并且不会报错。而 countLegs1 就不行。

### 2、上界通配符 < ? extends E>

> 上界：用 extends 关键字声明，表示参数化的类型可能是所指定的类型，或者是此类型的子类。

在类型参数中使用 extends 表示这个泛型中的参数必须是 E 或者 E 的子类，这样有两个好处：

- 如果传入的类型不是 E 或者 E 的子类，编译不成功
- 泛型中可以使用 E 的方法，要不然还得强转成 E 才能使用

```java
private <K extends A, E extends B> E test(K arg1, E arg2){
    E result = arg2;
    arg2.compareTo(arg1);
    //.....
    return result;
}
```

> 类型参数列表中如果有多个类型参数上限，用逗号分开

### 3、下界通配符 < ? super E>

> 下界: 用 super 进行声明，表示参数化的类型可能是所指定的类型，或者是此类型的父类型，直至 Object

在类型参数中使用 super 表示这个泛型中的参数必须是 E 或者 E 的父类。

```java
private <T> void test(List<? super T> dst, List<T> src){
    for (T t : src) {
        dst.add(t);
    }
}

public static void main(String[] args) {
    List<Dog> dogs = new ArrayList<>();
    List<Animal> animals = new ArrayList<>();
    new Test3().test(animals,dogs);
}
// Dog 是 Animal 的子类
class Dog extends Animal {

}
```

dst 类型 “大于等于” src 的类型，这里的“大于等于”是指 dst 表示的范围比 src 要大，因此装得下 dst 的容器也就能装 src 。

### 4、？和 T 的区别

```java
// 指定集合元素只能是 T 类型
List<T> list = new ArrayList<>();
// 集合元素可以是任意类型，这种没有意义，一般是方法中，只是为了说明用法
List<?> list = new ArrayList<>();
```

？和 T 都表示不确定的类型，区别在于我们可以对 T 进行操作，但是对 ？不行，比如如下这种 ：

```
// 可以
T t = operate();

// 不可以
？car = operate();
```

简单总结下：

T 是一个确定的类型，通常用于泛型类和泛型方法的定义，？是一个 不确定 的类型，通常用于泛型方法的调用代码和形参，不能用于定义类和泛型方法。

#### 区别1：通过 T 来确保泛型参数的一致性

```
// 通过 T 来确保泛型参数的一致性
public <T extends Number> void test(List<T> dest, List<T> src)

// 通配符是不确定的，所以这个方法不能保证两个 List 具有相同的元素类型
public void test(List<? extends Number> dest, List<? extends Number> src)
```

#### 区别2：类型参数可以多重限定而通配符不行

```java
package com.qinweizhao.generics;

/**
 * @author qinweizhao
 * @since 2021/12/29
 */
public class MultiLimit implements MultiLimitInterfaceA, MultiLimitInterfaceB {

    /**
     * 符号设定多重边界
     *
     * @param t   t
     * @param <T> T
     */
    public static <T extends MultiLimitInterfaceA & MultiLimitInterfaceB> void test(T t) {

    }

}

/***接口A*/
interface MultiLimitInterfaceA {
}

/*** i口в*/
interface MultiLimitInterfaceB {
}
```

使用 & 符号设定多重边界（Multi Bounds)，指定泛型类型 T 必须是 MultiLimitInterfaceA 和 MultiLimitInterfaceB 的共有子类型，此时变量 t 就具有了所有限定的方法和属性。对于通配符来说，因为它不是一个确定的类型，所以不能进行多重限定。

#### 区别3：通配符可以使用超类限定而类型参数不行

类型参数 T 只具有 一种 类型限定方式：

```
T extends A
```

但是通配符 ? 可以进行 两种限定：

```
? extends A
? super A
```

### 5、`Class<T>` 和 `Class<?>` 区别

前面介绍了 ？和 T 的区别，那么对于，`Class<T>`和 `<Class<?>`又有什么区别呢？`Class<T>`和 `Class<?>`

最常见的是在反射场景下的使用，这里以用一段发射的代码来说明下。

```
// 通过反射的方式生成  multiLimit
// 对象，这里比较明显的是，我们需要使用强制类型转换
MultiLimit multiLimit = (MultiLimit) Class.forName("com.glmapper.bridge.boot.generic.MultiLimit").newInstance();
```

对于上述代码，在运行期，如果反射的类型不是 MultiLimit 类，那么一定会报 java.lang.ClassCastException 错误。

对于这种情况，则可以使用下面的代码来代替，使得在在编译期就能直接检查到类型的问题：

```java
package com.qinweizhao.generics;

/**
 * @author qinweizhao
 * @since 2021/12/29
 */
public class Test {

    public static <T> T createInstance(Class<T> clazz) throws IllegalAccessException, InstantiationException {
        return clazz.newInstance();
    }

    public static void main(String[] args) throws
            InstantiationException, IllegalAccessException {
        A a = createInstance(A.class);
        B b = createInstance(B.class);
    }

}

class A {
}

class B {
}
```

`Class<T>`在实例化的时候，T 要替换成具体类。`Class<?>`它是个通配泛型，? 可以代表任何类型，所以主要用于声明时的限制情况。比如，我们可以这样做申明：

```java
// 可以
public Class<?> clazz;
// 不可以，因为 T 需要指定类型
public Class<T> clazzT;
```

所以当不知道定声明什么类型的 Class 的时候可以定义一 个Class<?>。

那如果也想 `public Class<T> clazzT;`这样的话，就必须让当前的类也指定 T ，

```java
public class Test3<T> {
    public Class<?> clazz;
    // 不会报错
    public Class<T> clazzT;
```

##  

>代码地址：
>
>https://github.com/qinweizhao/qwz-sample/tree/master/basic/b-generics

