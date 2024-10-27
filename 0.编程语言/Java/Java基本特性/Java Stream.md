

# 概述

Stream代表的是任意Java对象的序列。

> 怎么理解「序列」？
> 
> 可以和`List`做个对比。`List`存储的每个元素都是已经存储在内存中的某个Java对象；而`Stream`输出的元素可能并没有预先存储在内存中，而是实时计算出来的。因此，`Stream`是一种「惰性计算」。由于`Stream`中的元素是实时计算出来的，所以可以把它理解成它「存储」无限的元素。

Stream的特点：

1. 可以「存储」有限个或无限个元素。这里的「存储」是带引号的。
   
2. `Stream`可以转换为另一个`Stream`，而非修改其本身。
   
3. 惰性计算：一个`Stream`转为另一个`Stream`时，实际上只存储了转换规则，并没有任何计算发生。真正的计算发生在最后结果的获取。
   
    ```java
    createNaturalStream()
        .map(BigInteger::multiply)
        .limit(100)
        .forEach(System.out::println);
    ```
    

# 不可变的集合

## 基本概念

不可变的集合是指一旦创建后，集合中的元素不能被添加、删除、或修改的集合。不可变集合的一些使用场景如下：

- 如果某个数据不能被修改，把它防御性地拷贝到不可变集合中是一个很好的实践。
- 当集合对象被不可信的库调用时，不可变形式是安全的。

## 创建不可变集合

在 Java 中，`java.util.Collections.unmodifiableCollection()` 方法可以将一个普通的集合转换为不可变集合。此外，在 `List`、`Set`、`Map` 接口中，都存在静态的 `of` 方法，可以获取一个不可变的集合。这样创建出来的集合不能添加、不能修改、不能删除。

> `.of` 静态方法 Java8 不能用

Java 8 中，一般采用这样的方法来创建一个不可变的集合：

```java
List<String> immutableList = Collections.unmodifiableList(Arrays.asList("A", "B", "C"));
Set<String> immutableSet = Collections.unmodifiableSet(new HashSet<>(Arrays.asList("A", "B", "C")));
Map<String, Integer> immutableMap = Collections.unmodifiableMap(new HashMap<String, Integer>() {{
    put("A", 1);
    put("B", 2);
    put("C", 3);
}});
```

> **`Arrays.asList()` 和 `List.of()`**
> 
> - `List.of()` 是一个静态工厂方法，返回的是一个不可变的列表，这个列表不是由数组支持的，而是一个实现了 `List` 接口的独立对象。使用 `List.of()` 返回的列表，不能添加、删除或替换其中的元素。
>     
> - `Arrays.asList()` 返回的是一个 **固定大小** 的列表，它是 `java.util.Arrays` 的一个静态方法，这个列表是由数组支持的，所以他的元素可以直接通过数组的索引来访问。`Arrays.asList()` 方法返回的列表，大小是固定的，如果尝试添加或删除元素，会抛出 `UnsupportedOperationException` 异常，因为底层数组的大小是固定的。（但是 IDEA 调 `.add` 方法不会有任何提示，因此要注意）：
>     
>     ```java
>     List<String> list = Arrays.asList("a", "b", "c");
>     list.set(1, "z");  // 这是允许的，替换了索引为1的元素
>     System.out.println(list);
>     list.add("d");  // 这将抛出 UnsupportedOperationException 
>     list.remove("a");  // // 这也将抛出 UnsupportedOperationException 
>     ```
>     
>     ```none
>     [a, z, c]
>     Exception in thread "main" java.lang.UnsupportedOperationException
>     	at java.util.AbstractList.add(AbstractList.java:148)
>     	at java.util.AbstractList.add(AbstractList.java:108)
>     	at org.example.Main.main(Main.java:11)
>     ```
>     

# 创建Stream

## Stream.of()

最简单的方式是直接使用`Stream.of()`静态方法，传入可变参数。不过这种方式比较没卵用。

```java
import java.util.stream.Stream;

public class Main {
    public static void main(String[] args) {
        Stream<String> stream = Stream.of("A", "B", "C", "D");
        ...
    }
}
```

