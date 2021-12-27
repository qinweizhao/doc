## 关于 Collectors.toMap 方法

Collectors.toMap 有三个重载方法：

```
toMap(Function<? super T, ? extends K> keyMapper, Function<? super T, ? extends U> valueMapper);
toMap(Function<? super T, ? extends K> keyMapper, Function<? super T, ? extends U> valueMapper,
        BinaryOperator<U> mergeFunction);
toMap(Function<? super T, ? extends K> keyMapper, Function<? super T, ? extends U> valueMapper,
        BinaryOperator<U> mergeFunction, Supplier<M> mapSupplier);
```

参数含义分别是：

1. keyMapper：Key 的映射函数
2. valueMapper：Value 的映射函数
3. mergeFunction：当 Key 冲突时，调用的合并方法
4. mapSupplier：Map 构造器，在需要返回特定的 Map 时使用

还是用上面的例子，如果 List 中 userId 有相同的，使用上面的写法会抛异常：

```
List<User> userList = Lists.newArrayList(
        new User().setId("A").setName("张三"),
        new User().setId("A").setName("李四"), // Key 相同 
        new User().setId("C").setName("王五")
);
userList.stream().collect(Collectors.toMap(User::getId, User::getName));

// 异常：
java.lang.IllegalStateException: Duplicate key 张三 
    at java.util.stream.Collectors.lambda$throwingMerger$114(Collectors.java:133)
    at java.util.HashMap.merge(HashMap.java:1245)
    at java.util.stream.Collectors.lambda$toMap$172(Collectors.java:1320)
    at java.util.stream.ReduceOps$3ReducingSink.accept(ReduceOps.java:169)
    at java.util.ArrayList$ArrayListSpliterator.forEachRemaining(ArrayList.java:1374)
    at java.util.stream.AbstractPipeline.copyInto(AbstractPipeline.java:481)
    at java.util.stream.AbstractPipeline.wrapAndCopyInto(AbstractPipeline.java:471)
    at java.util.stream.ReduceOps$ReduceOp.evaluateSequential(ReduceOps.java:708)
    at java.util.stream.AbstractPipeline.evaluate(AbstractPipeline.java:234)
    at java.util.stream.ReferencePipeline.collect(ReferencePipeline.java:499)
    at Test.toMap(Test.java:17)
    ...
```

这时就需要调用第二个重载方法，传入合并函数，如：

```
userList.stream().collect(Collectors.toMap(User::getId, User::getName, (n1, n2) -> n1 + n2));

// 输出结果：
A-> 张三李四 
C-> 王五 
```

第四个参数（mapSupplier）用于自定义返回 Map 类型，比如我们希望返回的 Map 是根据 Key 排序的，可以使用如下写法：

```
List<User> userList = Lists.newArrayList(
        new User().setId("B").setName("张三"),
        new User().setId("A").setName("李四"),
        new User().setId("C").setName("王五")
);
userList.stream().collect(
    Collectors.toMap(User::getId, User::getName, (n1, n2) -> n1, TreeMap::new)
);

// 输出结果：
A-> 李四 
B-> 张三 
```