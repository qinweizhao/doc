# Stream

## 一、什么是 Stream

​		Stream 将要处理的元素集合看作一种流，在流的过程中，借助 Stream API 对流中的元素进行操作。

## 二、Stream的创建

1. 通过 `java.util.Collection.stream()` 方法用集合创建流

```java
		List<String> list = Arrays.asList("a", "b", "c");
        // 创建一个顺序流
        Stream<String> stream1 = list.stream();
        stream1.forEach(System.out::println);
        // 创建一个并行流
        Stream<String> parallelStream1 = list.parallelStream();
        parallelStream1.forEach(System.out::println);
```

输出结果：

![2021-08-23_144245](../../img/2021/08/2021-08-23_144245.png)

2. 使用`java.util.Arrays.stream(T[] array)`方法用数组创建流

```java
        int[] array={1,3,5,6,8};
        IntStream stream2 = Arrays.stream(array);
        stream2.forEach(System.out::println);
```

输出结果：

![2021-08-23_144324](../../img/2021/08/2021-08-23_144324.png)

3. 使用`Stream`的静态方法：`of()、iterate()、generate()`

```java
        Stream<Integer> stream3 = Stream.of(1, 2, 3, 4, 5, 6);
        stream3.forEach(System.out::println);
            // iterate()
        Stream<Integer> stream4 = Stream.iterate(0, (x) -> x + 3).limit(4);
        stream4.forEach(System.out::println);
            // generate()
        Stream<Double> stream5 = Stream.generate(Math::random).limit(3);
        stream5.forEach(System.out::println);
```

输出结果：

![2021-08-23_144344](../../img/2021/08/2021-08-23_144344.png)

### 三、Stream 的使用

