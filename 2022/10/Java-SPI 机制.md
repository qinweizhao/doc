# Java-SPI 机制

SPI（Service Provider Interface） ，直译过来就是： 服务提供接口。是 JDK 内置的一种服务提供发现机制，可以用来启用框架扩展和替换组件，主要是被框架的开发人员使用，不同厂商可以针对同一接口做出不同的实现。最常见的 SPI 机制实践案例就是 JDBC 的驱动加载可以根据不同的数据库厂商来引入不同的 JDBC 驱动包。Java 中 SPI 机制主要思想是将装配的控制权移到程序之外，在模块化设计中这个机制尤其重要，其核心思想就是解耦。

## 一、准备

接口：

```java
package com.qinweizhao.other.spi.service;

/**
 * @author qinweizhao
 * @since 2022/10/19
 */
public interface Animal {

    String getName();

}
```

实现：

```java
package com.qinweizhao.other.spi.service.impl;

import com.qinweizhao.other.spi.service.Animal;

/**
 * @author qinweizhao
 * @since 2022/10/19
 */
public class Cat implements Animal {

    @Override
    public String getName() {
        return "Cat";
    }

}
```

```java
package com.qinweizhao.other.spi.service.impl;

import com.qinweizhao.other.spi.service.Animal;

/**
 * @author qinweizhao
 * @since 2022/10/19
 */
public class Dog implements Animal {

    @Override
    public String getName() {
        return "Dog";
    }

}
```

## 二、配置

1. 在 src/main/resources/ 下建立目录： /META-INF/services ，位置和名字固定。
2. 在 /META-INF/services 目录下创建一个以接口全限定类名为名的文件。
3. 将接口实现类的全限定类名写入到创建的文件中，一个实现占一行。

![2022-10-19_215623](https://img.qinweizhao.com/2022/10/2022-10-19_215623.png)

## 三、加载

通过 ServiceLoader 进行加载：

```java
package com.qinweizhao.other.spi;

import com.qinweizhao.other.spi.service.Animal;

import java.util.ServiceLoader;

/**
 * @author qinweizhao
 * @since 2022/10/19
 */
public class SpiMain {

    public static void main(String[] args) {

        ServiceLoader<Animal> load = ServiceLoader.load(Animal.class);

        for (Animal next : load) {
            System.out.println(next.getName());
        }
    }

}
```

输出：

![2022-10-19_215710](https://img.qinweizhao.com/2022/10/2022-10-19_215710.png)

## 四、原理

1. 传入当前线程的类加载器，和这个class对象，构造一个新的ServiceLoader对象

   ```java
       public static <S> ServiceLoader<S> load(Class<S> service,
                                               ClassLoader loader)
       {
           return new ServiceLoader<>(service, loader);
       }
   ```

2. 在 ServiceLoader 构造方法中，new 一个新的 LazyIterator(service, Loader)

   ```java
       public void reload() {
           providers.clear();
           lookupIterator = new LazyIterator(service, loader);
       }
   ```

3. LazyIterator 是一个私有的内部类，其作用是扫描（包括所有引用的 jar 包里的 META- INF/services/ 目录下的配置文件并 parse 其中的配置的所有的 service 的名字

   ```java
           private boolean hasNextService() {
               if (nextName != null) {
                   return true;
               }
               if (configs == null) {
                   try {
                       String fullName = PREFIX + service.getName();
                       if (loader == null)
                           configs = ClassLoader.getSystemResources(fullName);
                       else
                           configs = loader.getResources(fullName);
                   } catch (IOException x) {
                       fail(service, "Error locating configuration files", x);
                   }
               }
               while ((pending == null) || !pending.hasNext()) {
                   if (!configs.hasMoreElements()) {
                       return false;
                   }
                   pending = parse(service, configs.nextElement());
               }
               nextName = pending.next();
               return true;
           }
   ```

4. 将配置文件中所有配置的类的全限定名，通过反射加载，实例化并且加载到缓存中去

   ```java
           private S nextService() {
               if (!hasNextService())
                   throw new NoSuchElementException();
               String cn = nextName;
               nextName = null;
               Class<?> c = null;
               try {
                   c = Class.forName(cn, false, loader);
               } catch (ClassNotFoundException x) {
                   fail(service,
                        "Provider " + cn + " not found");
               }
               if (!service.isAssignableFrom(c)) {
                   fail(service,
                        "Provider " + cn  + " not a subtype");
               }
               try {
                   S p = service.cast(c.newInstance());
                   providers.put(cn, p);
                   return p;
               } catch (Throwable x) {
                   fail(service,
                        "Provider " + cn + " could not be instantiated",
                        x);
               }
               throw new Error();          // This cannot happen
           }
   ```

## 五、总结

**弊端**：

- 只能遍历所有的实现，并全部实例化。
- 配置文件中只是简单的列出了所有的扩展实现，而没有给他们命名。导致在程序中很难去准确的引用它们。
- 扩展很难和其他的框架集成，比如扩展里面依赖了一个Spring bean，原生的 Java SPI 不支持。

**好处**：

- 不需要改动源码就可以实现扩展，解耦。
- 实现扩展对原来的代码几乎没有侵入性。
- 只需要添加配置就可以实现扩展，符合开闭原则。

SPI 机制为我们的程序提供拓展功能。而不必将框架的一些实现类写死在代码里面。我们在相应配置文件中定义好某个接口的实现类全限定名，并由服务加载器读取配置文件，加载实现类。这样可以在运行时，动态为接口替换实现类。除了 Java 的 SPI 机制，另外，还有 Dubbo 和 SpringBoot 自定义的 SPI 机制。

## 附

>代码地址：
>
>https://github.com/qinweizhao/qwz-sample/tree/master/basic/b-other









