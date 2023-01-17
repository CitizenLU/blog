# 1 Python函数

python中的函数（也可以叫方法，强调面向对象时，一般叫方法，强调面向过程时，一般叫函数）本质上也是一个对象。



示例：

```python
def fun():
    print("这是一个函数")
   
# 调用fun方法
fun()

# fun是一个函数名，本质上是一个指向函数对象的变量
# 赋值
new_fun = fun

# new_fun和fun指向同一个内存地址
# is用于判断两个变量是否指向同一个内存地址
print(new_fun is fun)
# id方法用于查看该变量指向的内存地址
print(id(fun))
print(id(new_fun))
```



# 2 高阶函数

既然python的函数名是一个指向函数对象的变量，那么该变量就可以作为参数传给另外一个函数。

函数名作为一个参数返回。

```python
def foo():
    # inner_fun是定义在foo函数内的一个函数，可以作为返回参数
    def inner_fun():
        x = 10
        return x

    return inner_fun  # inner_fun是一个函数名，可以作为返回参数
```

调用时

```python
# 执行foo函数，会返回inner_fun函数，这里用变量a来接收这个函数
a = foo() # a的变量类型为function
# 执行inner_fun方法，并将返回值赋给变量b
b = a() # 调用a方法，也就是inner_fun方法
```



# 3 闭包

定义：将函数作为返回值返回。

形成闭包的条件：

1、函数嵌套

2、将内部函数作为返回值返回

3、内部函数必须使用到外部函数的变量

```python
def outer():
    x = 10

    def inner():
        nonlocal x
        x += 20
        return x

    return inner
```



# 4 装饰器

装饰器的使用场景：

1、参数校验

2、性能分析

3、日志跟踪

4、权限管理

5、数据库事务

## 4.1 使用装饰器计算某个方法的执行时间

例子：

需要计算调用study方法的时间

```python
# 记录一个方法执行的时间
def study():
    start_time = time.time()  # 当前时间
    print("study Chinese")
    print("study Math")
    print("study English")
    time.sleep(1)
    end_time = time.time()  # 结束时间
    print("All time is ", (end_time - start_time), "seconds")


study()
```

问题：

1、如果有多个任务，都需要记录时间，是否需要在每个任务中都写计算时间的方法？

2、控制代码和业务代码耦合在一起，没有实现有效的分离

控制代码：计算运行的时间的代码，出现了重复也就是冗余
业务代码：具体学习的科目

使用高阶函数进行改进：

```python
# func是函数变量
# 调用record_time方法时，会记录起始时间，然后调用func方法，调用完成后会记录结束时间，差值即是func方法的执行时间，可以记录到日志中
def record_time(func):
    start_time = time.time()
    func()
    end_time = time.time()
    print("All time is ", (end_time - start_time), "seconds")
    
def study():
    print("study Chinese")
    print("study Math")
    print("study English")
    time.sleep(1)
    
record_time(study)
```

问题：

改变了业务代码的调用方式。即：需要做的是study，原本的调用方式是`study()`，而此处为了记录时间，将调用方式改变成了`record_time(study)`。

使用装饰器进行改进：

```python
# record_time方法不变
def record_time(func):
    start_time = time.time()
    func()
    end_time = time.time()
    print("All time is ", (end_time - start_time), "seconds")
    
# 在study方法上，加一个语法糖。语法糖也叫装饰器，record_time方法装饰了study方法
@record_time
def study():
    print("study Chinese")
    print("study Math")
    print("study English")
    time.sleep(1)

study
```

装饰器的要求：

1、不改变被装饰的函数

2、不改变被装饰的函数的调用方式

## 4.2 使用装饰器进行参数校验

例子：

某个方法的其中一个入参是手机号，需要对此做校验，校验规则如下：

1、手机号不为空

2、手机号必须是纯数字

3、手机号是11位

```python

```













