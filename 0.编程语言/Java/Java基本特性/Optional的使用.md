# 1 基本介绍

在 Java 中最烦人的就是 `NullPointerException` 空指针异常。为了避免这个问题，Java 8 引入了一个新的类 `Optional` （`java.util.Optional`）。使用 `Optional` 类，我们可以减少空值检测的验证代码，用一种更加简洁的方式来对一些变量进行校验，避免空指针异常。

通过使用 `Optional`，我们可以指定要返回的替代值或者要运行的替代代码，这使得代码更具可读性。

## 1.1 引例

在Java中经常遇到空指针异常，其中最经常出现该异常的情况，就是对一个**空的** Array 或 Collection 用下标（索引）取其中的元素。比如如下代码：

```java
public class OptionalDemo {
    public static void main(String[] args) {
        String[] words = new String[10];
        String word = words[5].toLowerCase();
        System.out.println(word);
    }
}
```

输出：

```
Exception in thread "main" java.lang.NullPointerException
```

---

为了避免空指针异常的发生，我们可以使用 `Optional` 类，将上述代码修改如下：

```java
public class OptionalDemo {
    public static void main(String[] args) {
        String[] words = new String[10];
        Optional<String> checkNull = Optional.ofNullable(words[5]);
        if (checkNull.isPresent()) {
            System.out.println(words[5].toLowerCase());
        } else {
            System.out.println("word is null");
        }
    }
}
```

输出：

```
word is null
```

## 1.2 Optional类

`Optional` 是一个 **容器类**，它可以保存某个类型的值，并且提供一些类的方法来避免空指针。`Optional` 主要可以用来代替 `if ... else` 来实现判断 `null`，使代码更加整洁。

### 1.2.1 创建和使用 Optional 对象

1. 使用 `Optional.empty()` 方法，返回一个空的 `Optional` 实例

   ```java
   Optional<String> empty = Optional.empty();
   ```

2. 使用 `Optional.of(T value)` 方法：如果 `value` 不为 `null`，则返回一个新的 `Optional` 实例，否则抛出 `NullPointerException`。

3. 使用 `Optional.ofNullable(T value)` 方法：如果 `value` 不为 `null`，则返回一个新的 `Optional` 实例；如果 `value` 为 `null`，则返回一个空的 `Optional` 实例。

   ```java
   String possibleNull = getSomeString();
   Optional<String> optionalValue = Optional.ofNullable(possibleNull)
   ```

### 1.2.2 使用 Optional 值

当有了 `Optional` 对象后，我们就可以执行各种操作来安全处理这个对象包含的 `value`：

1. 使用 `isPresent()` 方法检查是否存在值：

   ```java
   if (optionalValue.isPresent()) {
       // Do something with the value
   }
   ```

2. 如果 `Optional` 为 `null` 的话，可以使用 `orElse(defaultValue)` 来提供一个默认值：

   ```java
   String valueOrDefault = optionalValue.orElse("default value");
   ```

3. 当 `Optional` 为 `null` 时，我们如果想引发特定异常的情况，可以用 `orElseThrow(exceptionSupplier)`：

   ```java
   String valueOrException = optionalValue.orElseThrow(NoSuchElementException::new);
   ```

### 1.2.3 通过 Lambda 表达式利用 Optional

1. 仅当值存在时，才能使用 `ifPresent(consumer)` 方法执行代码块：

   ```java
   optionalValue.ifPresent(value -> System.out.println("Value is present: " + value));
   ```

2. 可以使用 `map(function)` 方法转换 `Optional` 中的值。如果转换函数后返回 not-null 结果，就可以获得一个包含转换后的值的 `Optional` 实例；否则就会得到一个空的 `Optional`：

   ```java
   Optional<Integer> stringLength = optionalValue.map(String::length);
   ```

### 1.2.4 链式调用 Optional 方法

例：

```java
Optional<String> filteredAndTransformed = optionalValue.filter(s -> s.startsWith("J")).map(String::toUpperCase);
```