## 基于数组或Collection

直接基于数组或者`Collection`来创建`Stream`，这样`Stream`的元素就直接持有数组或者`Collection`的元素了。

```java
import java.util.*;
import java.util.stream.*;

public class Main {
    public static void main(String[] args) {
        Stream<String> stream1 = Array.stream(new String[] {"H", "A", "L", "O"});
        Stream<String> stream2 = List.of("D", "O", "G").stream();
        ...
    }
}
```

- 将数组变为`Stream`：使用`Arrays.stream()`方法。
- 将`Collection`变为`Stream`：直接调用`.stream()`方法即可。

## 关于Primitives

Java泛型不支持Primitives，所以`Stream<int>`是非法的。但是用`Stream<Integer>`会频繁装箱拆箱，降低了效率。故Java直接提供了`IntStream`、`LongStream`、`DoubleStream`来直接使用primitive的`Stream`，其使用方法没有太大差别。

```java
// 将int[] 数组变为IntStream
IntStream is = Arrays.stream(new int[]{1, 2, 3});

// 将Stream<String>转换为LongStream
LongStream ls = List.of("1", "2", "3").stream().mapToLong(Long::parseLong);
```

# Stream的中间方法

Stream常见的中间方法见如下表格

| 名称                                               | 说明                                   |
| -------------------------------------------------- | -------------------------------------- |
| `Stream<T> filter(Predicate<? super T> predicate)` | 过滤                                   |
| `Stream<T> limit(long maxSize)`                    | 获取前几个元素                         |
| `Stream<T> skip(long n)`                           | 跳过前几个元素                         |
| `Stream<T> distinct()`                             | 元素去重，依赖`hashCode`和`equals`方法 |
| `static<T> Stream<T> concat(Stream a, Stream b)`   | 合并a和b两个流为一个流                 |
| `Stream<R> map(Function<T, R> mapper)`             | 转换流中的数据类型                     |



## 使用map

`map()`操作把`Stream`的每个元素都应用一遍目标函数并输出结果。

```java
Stream<Integer> s = Stream.of(1, 2, 3, 4, 5);
Stream<Integer> s2 = s.map(n -> n * n);     // 1, 4, 9, 16, 25
```

```java
List.of("  Apple ", " pear ", "ORAMGE", "  BaNaNa  ")
    .stream()
    .map(String::trim)  // 去空格
    .map(String::toLowerCase)  // 变小写
    .forEach(System.out::println);  // 打印
```

```java
List<String> list = new ArrayList<>();
Collections.addAll(list, "张无忌-15", "周芷若-14", "赵敏-13", "张强-20", "张三丰-100", "张翠山-40");
// 需求： 只获取里面的年龄并打印
// 
list.stream().map(new Function<String, Integer>() {
    @Override
    public Integer apply(String s) {
        String[] arr = s.split("-");
        String ageString = arr[1];
        int age = Integer.parseInt(ageString);
        return age;
    }
}).forEach(System.out::println);

// 使用 Lambda 表达式：
list.stream().map(s -> Integer.parseInt(s.split("-")[1])).forEach(System.out::println);
```

## 使用filter

`filter()`操作对`Stream`的每个元素进行测试，如果不满足条件就直接pass掉。

```java
IntStream.of(1, 2, 3, 4, 5, 6, 7, 8, 9)
    .filter(n -> n % 2 != 0)   // 选择奇数，过滤掉偶数
    .forEach(System.out::println);
```



# Stream的终结方法

## 使用reduce

`reduce()`操作把`Stream`的所有元素按照聚合函数聚合成一个结果。

```java
int sum = Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9)
    .reduce(0, (acc, n) -> acc + n);    // 初始化结果值为0，然后对每个元素依次调用函数
System.out.println(sum);   // 45
```

## 输出集合

`Stream`有两类操作：

- 转换操作：把一个Stream转换为另一个Stream，期间不做真正的计算操作。如`map()`和`filter()`
- 聚合操作：对Stream的每个元素进行计算，得到一个确定的结果，如`reduce()`。

