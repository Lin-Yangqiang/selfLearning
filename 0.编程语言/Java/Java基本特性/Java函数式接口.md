# 函数式接口概览

函数式接口是一种特殊类型的接口，它只有一个抽象方法，因此可以实现函数作为一等公民（first-class functions），即函数可以像任何其他对象一样被传递和操作。

## 基本概念

1. **只有一个抽象方法的接口**：这是函数式接口的基本定义。除了这个抽象方法，函数式接口可以有**多个默认方法或静态方法**。
2. **@FunctionalInterface 注解**：这是一个元注解，用于明确标识一个接口是函数式接口。虽然这个注解不是必需的，但它有两个用途：一是提供编译时检查，确保接口只有一个抽象方法；二是作为一种文档说明，表明该接口是设计为函数式接口的。
3. **Lambda 表达式**：Lambda 表达式是一种简洁的方式来实现函数式接口的匿名实现。Lambda 允许你以更简洁的形式表达单方法接口的实现。
4. **方法引用**：方法引用是一种使用现有方法作为Lambda表达式的语法糖。它允许你引用类的方法或者构造函数，而不是显式地实现整个函数式接口。
5. **构造函数引用**：与方法引用类似，构造函数引用允许你引用一个类的构造函数。
6. **泛型函数式接口**：Java 8 提供了几个泛型函数式接口，如 `Function<T,R>`、`Predicate<T>`、`Consumer<T>` 和 `Supplier<T>` 等，它们分别用于代表不同的函数操作，比如函数映射、条件判断、消费操作和供应操作。

## Java 8 中的函数式接口

以下是 Java 8 提供的一些核心函数式接口：

| 函数式接口      | 说明                                             |
| --------------- | ------------------------------------------------ |
| `Consumer<T>`   | 接受一个参数并执行某种操作，无返回值。           |
| `Predicate<T>`  | 接受一个类型为 T 的参数，并返回一个**布尔值**。  |
| `Function<T,R>` | 接受一个类型为 T 的参数，并返回类型为 R 的结果。 |
| `Supplier<T>`   | 不接受如何参数，返回类型为 T 的结果。            |

除此之外，还有以下一些函数式接口：

- `BinaryOperator<T>`：接受两个类型为 T 的参数，并返回类型为 T 的结果。
- `UnaryOperator<T>`：接受一个类型为 T 的参数，并返回类型为 T 的结果

```java
import java.util.function.BiConsumer;  
import java.util.function.BiFunction;  
import java.util.function.BinaryOperator;  
import java.util.function.Consumer;  
import java.util.function.Function;  
import java.util.function.Predicate;  
import java.util.function.Supplier;  
import java.util.function.ToDoubleFunction;  
import java.util.function.ToIntFunction;  
import java.util.function.ToLongFunction;
```

# Consumer

## 基本信息

`Consumer` 接口是 `java.util.function` 包的一部分。`Consumer` 接口的泛型声明如下：

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}
```

`Consumer` 接口的目的是执行一个操作，它接受一个输入参数，并返回 `void`。也就是说，它接受了一个参数，并产生了一些结果，但是它是没有返回值的。`accept` 方法是 `Consumer` 接口的**唯一抽象方法**，它接受一个参数并执行某些操作。

## Consumer 的使用

`Consumer` 接口通常用在需要对集合中的每个元素执行操作的场景中，比如在 `forEach` 方法中。

### 示例1：使用 `forEach` 方法

![image](https://raw.githubusercontent.com/Lin-Yangqiang/image-host/master/2958440-20241102012057883-1592409731.png)

```java
package org.example;

import java.util.Arrays;
import java.util.List;
import java.util.function.Consumer;

public class Main {
    public static void main(String[] args) {
        List<String> list = Arrays.asList("Apple", "Banana", "Cherry");
        list.forEach(new Consumer<String>() {
            @Override
            public void accept(String s) {
                System.out.println(s);
            }
        });
    }
}
```

在这个例子中，我们使用 `forEach` 方法遍历列表，并为每个元素执行 `accept` 方法。这里我们创建了一个匿名内部类来实现 `Consumer` 接口。

### 示例2：使用 Lambda表达式改写

在函数式编程中，可以使用 lambda 表达式简化代码，因此上述代码还能够写为：

```java
package org.example;

