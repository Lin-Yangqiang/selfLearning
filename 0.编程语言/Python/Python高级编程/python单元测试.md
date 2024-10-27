# 使用 doctest

doctest 应该是 Python 中最轻量级的单元测试框架了，其特点为：

1. 无需编写测试代码，只需在**文档**中添加测试用例
2. Python内置，无需安装
3. 学习成本低

在编写代码时，我们可以在文档中添加测试用例，然后使用 `doctest` 来执行测试用例，从而验证代码的正确性。

以下用一个求阶乘的函数来演示 doctest 的使用：

```python
"""
This is the "example" module.

The example module supplies one function, factorial().  For example,

>>> factorial(5)
120
"""

def factorial(n):
    """Return the factorial of n, an exact integer >= 0.

    >>> [factorial(n) for n in range(6)]
    [1, 1, 2, 6, 24, 120]
    >>> factorial(30)
    265252859812191058636308480000000
    >>> factorial(-1)
    Traceback (most recent call last):
        ...
    ValueError: n must be >= 0

    Factorials of floats are OK, but the float must be an exact integer:
    >>> factorial(30.1)
    Traceback (most recent call last):
        ...
    ValueError: n must be exact integer
    >>> factorial(30.0)
    265252859812191058636308480000000

    It must also not be ridiculously large:
    >>> factorial(1e100)
    Traceback (most recent call last):
        ...
    OverflowError: n too large
    """

    import math
    if not n >= 0:
        raise ValueError("n must be >= 0")
    if math.floor(n) != n:
        raise ValueError("n must be exact integer")
    if n+1 == n:  # catch a value like 1e300
        raise OverflowError("n too large")
    result = 1
    factor = 2
    while factor <= n:
        result *= factor
        factor += 1
    return result

if __name__ == "__main__":
    import doctest
    doctest.testmod()
```

直接运行上述代码，会发现什么输出都没有。这说明通过了所有的测试用例。如果你仍然想看看详细的信息，可以指定 `testmod()` 中的 `verbose=True` ：

```python
if __name__ == "__main__":
    import doctest
    doctest.testmod(verbose=True)
```

加上后，输出如下：

```
Trying:
    factorial(5)
Expecting:
    120
ok
Trying:
    [factorial(n) for n in range(6)]
Expecting:
    [1, 1, 2, 6, 24, 120]
ok
Trying:
    factorial(30)
Expecting:
    265252859812191058636308480000000
ok
Trying:
    factorial(-1)
Expecting:
    Traceback (most recent call last):
        ...
    ValueError: n must be >= 0
ok
Trying:
    factorial(30.1)
Expecting:
    Traceback (most recent call last):
        ...
    ValueError: n must be exact integer
ok
Trying:
    factorial(30.0)
Expecting:
    265252859812191058636308480000000
ok
Trying:
    factorial(1e100)
Expecting:
    Traceback (most recent call last):
        ...
    OverflowError: n too large
ok
2 items passed all tests:
   1 tests in __main__
   6 tests in __main__.factorial
7 tests in 2 items.
7 passed and 0 failed.
Test passed.

Process finished with exit code 0

```

现在，我们故意改错其中的一个测试用例，然后尝试再次执行代码：

```python
def factorial(n):  
    """Return the factorial of n, an exact integer >= 0.  
    >>> factorial(30)  
    1    """  
    import math  
    if not n >= 0:  
        raise ValueError("n must be >= 0")  
    if math.floor(n) != n:  
        raise ValueError("n must be exact integer")  
    if n+1 == n:  # catch a value like 1e300  
        raise OverflowError("n too large")  
    result = 1  
    factor = 2  
    while factor <= n:  
        result *= factor  
        factor += 1  
    return result  
  
  
if __name__ == "__main__":  
    import doctest  
    doctest.testmod()
```

输出为：

```
**********************************************************************
File "D:\workingDirectory\pycharmProject\pydemo\main.py", line 12, in __main__.factorial
Failed example:
    factorial(30)
Expected:
    1
Got:
    265252859812191058636308480000000
**********************************************************************
1 items had failures:
   1 of   1 in __main__.factorial
***Test Failed*** 1 failures.

Process finished with exit code 0

```

>[!IMPORTANT]
> 由于 `doctest` 是 python 的内置模块，因此，可以使用 `python -m doctest` 来执行测试用例。
> 
	 ```
	 python -m doctest -v main.py  # -v 参数表示打印测试用例的执行结果
	 ```



# 使用 pytest

TODO