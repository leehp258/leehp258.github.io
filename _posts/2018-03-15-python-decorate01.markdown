---
layout:     post
title:      "Python Decorator01"
subtitle:   "关于装饰器"
date:       2018-03-15
author:     "leehp258"
header-img: "img/home-bg.jpg"
tags:
    - Python
    - Decorator
    - 装饰器
---

# Decorator 第一讲

> 学习装饰器，有助于理解命名空间，作用域，闭包，类的可调用本质，导入时和运行时，以及切面编程

- 命名空间和变量作用域(Namespace and Scope)
- 嵌套函数
- 闭包 【携带自由变量的嵌套函数，被父层函数返回给外部调用者，从而使得内部函数成为自带状态的特殊函数】
- 装饰器 【特殊的闭包，接收一个函数，返回一个函数，在外部用返回的函数替换掉了原函数】
    - 类装饰器 【接收一个 callable 对象，返回一个 callable 对象】
- 面向切面编程

## 命名空间和作用域
名称到对象的映射。命名空间是一个字典的实现，键为变量名，值是变量对应的值。各个命名空间是独立没有关系的，一个命名空间中不能有重名，但是不同的命名空间可以重名而没有任何影响。
- Namespace 是命名到对象的一个映射（a mapping from name to objects），python 中这个映射可能是用 dict 来实现的。
    - 函数的 `local` namespace
    - 嵌套闭包(Enclosing)的 `nonlocal` namespace
    - module 的 `global` namespace
    - 内置的 `builtins` namespace: 包含内置function和exceptions
- Scope 定义如何搜索Namespace的层级关系(hierarchy level)，以确定"name-to-object"的映射。
- 命名作用域按照"LEGB-rule"规则顺序查找: local > enclosing > global > builtins
- 这里的 enclosing 域是闭包才有的，不一定存在。
- 按顺序命名找不到就报 NameError

> 这里的 nonlocal, global, builtins 都是有效关键字
```python
# 列出所有的内置对象
import builtins
dir(builtins) 
```

> 与命名空间作用域相关的内置函数: locals(), globals(), vars()

```python
# 全局变量，函数内部可以引用，但不能修改
x = 1
def printx(): print(x)
def changex(): x += 1
printx()
# changex()  # raise UnboundLocalError

# 如果直接赋值，其实是一个同名的局部变量
def setx(): 
    x = 2
    print(x)
setx()
print(x) # 全局的 x 并没有被修改
```
```
1
2
1
```
```python
# 在函数内通过 global 关键字说明引用的是全局变量
x = 1
def setx():
    global x # 标明用的是全局变量
    x += 1
setx()
print(x) # 全局的 x 改变
```
```
2
```
```python
def sety():
    global y
    y = 10 # 函数内初始化全局变量
sety()
y += 1
print(y)
```
```
11
```
### 命名空间与作用域的关系
命名空间定义了在某个作用域内变量名和绑定值之间的对应关系，命名空间是键值对的集合，变量名与值是一一对应关系。作用域定义了命名空间中的变量能够在多大范围内起作用。

命名空间在python解释器中是以字典的形式存在的，是以一种可以看得见摸得着的实体存在的。作用域是python解释器定义的一种规则，该规则确定了运行时变量查找的顺序，是一种形而上的虚的规定。

```python
# locals(), globals()都返回一个dict，locals只读，globals可以写
globs = globals()
globs['new_variable'] = 3
print(new_variable) # 打印3

# 而下面这样却会报错：NameError: name 'new_c' is not defined
def func(a:'参数a'=1):
    b = 2
    locs = locals()
    print(locs)
    locs['new_c'] = 3
#     print(new_c) # 报错 NameError: name 'new_c' is not defined
func()
```
```
3
{'b': 2, 'a': 1}
```
### import module & from module import x 区别
这两种导入方式的区别在于，`import module` 导入模块本身，包括命名空间。而`from module import`是将其它模块的函数或变量引到当前命名空间，如果有重名，`import module` 比较安全。