import java.util.Arrays;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<String> list = Arrays.asList("Apple", "Banana", "Cherry");
        list.forEach(s -> System.out.println(s));
    }
}
```

这里我们直接提供了一个 lambda 表达式作为参数，它实现了 `accept` 方法。

### 示例3：使用 forEach 方法和方法引用

如果我们的方法是接收一个参数，并执行操作，我们就可以使用方法引用：

```java
package org.example;

import java.util.Arrays;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<String> list = Arrays.asList("Apple", "Banana", "Cherry");
        list.forEach(System.out::println);
    }
}
```

或者我直接自定义一个方法来更好地体现「接受一个参数，并执行操作」的含义：

```java
package org.example;

import java.util.Arrays;
import java.util.List;

public class Main {
    public static void print(String s) {
        System.out.println(s);
    }

    public static void main(String[] args) {
        List<String> list = Arrays.asList("Apple", "Banana", "Cherry");
        list.forEach(Main::print);
    }
}
```

在这个例子中，我们使用方法引用 `Main::print` 来引用 `print` 方法，它接受一个 `String` 参数并打印它。

## andThen 方法

`andThen` 是 `Consumer` 接口提供的一个默认方法，它允许你将两个 `Consumer` 组合在一起，以便在同一个元素上顺序执行两个操作。这个方法的声明如下：

```java
default Consumer<T> andThen(Consumer<? super T> after) {
    Objects.requireNonNull(after);
    return (T t) -> { accept(t); after.accept(t); };
}
```

### andThen 方法的工作原理

当你调用 `andThen` 方法时，你需要提供一个 `Consumer` 对象作为参数。`andThen` 方法会返回一个新的 `Consumer` 对象，这个新对象在它的 `accept` 方法中会先调用当前 `Consumer` 的 `accept` 方法，然后调用传入的 `Consumer` 对象的 `accept` 方法。

### andThen 方法的使用示例

假设我们有两个 `Consumer` 对象，一个用于打印字符串，另一个用于将字符串转换为大写：

```java
Consumer<String> printConsumer = System.out::println;
Consumer<String> toUpperCaseConsumer = s -> System.out.println(s.toUpperCase());
```

我们可以使用 `andThen` 方法将这两个操作组合在一起，使得每个字符串首先被打印，然后它的大写形式也被打印：

```java
Consumer<String> combinedConsumer = printConsumer.andThen(toUpperCaseConsumer);

List<String> list = Arrays.asList("apple", "banana", "cherry");
list.forEach(combinedConsumer);
```

输出结果将是：

```text
apple
APPLE
banana
BANANA
cherry
CHERRY
```

在这个例子中，`andThen` 方法创建了一个新的 `Consumer`，它将两个操作串联起来。对于列表中的每个元素，都会先执行 `printConsumer` 的操作（打印字符串），然后执行 `toUpperCaseConsumer` 的操作（打印字符串的大写形式）。

### 注意事项

- `andThen` 方法返回一个新的 `Consumer` 对象，因此它不会修改原始的 `Consumer` 对象。
- 如果任何一个 `Consumer` 抛出异常，后续的 `Consumer` 将不会被执行。
- `andThen` 方法可以链式调用，即你可以将多个 `Consumer` 组合在一起，例如 `consumer1.andThen(consumer2).andThen(consumer3)`。

# Function

## 基本信息

`Function` 接口是 `java.util.funtion` 包的一部分，`Function` 表示一个函数，该函数接受一个参数并返回结果，因此 `Function` 接口接受两个泛型：

- `T` - 表示输入参数的类型
- `R` - 表示函数返回值的类型

`Function` 函数式接口的定义如下：

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```

`Function` 接口有一个抽象方法 `apply(T t)`，它接受`T`类型的参数并返回`R`类型的结果。

## Function 的使用

### 示例1：自定义 `findLength`

```java
import java.util.function.Function;

public class FunctionExample {
    public int findLength() {
        Function<String, Integer> findLength = (text) -> text.length();
        int strLength = findLength.apply("baidu");
        return strLength;
    }
    
    public static void main(String[] args) {
        FunctionExample functionExample = new FunctionExample();
        System.out.println("The length of the string" + functionExample.findLength());
    }
}
```

我们定义了一个名为 `findLength` 的函数，它以 `String` 作为输入 （`T`） 并返回一个表示字符串长度 （`R`） 的 `Integer`。

### 示例2： 使用 `map` 方法

