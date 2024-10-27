# Lambda表达式

## 1 概述

Lambda 是 JDK8 中的一个语法糖。它可以对某些匿名内部类的写法进行简化。它是函数式编程思想的一个重要体现。让我们不用关注是什么对象。而是更关注我们对数据进行了什么操作。

## 2 核心原则

可推导、可省略

## 3 基本格式

```
(参数列表) -> {代码}
```

### 例

创建线程并启动的时候，可以使用匿名内部类的写法：

```java
new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("abc");
    }
}).start();
```

可以使用 Lambda 的格式对其进行修改，修改后如下：

```java
new Thread(() -> {
    System.out.println("abc");
}).start()
```

