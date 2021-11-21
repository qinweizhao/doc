# List 去除重复数据

去除 Java 中 ArrayList 中的重复数据

### 一、使用 LinkedHashSet 删除 ArrayList 中的重复数据

LinkedHashSet 是在一个 ArrayList 删除重复数据的最佳方法。LinkedHashSet 在内部完成两件事：

- 删除重复数据
- 保持添加到其中的数据的顺序

```java
    public static void removeDuplicateOne(List<String> list) {
        LinkedHashSet<String> hashSet = new LinkedHashSet<>(list);
        ArrayList<String> listWithoutDuplicates = new ArrayList<>(hashSet);
        System.out.println(listWithoutDuplicates);
    }
```

### 二、使用 java8 新特性 stream 进行 List 去重

要从 arraylist 中删除重复项，我们也可以使用 java 8 stream api。使用 steam 的 distinct() 方法返回一个由不同数据组成的流，通过对象的 equals（）方法进行比较。

收集所有区域数据 List 使用 Collectors.toList()。

Java 程序，用于在不使用 Set 的情况下从 java 中的 arraylist 中删除重复项。

```java
    public static void removeDuplicateTwo(List<String> list) {
        List<String> listWithoutDuplicates = list.stream().distinct().collect(Collectors.toList());
        System.out.println(listWithoutDuplicates);
    }
```

### 三、利用 HashSet 不能添加重复数据的特性 由于 HashSet 不能保证添加顺序，所以只能作为判断条件保证顺序：

```java
    public static void removeDuplicateThree(List<String> list) {
        HashSet<String> set = new HashSet<>(list.size());
        List<String> result = new ArrayList<>(list.size());
        for (String str : list) {
            if (set.add(str)) {
                result.add(str);
            }
        }
        list.clear();
        list.addAll(result);
    }
```

### 四、利用 List 的 contains 方法循环遍历, 重新排序, 只添加一次数据, 避免重复：

```java
    public static void removeDuplicateFour(List<String> list) {
        List<String> result = new ArrayList<>(list.size());
        for (String str : list) {
            if (!result.contains(str)) {
                result.add(str);
            }
        }
        list.clear();
        list.addAll(result);
    }
```

### 五、双重 for 循环去重

```java
    public static void removeDuplicateFive(List<String> list) {
        for (int i = 0; i < list.size(); i++) {
            for (int j = 0; j < list.size(); j++) {
                if (i != j && Objects.equals(list.get(i), list.get(j))) {
                    list.remove(list.get(j));
                }
            }
        }
    }
```

