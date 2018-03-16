---
layout:     post
title:      "Python Decorator 第二讲"
subtitle:   "关于装饰器"
date:       2018-03-16
author:     "leehp258"
header-img: "img/home-bg.jpg"
tags:
    - Python
    - Decorator
    - 装饰器
---

# Decorator 第二讲

- 函数作为装饰器，装饰函数
- 函数作为装饰器，装饰类 (类装饰器) 【被装饰类变为了 function，不能再被继承了】
- 类作为装饰器，装饰函数
- 类作为装饰器，装饰类

## 总览
类能作为装饰器本质上是因为类是可调用对象，只要实现 __call__方法，类的实例对象可以像函数一样执行。注意不带参和带参装饰器__call__的调用时机不一样。

- 函数作为装饰器
    - 装饰函数
f=d(f)
f=d(x)(f)
    - 类装饰器: 被装饰类变为了 function，不能被继承了, 装饰器相当于类工厂
c=d(C)
c=d(x)(C)
- 类作为装饰器: 实现__init__, __call__
> 如果装饰器不带参数，被装饰对象作为__init__的第一个参数，__call__在运行时调用<br/>
> 如果装饰器带参数, 被装饰对象作为__call__的第一个参数，__call__在载入时调用

    - 装饰函数: 
f=D(f) 原函数变为了装饰类的实例对象
f=D(x)(f) 原函数变为了装饰类中返回的装饰函数
    - 装饰类:
c=D(C) 原类变为了装饰类的实例对象
c=D(x)(C) 原类变为了装饰类中返回的装饰函数，相当于类工程函数

## 函数作为装饰器，装饰函数

> 注意：装饰器在编译时执行; 

> f=d(f); f=d(x)(f)

```python
# demo1 这个是装饰器吗？
def d(f):
    print('d', f.__code__) # 编译时执行
    return f

@d
def f(x):print('f', x)
print("^^^^^ 此处之上为编译时执行，之后是运行时执行 vvvvv")
print(f) # 注意此处和 demo2 的差异
print(f.__closure__) #因为没有用内部包装函数替换 f, 此处并非闭包，返回 None
f(100)
```
```
d <code object f at 0x10ec1e270, file "<ipython-input-15-3430038345bd>", line 6>
^^^^^ 此处之上为编译时执行，之后是运行时执行 vvvvv
<function f at 0x10ebbabf8>
None
f 100
```
```python
# demo2 简单的闭包和装饰器
def d2(f):
    print('d2')
    def wrap(*arg):
        print("d2.wrap")
        return f(*arg)
    print("d2.wrap:", wrap, id(wrap)) # 注意与 f2 的比较，内存信息 和 id 都是一样的, wrap 就是 f2
    return wrap

@d2
def f2(x):print("f2", x)
print("^^^^^ 此处之上为编译时执行，之后是运行时执行 vvvvv")
print(f2, id(f2)) # f2已经被 d2.<locals>.wrap 替换掉了
print(f2.__closure__, len(f2.__closure__), f2.__closure__[0].cell_contents)
f2(200)
```
```
d2
d2.wrap: <function d2.<locals>.wrap at 0x10ec229d8> 4542573016
^^^^^ 此处之上为编译时执行，之后是运行时执行 vvvvv
<function d2.<locals>.wrap at 0x10ec229d8> 4542573016
(<cell at 0x10eb7e558: function object at 0x10ec21510>,) 1 <function f2 at 0x10ec21510>
d2.wrap
f2 200
```
```python
# demo3 functools.wraps 保留函数信息
def d3(f):
    import functools
    @functools.wraps(f)
    def wrap(*arg, **kw):
        print("d3.wrap")
        return f(*arg, **kw)
    print("d3.wrap:", wrap, id(wrap)) # 函数信息已经被 functools.wraps 重置了
    return wrap

@d3
def f3(x, y=0):print("f3", x, y)
print("^^^^^ 此处之上为编译时执行，之后是运行时执行 vvvvv")
print(f3, id(f3)) #f3已经被替换掉了，但是保留了函数信息
print(f3.__closure__, len(f3.__closure__), f3.__closure__[0].cell_contents)
f3(300)
```
```
d3.wrap: <function f3 at 0x10ebba730> 4542146352
^^^^^ 此处之上为编译时执行，之后是运行时执行 vvvvv
<function f3 at 0x10ebba730> 4542146352
(<cell at 0x10eb7ef48: function object at 0x10eb6fa60>,) 1 <function f3 at 0x10eb6fa60>
d3.wrap
f3 300 0
```
```python
# demo4 带参函数装饰器
def d4(x, y):
    print("d4:", x, y)
    def dec_real(f):
        print("dec_real", x, y)
        import functools
        @functools.wraps(f)
        def wrap(*arg, **kw):
            print("d4.dec_real.wrap", x, y)
            return f(*arg, **kw)
        print("d4.wrap:", wrap, id(wrap))
        return wrap
    return dec_real

@d4('hello', 10000)
def f4(x, y=1):print("f4", x, y)
print("^^^^^ 此处之上为编译时执行，之后是运行时执行 vvvvv")
print(f4, id(f4))
print(f4.__closure__, len(f4.__closure__), f4.__closure__[0].cell_contents, f4.__closure__[1].cell_contents)
f4(400)
```
```
d4: hello 10000
dec_real hello 10000
d4.wrap: <function f4 at 0x10ec21b70> 4542569328
^^^^^ 此处之上为编译时执行，之后是运行时执行 vvvvv
<function f4 at 0x10ec21b70> 4542569328
(<cell at 0x10eb7e798: function object at 0x10ec219d8>, <cell at 0x10eb7eeb8: str object at 0x10ebf5ab0>, <cell at 0x10eb7ecd8: int object at 0x10eb840b0>) 3 <function f4 at 0x10ec219d8> hello
d4.dec_real.wrap hello 10000
f4 400 1
```
## 函数作为装饰器，装饰类 (类装饰器)
> 装饰器应用于类，用于管理类自身，或者拦截实例创建调用以管理实例。

