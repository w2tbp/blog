---
title: java stream 的一些使用心得
date: 2023-07-27 20:01:24
categories:
    - 
tags:
    - java
---

java stream 流处理，有点像 Linux 中的管道。  
把一个集合中的元素当做流，java 提供了一些函数去处理，上一个函数的结果输出可以当做下一个函数输入。

## 寻找符合条件的元素
使用 filter
```java
User target1 = targetList.stream()  
        .filter(o -> uid.equals(o.getId())).findAny().orElse(null); 
         
Optional<User> target2 = targetList.stream()  
        .filter(o -> uid.equals(o.getId())).findAny();  
        
Optional<User> target3 = targetList.stream()  
        .filter(o -> uid.equals(o.getId())).findFirst();
```
findAny findFirst 都可以，视场景而定。  
orElse 设定一个默认值。

例子
```java
@Data  
@AllArgsConstructor  
public class User {  
  
    private Integer id;  
  
    private String name;  
  
    private String country;  
  
}
```

```java
public class StreamTest {  
  
    List<User> targetList = new ArrayList<>();  
  
    public StreamTest() {  
        User coldplay = new User(1, "Coldplay", "uk");  
        User oneOkRock = new User(2, "OneOkRock", "jp");  
        User queen = new User(3, "Queen", "uk");  
        User mayday = new User(4, "Mayday", "cn");  
        User radwimps = new User(5, "RADWIMPS", "jp");  
        User supercell = new User(6, "Supercell", "jp");  
        targetList.addAll(Arrays.asList(coldplay, oneOkRock, queen, mayday, radwimps, supercell));  
    }  
  
    public static void main(String[] args) {  
        StreamTest test = new StreamTest();  
        test.findOne(1);  
    }  
  
    private void findOne(Integer uid) {  
        User target1 = targetList.stream().filter(o -> uid.equals(o.getId())).findAny().orElse(null);  
        Optional<User> target2 = targetList.stream().filter(o -> uid.equals(o.getId())).findAny();  
        Optional<User> target3 = targetList.stream().filter(o -> uid.equals(o.getId())).findFirst();  
    }  
  
}
```

## 根据某个字段去重
如果是一个基础类型的集合，比如 Integer 、String 之类的，直接用 distinct() 方法就好了。但是如果是一个实体类的集合，就需要根据实体类中的某个字段来去重。  
有好几种方式，这里介绍两种。  

1. 通过 `Collectors.collectingAndThen` 的 `Collectors.toCollection` ，里面用 `TreeSet` 在构造函数中指定字段
```java
List<User> distinctList = targetList.stream()  
        .collect(Collectors.collectingAndThen(Collectors.toCollection(() -> new TreeSet<>(Comparator.comparing(o -> o.getCountry()))), ArrayList::new));
```

2. 使用 filter + map
```java
Map<Object, Boolean> map = new HashMap<>();  
List<User> distinctList = targetList.stream()  
        .filter(i -> map.putIfAbsent(i.getCountry(), Boolean.TRUE) == null).collect(Collectors.toList());
```