### 输出为List

`List`的元素是确定的Java对象，因此把`Stream`变为`List`不是一个转换操作，而是一个聚合操作，强制`Stream`输出每个元素。

```java
public class Main {
    public static void main(String[] args) {
        Stream<String> stream = Stream.of("Apple", "", null, "Pear", "  ", "Orange");
        List<String> list = stream.filter(s -> s != null && !s.isBlank()).collect(Collectors.toList());
        System.out.println(list);
    }
}
```

把`Stream`的每个元素收集到`List`的方法是调用`collect()`并传入`Collectors.toList()`对象，它实际上是一个`Collector`实例，通过类似`reduce()`的操作，把每个元素添加到一个收集器中（实际上是`ArrayList`）。

类似，`collect(Collectors.toSet())`可以把`Stream`的每个元素收集到`Set`中。

### 输出为数组

只需要调用`toArray()`方法，并传入数组的构造方法即可。

```java
List<String> list = List.of("Apple", "Banana", "Orange");
String[] array = list.stream().toArray(String[]::new);
```

### 输出为Map

```java
public class Main {
    public static void main(String[] args) {
        Stream<String> stream = Stream.of("APPL:Apple", "MSFT:Microsoft");
        Map<String, String> map = stream
                .collect(Collectors.toMap(
                        // 把元素s映射为key:
                        s -> s.substring(0, s.indexOf(':')),
                        // 把元素s映射为value:
                        s -> s.substring(s.indexOf(':') + 1)));
        System.out.println(map);
    }
}

```

### 分组输出

分组输出使用`Collectors.groupingBy()`，需要提供

```java
public class Main {
    public static void main(String[] args) {
        List<String> list = List.of("Apple", "Banana", "Blackberry", "Coconut", "Avocado", "Cherry", "Apricots");
        Map<String, List<String>> groups = list.stream()
                .collect(Collectors.groupingBy(s -> s.substring(0, 1), Collectors.toList()));
        System.out.println(groups);
    }
}

```

分组输出使用`Collectors.groupingBy()`，它需要提供两个函数：一个是分组的key，这里使用`s -> s.substring(0, 1)`，表示只要首字母相同的`String`分到一组，第二个是分组的value，这里直接使用`Collectors.toList()`，表示输出为`List`，上述代码运行结果如下：

# 其他操作

## 排序

用`.sorted()`方法即可。如果需要自定义排序方法，在其中传入指定的`Comparator`即可。

```java
List<String> list = List.of("Orange", "apple", "Banana")
    .stream()
    .sorted(String::compareToIgnoreCase)
    .collect(Collectors.toList());
```

## 去重

直接用`.distint()`方法即可。

```java
List.of("A", "B", "A", "C", "B", "D")
    .stream()
    .distinct()
    .collect(Collectors.toList()); // [A, B, C, D]
```

## 截取

用`limit()`和`skip()`配合。`skip()`用于跳过当前`Stream`的前N个元素，`limit()`用于截取当前`Stream`最多前N个元素。

```java
List.of("A", "B", "C", "D", "E", "F")
    .stream()
    .skip(2) // 跳过A, B
    .limit(3) // 截取C, D, E
    .collect(Collectors.toList()); // [C, D, E]
```

## 合并

使用`.concat()`方法即可。

```java
Stream<String> s1 = List.of("A", "B", "C").stream();
Stream<String> s2 = List.of("D", "E").stream();

// 合并
Stream<String> s = Stream.concat(s1, s2);
System.out.println(s.collect(Collectors.toList()));   // [A, B, C, D, E]
```

## flatMap

`flatMap()`是把Stream的每个元素映射为`Stream`，然后合并成一个新的`Stream`。

```java
Stream<List<Integer>> s = Stream.of(
        Arrays.asList(1, 2, 3),
        Arrays.asList(4, 5, 6),
        Arrays.asList(7, 8, 9));
Stream<Integer> i = s.flatMap(list -> list.stream());
```