#  List 集合去除 null 元素

## 一、使用for循环处理集合。

```java
public static List<Object> removeNull(List<Object> oldList) {
        oldList.removeAll(Collections.singleton(null));
        return oldList;
}
```

## 二、使用Collections工具类

```java
public static List<Object> removeNulls(List<Object> oldList) {
        // 临时集合
        List<Object> listTemp = new ArrayList<>();
        for (Object t : oldList) {
            // 保存不为空的元素
            if (t != null) {
                listTemp.add(t);
            }
        }
        return listTemp;
}
```

## 三、新增一个List集合

```java
public static List<Object> removeNull(List<Object> oldList) {
        List<Object> nullList = new ArrayList<>();
        nullList.add(null);
        oldList.removeAll(nullList);
        return oldList;
}
```