来源：[List列表运用Java8的stream流按某字段去重 - cdfive - 博客园 (cnblogs.com)](https://www.cnblogs.com/cdfive2018/p/15064524.html)

例子  
```java
public class StreamTest {  
  
    List<User> targetList = new ArrayList<>();  
  
    public StreamTest() {  
        User coldplay = new User(1, "Coldplay", "uk");  
        User oneOkRock = new User(2, "OneOkRock", "jp");  
        User queen = new User(3, "Queen", "uk");  
        User mayday = new User(4, "Mayday", "cn");  
        User radwimps = new User(5, "RADWIMPS", "jp");  
        User supercell = new User(6, "Supercell", "jp");  
        targetList.addAll(Arrays.asList(coldplay, oneOkRock, queen, mayday, radwimps, supercell));  
    }  
  
    public static void main(String[] args) {  
        StreamTest test = new StreamTest();  
        test.distinct1(); 
        test.distinct2(); 
    }  
  
    private void distinct1() {  
        List<User> distinctList = targetList.stream()  
                .collect(Collectors.collectingAndThen(Collectors.toCollection(() -> new TreeSet<>(Comparator.comparing(o -> o.getCountry()))), ArrayList::new));  
        System.out.println(distinctList);  
    }  
  
    private void distinct2() {  
        Map<Object, Boolean> map = new HashMap<>();  
        List<User> distinctList = targetList.stream()  
                .filter(i -> map.putIfAbsent(i.getCountry(), Boolean.TRUE) == null).collect(Collectors.toList());  
        System.out.println(distinctList);  
    }  
  
}
```

## 排序
可以使用提供的一系列方法，例如 `Comparator.comparingInt()` 之类的。  
也可以自定义一个 Comparator 
```java
Comparator<User> comparator = (o1, o2) -> {  
    // 可添加更多的条件  
    return o1.getId() - o2.getId();  
};  
targetList = targetList.stream().sorted(comparator).collect(Collectors.toList());
```

[java8 stream多字段排序 - SugarWater - 博客园 (cnblogs.com)](https://www.cnblogs.com/kuanglongblogs/p/11230250.html)

例子
```java
public class StreamTest {  
  
    List<User> targetList = new ArrayList<>();  
  
    public StreamTest() {  
        User coldplay = new User(1, "Coldplay", "uk");  
        User oneOkRock = new User(2, "OneOkRock", "jp");  
        User queen = new User(3, "Queen", "uk");  
        User mayday = new User(4, "Mayday", "cn");  
        User radwimps = new User(5, "RADWIMPS", "jp");  
        User supercell = new User(6, "Supercell", "jp");  
        targetList.addAll(Arrays.asList(radwimps, mayday, coldplay, queen, supercell, oneOkRock));  
    }  
  
    public static void main(String[] args) {  
        StreamTest test = new StreamTest();  
        test.sort();  
    }  
  
    private void sort() {  
        System.out.println(targetList);  
        Comparator<User> comparator = (o1, o2) -> {  
            // 可添加更多的条件  
            return o1.getId() - o2.getId();  
        };  
        targetList = targetList.stream().sorted(comparator).collect(Collectors.toList());  
        System.out.println(targetList);  
  
        targetList = targetList.stream().sorted(Comparator.comparing(User::getCountry)).collect(Collectors.toList());  
        System.out.println(targetList);  
  
    }  
}
```

## 分组
```java
Map<String, List<User>> map = targetList.stream()  
        .collect(Collectors.groupingBy(User::getCountry));
```

[Java8 Stream之group - 简书 (jianshu.com)](https://www.jianshu.com/p/0687e7003eb2)

例子

```java
public class StreamTest {  
  
    List<User> targetList = new ArrayList<>();  
  
    public StreamTest() {  
        User coldplay = new User(1, "Coldplay", "uk");  
        User oneOkRock = new User(2, "OneOkRock", "jp");  
        User queen = new User(3, "Queen", "uk");  
        User mayday = new User(4, "Mayday", "cn");  
        User radwimps = new User(5, "RADWIMPS", "jp");  
        User supercell = new User(6, "Supercell", "jp");  
        targetList.addAll(Arrays.asList(radwimps, mayday, coldplay, queen, supercell, oneOkRock));  
    }  
  
    public static void main(String[] args) {  
        StreamTest test = new StreamTest();  
        test.groupBy();  
    }  
  
    private void groupBy() {  
        Map<String, List<User>> map = targetList.stream()  
                .collect(Collectors.groupingBy(User::getCountry));  
        System.out.println(map);  
    }  
}
```

## 截取某个字段
在只需要实体类的某一个字段时比较好用，可以在后面再加个 distinct() 去重
```java
List<String> mapping = targetList.stream().map(User::getCountry).collect(Collectors.toList());
```

例子
```java
public class StreamTest {  
  
    List<User> targetList = new ArrayList<>();  
  
    public StreamTest() {  
        User coldplay = new User(1, "Coldplay", "uk");  
        User oneOkRock = new User(2, "OneOkRock", "jp");  
        User queen = new User(3, "Queen", "uk");  
        User mayday = new User(4, "Mayday", "cn");  
        User radwimps = new User(5, "RADWIMPS", "jp");  
        User supercell = new User(6, "Supercell", "jp");  
        targetList.addAll(Arrays.asList(radwimps, mayday, coldplay, queen, supercell, oneOkRock));  
    }  
  
    public static void main(String[] args) {  
        StreamTest test = new StreamTest();  
        test.mapping();  
    }  
  
    private void mapping() {  
        List<String> mapping = targetList.stream().map(User::getCountry).collect(Collectors.toList());  
        System.out.println(mapping);  
        List<String> mapping2 = targetList.stream().map(User::getCountry).distinct().collect(Collectors.toList());  
        System.out.println(mapping2);  
    }  
}
```

## 完整示例
```java
public class StreamTest {  
  
    List<User> targetList = new ArrayList<>();  
  
    public StreamTest() {  
        User coldplay = new User(1, "Coldplay", "uk");  
        User oneOkRock = new User(2, "OneOkRock", "jp");  
        User queen = new User(3, "Queen", "uk");  
        User mayday = new User(4, "Mayday", "cn");  
        User radwimps = new User(5, "RADWIMPS", "jp");  
        User supercell = new User(6, "Supercell", "jp");  
        targetList.addAll(Arrays.asList(radwimps, mayday, coldplay, queen, supercell, oneOkRock));  
    }  
  
    public static void main(String[] args) {  
        StreamTest test = new StreamTest();  
        test.findOne(1);  
        test.distinct1();  
        test.distinct2();  
        test.sort();  
        test.groupBy();  
        test.mapping();  
    }  
  
    private void mapping() {  
        List<String> mapping = targetList.stream().map(User::getCountry).collect(Collectors.toList());  
        System.out.println(mapping);  
        List<String> mapping2 = targetList.stream().map(User::getCountry).distinct().collect(Collectors.toList());  
        System.out.println(mapping2);  
    }  
  
    private void groupBy() {  
        Map<String, List<User>> map = targetList.stream()  
                .collect(Collectors.groupingBy(User::getCountry));  
        System.out.println(map);  
    }  
  
    private void sort() {  
        System.out.println(targetList);  
        Comparator<User> comparator = (o1, o2) -> {  
            // 可添加更多的条件  
            return o1.getId() - o2.getId();  
        };  
        targetList = targetList.stream().sorted(comparator).collect(Collectors.toList());  
        System.out.println(targetList);  
  
        targetList = targetList.stream().sorted(Comparator.comparing(User::getCountry)).collect(Collectors.toList());  
        System.out.println(targetList);  
  
    }  
  
    private void distinct1() {  
        List<User> distinctList = targetList.stream()  
                .collect(Collectors.collectingAndThen(Collectors.toCollection(() -> new TreeSet<>(Comparator.comparing(o -> o.getCountry()))), ArrayList::new));  
        System.out.println(distinctList);  
    }  
  
    private void distinct2() {  
        Map<Object, Boolean> map = new HashMap<>();  
        List<User> distinctList = targetList.stream()  
                .filter(i -> map.putIfAbsent(i.getCountry(), Boolean.TRUE) == null).collect(Collectors.toList());  
        System.out.println(distinctList);  
    }  
  
    private void findOne(Integer uid) {  
        User target1 = targetList.stream()  
                .filter(o -> uid.equals(o.getId())).findAny().orElse(null);  
        Optional<User> target2 = targetList.stream()  
                .filter(o -> uid.equals(o.getId())).findAny();  
        Optional<User> target3 = targetList.stream()  
                .filter(o -> uid.equals(o.getId())).findFirst();  
    }  
  
}
```
用这个东西得发挥想象力，然后有时候会发现...

for 其实挺好用的。