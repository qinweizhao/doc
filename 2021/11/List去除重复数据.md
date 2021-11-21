# List去除重复数据

去除 Java 中 ArrayList 中的重复数据

### 一、使用 LinkedHashSet 删除 arraylist 中的重复数据

LinkedHashSet 是在一个 ArrayList 删除重复数据的最佳方法。LinkedHashSet 在内部完成两件事：

- 删除重复数据
- 保持添加到其中的数据的顺序

Java 示例使用 LinkedHashSet 删除 arraylist 中的重复项。在给定的示例中，numbersList 是包含整数的 arraylist，其中一些是重复的数字。

例如 1,3 和 5. 我们将列表添加到 LinkedHashSet，然后将内容返回到列表中。结果 arraylist 没有重复的整数。

```
import java.util.ArrayList;
import java.util.Arrays;
import java.util.LinkedHashSet;
 
public class ArrayListExample
{
    public static void main(String[] args)
    {
 
        ArrayList<Integer> numbersList = new ArrayList<>(Arrays.asList(1, 1, 2, 3, 3, 3, 4, 5, 6, 6, 6, 7, 8));
 
        System.out.println(numbersList);
 
        LinkedHashSet<Integer> hashSet = new LinkedHashSet<>(numbersList);
 
        ArrayList<Integer> listWithoutDuplicates = new ArrayList<>(hashSet);
 
        System.out.println(listWithoutDuplicates);
    }
}

复制代码
```

输出结果

```
[1, 1, 2, 3, 3, 3, 4, 5, 6, 6, 6, 7, 8]
 
[1, 2, 3, 4, 5, 6, 7, 8]

复制代码
```

### 二、使用 java8 新特性 stream 进行 List 去重

要从 arraylist 中删除重复项，我们也可以使用 java 8 stream api。使用 steam 的 distinct() 方法返回一个由不同数据组成的流，通过对象的 equals（）方法进行比较。

收集所有区域数据 List 使用 Collectors.toList()。

Java 程序，用于在不使用 Set 的情况下从 java 中的 arraylist 中删除重复项。

```
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;
 
public class ArrayListExample
{
    public static void main(String[] args)
 
    {
 
        ArrayList<Integer> numbersList = new ArrayList<>(Arrays.asList(1, 1, 2, 3, 3, 3, 4, 5, 6, 6, 6, 7, 8));
        System.out.println(numbersList);
        List<Integer> listWithoutDuplicates = numbersList.stream().distinct().collect(Collectors.toList());
 
        System.out.println(listWithoutDuplicates);
 
    }
 
}

复制代码
```

输出结果

```
[1, 1, 2, 3, 3, 3, 4, 5, 6, 6, 6, 7, 8]
 
[1, 2, 3, 4, 5, 6, 7, 8]

复制代码
```

### 三、利用 HashSet 不能添加重复数据的特性 由于 HashSet 不能保证添加顺序，所以只能作为判断条件保证顺序：

```
private static void removeDuplicate(List<String> list) {
    HashSet<String> set = new HashSet<String>(list.size());
    List<String> result = new ArrayList<String>(list.size());
    for (String str : list) {
        if (set.add(str)) {
            result.add(str);
        }
    }
    list.clear();
    list.addAll(result);
}

复制代码
```

### 四、利用 List 的 contains 方法循环遍历, 重新排序, 只添加一次数据, 避免重复：

```
private static void removeDuplicate(List<String> list) {
    List<String> result = new ArrayList<String>(list.size());
    for (String str : list) {
        if (!result.contains(str)) {
            result.add(str);
        }
    }
    list.clear();
    list.addAll(result);
}

复制代码
```

### 五、双重 for 循环去重

```
for (int i = 0; i < list.size(); i++) { 
for (int j = 0; j < list.size(); j++) { 
if(i!=j&&list.get(i)==list.get(j)) { 
list.remove(list.get(j)); 
 } 
} 
```