> 被装饰类变为了 function，不能被继承了, 装饰器相当于类工厂

类装饰器在实例化类之前执行，可以决定是否要实例化被装饰类

```python
# demo5 类装饰器实现单例模式
def singleton(cls, *args, **kw):
    print("singleton", cls)
    instance={}
    def _singleton():
        print("_singleton")
        if cls not in instance:
            print("cls instance prepare")
            instance[cls]=cls(*args, **kw)
            print("cls instance done!")
        return instance[cls]
    return _singleton

@singleton
class singleclass(object):
    def __new__(cls):
        print("singleclass.new")
        return super().__new__(cls)
    def __init__(self):
        self.n = 0
        print("singleclass.init", self.n)
    def count(self):
        self.n += 1
        print(self.n)

print("^^^^^ 此处之上为编译时执行，之后是运行时执行 vvvvv")
cls1 = singleclass()
cls2 = singleclass()
cls1.count()
cls2.count()
print(cls1, cls2, type(singleclass), type(cls1))
```
```
singleton <class '__main__.singleclass'>
^^^^^ 此处之上为编译时执行，之后是运行时执行 vvvvv
_singleton
cls instance prepare
singleclass.new
singleclass.init 0
cls instance done!
_singleton
1
2
<__main__.singleclass object at 0x10ebab518> <__main__.singleclass object at 0x10ebab518> <class 'function'> <class '__main__.singleclass'>
```
### 类装饰器的不足：
**类被装饰后变成了函数，无法再被继承，虽然原类在本质上还是存在的，但是被装饰函数隔离了**

对于 cls1 = singleclass()：

被装饰前: cls1 是实例，singleclass 是其类<br/>
被装饰后: cls1 是实例，singleclass 是控制类实例化的函数，原来的类被隔离了

无法再操作原来的类: 无法被继承

```python
def cdec(cls):
    def _class(*args, **kw):
        return cls(*args, **kw)
    return _class

@cdec
class AA():
    a = 0
    def __init__(self):
        self.i = 0
        
# AA无法被继承
# class AAA(AA):pass # TypeError: function() argument 1 must be code, not str

a = AA()
print(a.a)
```
```
0
```
## 类作为装饰器，装饰函数
python 类也是可调用(callable)对象，通过__call__来实现

**类作为装饰器必须实现 __init__, __call__：**

- 如果装饰器不带参数，被装饰对象作为__init__的第一个参数
- 如果装饰器带参数, 被装饰对象作为__call__的第一个参数

> F = Dec(F) : 原函数变为了装饰类的实例对象

> F = Dec(x)(F) : 原函数变为了装饰类中返回的装饰函数

