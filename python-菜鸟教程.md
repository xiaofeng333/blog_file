---
title: python-菜鸟教程
tags: python
date: 2019-05-19 18:10:55
---

##### 语法

* 注释

  以#开头, 或 ''' 和 """。

* 行与缩进

  使用缩进来表示代码块, 同一个代码块的语句必须包含相同的缩进空格数。

  缩进数的空格数不一致，会导致运行错误。

* 多行语句

  语句很长，可以使用反斜杠\来实现多行语句。

  ```python
  total = item_one + \
          item_two + \
          item_three
  ```

* 数字类型

  整数(int)、布尔型(bool)、浮点数(float)和复数(complex)。

  int表示为长整型。

  True 和 False 定义成关键字了，但它们的值还是 1 和 0，它们可以和数字相加。
  数值的除法包含两个运算符：/ 返回一个浮点数，// 返回一个整数。
  在混合计算时，Python会把整型转换成为浮点数。

* 字符串

  字符串有两种索引方式，从左往右以 0 开始，从右往左以 -1 开始。

  反斜杠可以用来转义，使用r可以让反斜杠不发生转义。

  字符串的截取的语法格式如下：变量[头下标:尾下标:步长]

* 空行

  函数之间或类的方法之间用空行分隔，表示一段新的代码的开始。类和函数入口之间也用一行空行分隔，以突出函数入口的开始。

  空行与代码缩进不同，空行并不是Python语法的一部分。

* Print 输出
  print 默认输出是换行的，如果要实现不换行需要在变量末尾加上 end=“”

* import 与 from...import

  import 或者 from...import 来导入相应的模块。

  import导入整个模块。

  from...import从某个模块中导入多个函数。


##### Python3 命令行参数

Python 提供了 getopt 模块来获取命令行参数。

##### Python3 基本数据类型

Python 中的变量不需要声明。每个变量在使用前都必须赋值，变量赋值以后该变量才会被创建。

###### 多个变量赋值

Python允许同时为多个变量赋值, 也可以为多个对象指定多个变量。

```python
a = b = c = 1
a, b, c = 1, 2, 'runoob'
```

###### 标准数据类型

Python3 的六个标准数据类型中:

* 不可变数据

  Number、String、Tuple

* 可变数据

  List、Dictionary、Set

内置的 type() 函数可以用来查询变量所指的对象类型, 也可用isinstance来判断。

type()不会认为子类是一种父类类型, isinstance()会认为子类是一种父类类型。

一个变量可以通过赋值指向不同类型的对象。

```python
a = 1
type(a)
isinstance(a, int)
```

可以使用del语句删除一些对象引用。

```python
del a, b, c, d
```

* List

  列表中元素的类型可以不相同，它支持数字，字符串甚至可以包含列表（所谓嵌套）。

  加号 + 是列表连接运算符，星号 * 是重复操作

* Tuple

  元组的元素不能修改，但其中可变数据可以修改。

* set

  元素不重复

  创建一个空集合必须用 set()

* Dictionary

  键(key)必须使用不可变类型。

  在同一个字典中，键(key)必须是唯一的。

​       创建空字典使用 { }

##### python解释器

Python 解释器不止一种哦，有 CPython、IPython、Jython、PyPy 等。

Jython 是专为 Java 平台设计的 Python 解释器，它把 Python 代码编译成 Java 字节码执行。

##### TIP

1.在python中操作集合或字典中不存在的元素会报错。

2.右边的表达式会在赋值变动之前执行。右边表达式的执行顺序是从左往右的。

```python
def calculate():
    a = 1
    b = 2
    a, b = b, a+b
    print(a) # 2
    print(b) # 3
```

3.关键字end可以用于将结果输出到同一行，或者在输出的末尾添加不同的字符。

4.递归以空间换取可读性，当过深时，会造成溢出。

##### 条件控制

###### ifelse

if语句的关键字为：if – elif – else。

###### while

在while中可添加else的语句块

```python
count = 0 
while count < 5: 
	print (count, " 小于 5") 
	count = count + 1
else: 
	print (count, " 大于或等于 5")
```

###### for

for循环可以遍历任何序列的项目，如一个列表或者一个字符串。

###### range

内置range()函数, 会生成数列。

###### pass
pass是空语句，是为了保持程序结构的完整性。

##### 迭代器与生成器

1.把一个类作为一个迭代器使用需要在类中实现两个方法 __iter__() 与 __next__() 。

2.StopIteration 异常用于标识迭代的完成，防止出现无限循环的情况，在 __next__() 方法中我们可以设置在完成指定循环次数后触发 StopIteration 异常来结束迭代。

3.yield 的作用就是把一个函数变成一个 generator，带有 yield 的函数不再是一个普通函数，Python 解释器会将其视为一个 generator。

```python
# 斐波那契
def fibonacci(n):
    a, b, counter = 0, 1, 0
    while True:
        if (counter > n): 
            return
        yield a
        a, b = b, a + b
        counter += 1
```