module也是对象，它有些内置属性：__name__, __file__(module的绝对路径), __dict__, __doc__, __package__, __path__

__name__ trick:取值决定于如何使用模块，如果是import导入，__name__的值就是模块的名字，如果是执行，__name__的值就是'__main__'

```python
# python for循环的作用域
for itm in range(5):pass
print(itm) # 打印 4, 循环外依然可以访问 itm

# 大部分语言（如C）, for-loop会引入一个新的作用域，python 不太一样，for后的变量在循环结束后，并不会被删除，循环结束后依然可以访问
```
```
4
```
### 类，列表推导(List Comprehension)，生成器表达式(Generator Expression)的命名空间和作用域
- 类有局部的命名空间，但并不构成一个作用域
- 列表推导 PY2没有引入新作用域，PY3引入了
- 生成器表达式都有引入新的作用域
- 函数都会引入一个新的作用域

```python
class A:
    a = 3
#     b = list(a+i for i in range(5)) # PY2, PY3 都报错NameError: name 'a' is not defined
#     b = [a+i for i in range(5)] # PY2正常，PY3 报错 NameError: name 'a' is not defined
    b = [A.a+i for i in range(5)]
    def fn(self):
        print(a, self.a, A.a)
ca = A()
print(ca.fn())
# 原因在于，类有自己的命名空间，b 的赋值表达式引入了新的作用域，该作用域并不能访问类的名空间。
```
```
<__main__.A object at 0x10ec85160> 3 3
None
```
#访问权限汇总表
<table>
    <tr><th>Can access class attributes</th><th>PY2</th><th>PY3</th></tr>
    <tr><td>list comp. iterable</td><td>Y</td><td>Y</td></tr>
    <tr><td>list comp. expression</td><td>Y</td><td>N</td></tr>
    <tr><td>gen expr. iterable</td><td>Y</td><td>Y</td></tr>
    <tr><td>gen expr. expression</td><td>N</td><td>N</td></tr>
    <tr><td>dict comp. iterable</td><td>Y</td><td>Y</td></tr>
    <tr><td>dict comp. expression</td><td>N</td><td>N</td></tr>
</table>

## 嵌套函数和作用域
- 函数内定义的函数，作用域是父函数内有效，外部不可引用(报 NameError)。
- 全局变量，局部变量，自由变量，非局部变量(nonlocal)
- global 不推荐使用，会降低程序的封装性，提高耦合，易造成混乱和错误

```python
# 嵌套函数，作用域在父函数内，外部不可引用
def outer():
    def inner():print("inner")
    inner()
outer()
#inner() # raise NameError
```
```
inner
```
```python
# 两个 x 的作用域的不一样的，各是各的
def outer():
    x = 0 # outer 的 x
    def inner():
        x = 1 # inner 的 x
        print("inner:", x)
    inner()
    print("outer", x)
outer()
```
```
inner: 1
outer 0
```
```python
# 嵌套函数如果想操作父函数的变量，怎么办？
# PY3 提供了 nonlocal 关键字
# inner 引用了 outer 的变量
def outer():
    x = 0
    def inner():
        nonlocal x
        x = 1
        print("inner:", x)
    inner()
    print("outer:", x) # 值被改变
outer()
```
```
inner: 1
outer: 1
```
## 闭包
看一个例子🌰：

```python
# 这样，嵌套函数引用了父函数的变量
# 一般的内部函数只在内部使用，但如果父函数把内部函数作为返回值返回给了调用者
# 那么此时的内部函数，就可以被父函数以外调用了，而父函数的变量 x 也因为被内部函数使用而生存下来
# 这个 x 就是一个自由变量了，因为它脱离了定义体，就是说即使 outer 函数执行完毕，x 仍然存在
def counter():
    i = 0
    def incr():
        nonlocal i
        i += 1
        return i
    return incr

c = counter() # counter 执行完毕将内部函数incr返回，并赋值给 c，c本质上是 incr
print(c) # c 本质上是 incr
# counter 定义的变量i，一直可用，并没有因为 counter 执行结束就被收回
c()
c()
```
```
<function counter.<locals>.incr at 0x10ecae1e0>
2
```