![image-20241102014214567](https://raw.githubusercontent.com/Lin-Yangqiang/image-host/master/image-20241102014214567.png)

```java
class Main {
    public static void main(String[] args) {
        List<String> words = Arrays.asList("apple", "banana", "cherry");
        Function<String, String> toUpperCase = String::toUpperCase;
        List<String> upperCaseWords = words.stream().map(toUpperCase).collect(Collectors.toList());
        System.out.println(upperCaseWords);
    }
}
```

输出：

```
[APPLE, BANANA, CHERRY]
```

## andThen 方法

`andThen()` 方法返回一个组合函数（Composed Function），其中参数化函数将在第一个函数之后执行。如果任一函数的计算引发错误，则会将其中继给组合函数的调用方。

> 「中继给组合函数的调用方」：如果在执行第一个函数或参数化函数的过程中出现了错误（例如，函数内部抛出了异常），那么这个错误不会被组合函数内部处理，而是会**直接传递给调用组合函数的外部代码**。这里的"中继"（relayed）意味着错误不会被组合函数捕获或处理，而是直接传递给调用者，让调用者来决定如何处理这个错误。

语法：

```java
default <V> Function<T, V> 
andThen(Function<? super R, ? extends V> after)
```

其中 `V` 是 `after` 函数和组合函数的输出类型。

- 示例1：

  ```java
  class Main {
      public static void main(String[] args) {
          Function<Integer, Double> half = a -> a / 2.0;
          half = half.andThen(a -> 3 * a);
          System.out.println(half.apply(10));
      }
  }
  ```

  输出：

  ```
  15.0
  ```

## compose 方法

`composed()` 方法返回一个组合函数，其中参数化函数将首先执行，然后是第一个函数。如果任一函数的计算引发错误，则会将其中继给组合函数的调用方。

语法：

```java
default <V> Function<V, R> 
compose(Function<? super V, ? extends T> before)
```

其中 `V` 是 `before` 函数和组合函数的输入类型

# Predicate

`Predicate` 是 Java 8 中引入的一个函数式接口，位于 `java.util.function` 包中。它代表了一个接受一个参数并返回布尔值（true 或 false）的布尔函数。这个接口主要用于条件检查或条件过滤，它提供了一种简洁的方式来表达条件逻辑。

## 基本介绍

`Predicate<T>` 接口有一个抽象方法 `boolean test(T t)`，它接受一个类型为 `T` 的参数，并返回一个布尔值。这个接口的设计目的是将条件逻辑作为参数传递，使得我们可以将条件逻辑作为数据来处理，这是函数式编程的一个核心特性。

`Predicate` 接口的语法非常简单：

```java
public interface Predicate<T> {
    boolean test(T t);
}
```

## 示例

以下是一些使用 `Predicate` 的示例：

### 示例 1：检查数字是否为正数

```java
import java.util.function.Predicate;

public class PredicateExample {
    public static void main(String[] args) {
        Predicate<Integer> isPositive = x -> x > 0;

        int number = 5;
        boolean result = isPositive.test(number);
        System.out.println("Is " + number + " positive? " + result);
    }
}
```

在这个例子中，我们创建了一个 `Predicate<Integer>` 实例 `isPositive`，它检查一个整数是否大于 0。然后我们使用 `test` 方法来检查数字 5 是否为正数。

### 示例 2：过滤列表中的偶数

```java
import java.util.Arrays;
import java.util.List;
import java.util.function.Predicate;
import java.util.stream.Collectors;

public class PredicateExample {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);

        Predicate<Integer> isEven = x -> x % 2 == 0;

        List<Integer> evenNumbers = numbers.stream()
                .filter(isEven)
                .collect(Collectors.toList());

        System.out.println("Even numbers: " + evenNumbers);
    }
}
```

在这个例子中，我们使用 `Predicate<Integer>` 实例 `isEven` 来检查一个整数是否为偶数。然后我们使用 Java 8 的 Stream API 和 `filter` 方法来过滤出列表中的所有偶数。

### 示例 3：使用 `Predicate` 接口的静态方法

`Predicate` 接口提供了一些静态方法，如 `isEqual`、`isNotNull`、`isNull` 等，这些方法可以用来创建常见的条件检查。

```java
import java.util.function.Predicate;

public class PredicateExample {
    public static void main(String[] args) {
        String str = "Hello";

        Predicate<String> isNotEmpty = Predicate.not(String::isEmpty);
        Predicate<String> endsWithL = Predicate.isEqual("l");

        boolean result1 = isNotEmpty.test(str);
        boolean result2 = endsWithL.test(str);

        System.out.println("Is string not empty? " + result1);
        System.out.println("Does string end with 'l'? " + result2);
    }
}
```

在这个例子中，我们使用 `Predicate` 接口的 `not` 和 `isEqual` 静态方法来创建条件检查。`isNotEmpty` 检查字符串是否不为空，`endsWithL` 检查字符串是否以 "l" 结尾。

# Supplier

`Supplier` 是 Java 8 中引入的一个函数式接口，位于 `java.util.function` 包中。它代表了一个不接受任何参数并返回一个结果的供应器。这个接口主要用于提供数据，特别是在需要延迟计算或者按需生成数据时非常有用。

### 基本介绍

`Supplier<T>` 接口有一个抽象方法 `T get()`，它不接受任何参数，并返回一个类型为 `T` 的结果。这个接口的设计目的是将数据生成逻辑作为参数传递，使得我们可以将数据生成逻辑作为数据来处理，这是函数式编程的一个核心特性。

### 语法

`Supplier` 接口的语法非常简单：

```java
public interface Supplier<T> {
    T get();
}
```

### 示例

以下是一些使用 `Supplier` 的示例：

#### 示例 1：生成当前时间

```java
import java.util.function.Supplier;
import java.time.LocalDateTime;

public class SupplierExample {
    public static void main(String[] args) {
        Supplier<LocalDateTime> currentDateTimeSupplier = LocalDateTime::now;

        LocalDateTime now = currentDateTimeSupplier.get();
        System.out.println("Current DateTime: " + now);
    }
}
```

在这个例子中，我们创建了一个 `Supplier<LocalDateTime>` 实例 `currentDateTimeSupplier`，它使用 `LocalDateTime::now` 方法引用来获取当前的日期和时间。然后我们使用 `get` 方法来获取当前的日期和时间。

#### 示例 2：生成随机数

```java
import java.util.Random;
import java.util.function.Supplier;

public class SupplierExample {
    public static void main(String[] args) {
        Random random = new Random();
        Supplier<Integer> randomIntSupplier = () -> random.nextInt(100);

        int randomInt = randomIntSupplier.get();
        System.out.println("Random Int: " + randomInt);
    }
}
```

在这个例子中，我们创建了一个 `Supplier<Integer>` 实例 `randomIntSupplier`，它使用一个 `Random` 对象来生成一个介于 0 和 99 之间的随机整数。然后我们使用 `get` 方法来获取一个随机整数。

#### 示例 3：使用 `Supplier` 接口的实例来初始化集合

```java
import java.util.Arrays;
import java.util.List;
import java.util.function.Supplier;

public class SupplierExample {
    public static void main(String[] args) {
        Supplier<List<String>> listSupplier = () -> Arrays.asList("Apple", "Banana", "Cherry");

        List<String> fruits = listSupplier.get();
        System.out.println("Fruits: " + fruits);
    }
}
```

在这个例子中，我们创建了一个 `Supplier<List<String>>` 实例 `listSupplier`，它使用一个 lambda 表达式来初始化一个包含三种水果的列表。然后我们使用 `get` 方法来获取这个列表。

这些示例展示了 `Supplier` 接口的灵活性和强大功能，它使得数据生成和提供变得更加简洁和模块化。通过将数据生成逻辑封装在 `Supplier` 实例中，我们可以轻松地在不同的上下文中重用和组合这些逻辑。



# References

1. [A Deep Dive into Supplier, Consumer, and Function in Java 8. | by Java Fusion | Medium](https://medium.com/@JavaFusion/a-deep-dive-into-supplier-consumer-and-function-in-java-8-cc2025cea6c1)
2. [Java 8 | Consumer Interface in Java with Examples - GeeksforGeeks](https://www.geeksforgeeks.org/java-8-consumer-interface-in-java-with-examples/)
3. [Java 8 Predicate with Examples - GeeksforGeeks](https://www.geeksforgeeks.org/java-8-predicate-with-examples/?ref=next_article)
4. [Function Interface in Java with Examples - GeeksforGeeks](https://www.geeksforgeeks.org/function-interface-in-java-with-examples/?ref=next_article)
5. [Supplier Interface in Java with Examples - GeeksforGeeks](https://www.geeksforgeeks.org/supplier-interface-in-java-with-examples/?ref=next_article)
6. [Method References in Java with examples - GeeksforGeeks](https://www.geeksforgeeks.org/method-references-in-java-with-examples/?ref=next_article)