```python
class Dec1(object):
    def __init__(self, f):
        print("Dec1.__init__", f)
        self.f = f
    def __call__(self, *args, **kw):
        print("Dec1.__call__", args, kw)
        self.f(*args, **kw)
@Dec1
def F1(x):print("F1", x)
print("^^^^^ 此处之上为编译时执行，之后是运行时执行 vvvvv")
print("F1 is:", F1, type(F1))
F1(100)
```
```
Dec1.__init__ <function F1 at 0x10ec1cf28>
^^^^^ 此处之上为编译时执行，之后是运行时执行 vvvvv
F1 is: <__main__.Dec1 object at 0x10ec62358> <class '__main__.Dec1'>
Dec1.__call__ (100,) {}
F1 100
```
```python
class Dec2():
    '''支持带参和不带参装饰:
    @Dec2
    @Dec2(x)
    '''
    def __init__(self, x):
        self.x = x # x 有可能是装饰器的参数，有可能是被装饰函数
    def __call__(self, *args, **kw):
        if callable(self.x): # x 是被装饰函数
            print("Dec2.__call__.callable", args, kw)
            self.x(*args, **kw)
        else:
            print("Dec2.__call__.dec_param", self.x, args, kw)
            import functools as ft
            f = args[0]
            @ft.wraps(f)
            def wrap(*args, **kw):
                print("Dec2.__call__.locals.wrap", self.x, f, args, kw)
                return f(*args, **kw)
            return wrap
            
@Dec2
def F2(x, y=2):print("F2", x, y)
print("^^^^^ 此处之上为编译时执行，之后是运行时执行 vvvvv")
print("F2 is:", F2, type(F2))
F2(100, y=2)
print('- '*30)
@Dec2(8)
def F3(x, y=3):print("F3", x, y)
print("^^^^^ 此处之上为编译时执行，之后是运行时执行 vvvvv")
print("F3 is:", F3, type(F3))
F3(200, y=300)
```
```
^^^^^ 此处之上为编译时执行，之后是运行时执行 vvvvv
F2 is: <__main__.Dec2 object at 0x10ec30c50> <class '__main__.Dec2'>
Dec2.__call__.callable (100,) {'y': 2}
F2 100 2
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
Dec2.__call__.dec_param 8 (<function F3 at 0x10ec6fd90>,) {}
^^^^^ 此处之上为编译时执行，之后是运行时执行 vvvvv
F3 is: <function F3 at 0x10ec8b048> <class 'function'>
Dec2.__call__.locals.wrap 8 <function F3 at 0x10ec6fd90> (200,) {'y': 300}
F3 200 300
```
## 类作为装饰器，装饰类

> c = Dec(C) : 原类变为了装饰类的实例对象

> c = Dec(x)(C) : 原类变为了装饰类中返回的装饰函数

```python
class Dec3():
    def __init__(self, *args, **kw):
        self.args = args
        self.kw = kw
        self.cls = {}
        print("Dec3.__init__:", self.args, self.kw)
    def __call__(self, *args, **kw):
        arg0 = self.args[0]
        if callable(arg0):
            print("Dec3.__call__.callable", arg0, args, kw)
            if arg0 not in self.cls:
                self.cls[arg0] = arg0(*args, **kw)
            return self.cls[arg0]
        else:
            print("Dec3.__call__.dec_param", self.args, args, kw)
            import functools as ft
            f = args[0]
            @ft.wraps(f)
            def singleton(*args, **kw):
                if f not in self.cls:
                    self.cls[f] = f(*args, **kw)
                return self.cls[f]
            return singleton
        
@Dec3
class clz3():
#     def __new__(cls, *args, **kwargs):
#         print("clz3.__new__", cls, args, kwargs)
#         return super(cls, cls).__new__(cls, *args, **kwargs)

    def __init__(self, *args, **kw):
        print("clz3.__init__", args, kw)
        self.args = args
        self.kw = kw
        
print("^^^^^ 此处之上为编译时执行，之后是运行时执行 vvvvv")
print("clz3 is:", clz3, type(clz3))

c1 = clz3(100, y=200)
c2 = clz3(200, y=300)
print(c1, c2)
print(c1.args, c1.kw, c2.args, c2.kw)

print('- '*30)
@Dec3(10,y=20)
class clz33():
    def __init__(self, *args, **kw):
        print("clz33.__init__", args, kw)
        self.args = args
        self.kw = kw

print("^^^^^ 此处之上为编译时执行，之后是运行时执行 vvvvv")
print("clz33 is:", clz33, type(clz33))
c4 = clz33(100, y=200)
c5 = clz33(200, y=300)
print(c4, c5)
print(c4.args, c4.kw, c5.args, c5.kw)
```
```
Dec3.__init__: (<class '__main__.clz3'>,) {}
^^^^^ 此处之上为编译时执行，之后是运行时执行 vvvvv
clz3 is: <__main__.Dec3 object at 0x10ebf5d68> <class '__main__.Dec3'>
Dec3.__call__.callable <class '__main__.clz3'> (100,) {'y': 200}
clz3.__init__ (100,) {'y': 200}
Dec3.__call__.callable <class '__main__.clz3'> (200,) {'y': 300}
<__main__.clz3 object at 0x10ebf56a0> <__main__.clz3 object at 0x10ebf56a0>
(100,) {'y': 200} (100,) {'y': 200}
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
Dec3.__init__: (10,) {'y': 20}
Dec3.__call__.dec_param (10,) (<class '__main__.clz33'>,) {}
^^^^^ 此处之上为编译时执行，之后是运行时执行 vvvvv
clz33 is: <function clz33 at 0x10ec8b2f0> <class 'function'>
clz33.__init__ (100,) {'y': 200}
<__main__.clz33 object at 0x10ebf5b38> <__main__.clz33 object at 0x10ebf5b38>
(100,) {'y': 200} (100,) {'y': 200}
```