综上，内部函数携带自由变量，被父函数以返回值的形式，传递到了外部，就构成了一个闭包。它们的作用范围已经脱离了定义体。

> 闭包三要素：嵌套函数，自由变量(保存了状态)，被返回到外部(改变了作用域)

** 为什么要用闭包？闭包的特性或优势？ **
> 因为闭包携带着自由变量，相当于一个自带状态的函数，能够实现类似类的功能。

> 考虑下 python 中的函数和类，都是 callable 对象。而装饰器既可以用函数也可以用类来实现，既可以修饰函数也可以修饰类！

```python
# PY3有 nonlocal，PY2 怎么办？
# 方法是：使用可变对象来代替，如 list, dict 自定义 class
def counter():
    x=[0]
    def incr():
        x[0] += 1 # 这样只是改变了 x 包含的对象，x 本身没变
        return x[0]
    return incr

c = counter()
c()
c()
```
```
2
```
```python
# 闭包的__closure__变量
print(c.__closure__)
print(c.__closure__[0].cell_contents)
```
```
(<cell at 0x10eb94f48: list object at 0x10ecd4788>,)
[2]
```
```python
# 普通函数, __closure__ = None
def fn():
    def ff():print('ff')
    return ff
f = fn() # 虽然是嵌套函数，但不携带自由变量
f()
print(f.__closure__)
```
```
ff
None
```
```python
# closure demo
def compare(base):
    def _cmp(v):
        return v > base
    return _cmp
cmp10 = compare(10)
print(cmp10(5), cmp10(20))
```
```
False True
```
```python
def mult(x):
    def _mul(y):
        return x*y
    return _mul
mult5 = mult(5)
mult5(10)
```
```
50
```
```python
from  urllib.request import urlopen
def geturl(url):
    def _get():
        return urlopen(url).read()
    return _get
get_baidu = geturl('http://www.baidu.com')
get_sina = geturl('http://www.sina.com')
# get_baidu()
# get_sina()
```
## 装饰器是一种特殊的闭包
> 接收一个函数作为参数，处理后，返回一个函数，并替换掉元函数

```python
#定义装饰器 d
def d(f): # 接收一个函数
    def _wrap(*args, **kws): #定义一个嵌套函数
        rtn = f(*args, **kws) #处理，并执行原函数 
        return rtn
    return _wrap # 返回一个函数
# 使用装饰器
@d
def f():pass
# 本质上等于
f=d(f) # 用返回的函数替换原函数
```
```python
# 装饰器是在 载入时 执行的
def d(f):
    print('before f')
    f() # 载入时就会执行
    print('after f')
#     return f
@d
def f():print('exec f')
# 如果装饰器不返回，那么 f 就为 None 了
# f = d(f)
```
```
before f
exec f
after f
```
```python
# 带参装饰器，相当于 fun = dec(d_args, k_kws)(fun), _dec是真正的装饰器
# def dec(*d_args, **d_kws): #装饰器参数
def dec(d_arg, d_kw=1): #装饰器参数
    print("return real decorator") # 载入时执行
    def _dec(f): # 真正的装饰器
        print("exec real decorator") # 载入时执行
        import functools
        @functools.wraps(f)
        def _wrap(*args, **kws): # 替换函数
            args = (args[0]+d_arg, args[1]*d_kw, *args[2:])
            print("preprocessing:", args)
            rtn = f(*args, **kws)
            return rtn
        return _wrap
    return _dec

@dec(1,d_kw=2)
def fun(x, y):
    '''return x+y'''
    return x+y
print(fun(2, 3))
print(fun.__name__, fun.__doc__)
```
```
return real decorator
exec real decorator
preprocessing: (3, 6)
9
fun return x+y
```
> 装饰后，fun 已经被 _wrap 替换了，所以原函数的信息丢失了，解决方法是 functools.wraps
