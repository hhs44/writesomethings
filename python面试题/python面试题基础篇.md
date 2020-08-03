[TOC]

# 基础篇

## 1. 在判断object是否是class的instances时，type和isinstance函数的区别？

- https://blog.csdn.net/bitcarmanlee/article/details/85263614
- type源代码

```
  def __init__(cls, what, bases=None, dict=None): # known special case of type.__init__
    """
    type(object) -> the object's type
    type(name, bases, dict) -> a new type
    # (copied from class doc)
    """
    pass
```

- isinstance源代码

```
def isinstance(p_object, class_or_type_or_tuple): # real signature unknown; restored from __doc__
  """
  isinstance(object, class-or-type-or-tuple) -> bool
  
  Return whether an object is an instance of a class or of a subclass thereof.
  With a type as second argument, return whether that is the object's type.
  The form using a tuple, isinstance(x, (A, B, ...)), is a shortcut for
  isinstance(x, A) or isinstance(x, B) or ... (etc.).
  """
  return False
```

- 异同：

- - 相同点

  - - 都可以判断变量是否属于某个内建类型。

  - 不同点

  - - 1.type只接受一个参数，不仅可以判断变量是否属于某个类型，还可以直接返回参数类型。而isinstance只能判断是否属于某个已知的类型，不能直接得到变量所属的类型。
    - 2.isinstance可以判断子类实例对象是属于父类的；而type会判断子类实例对象和父类类型不一样。

## 2. 通过重写内建函数，实现文件open之前检查文件格式？

代码实现：

```python
def open(filename,mode):
    import __builtin__
    file = __builtin__.open(filename,mode)
    
    if file.read(5) not in("GIF87", "GIF89"): 
        raise IOError, "not aGIF file"
    file.seek(0) 
    return file

fp = open("sample/test.gif","r")
print(len(fp.read()), "bytes")
```

## 3. 重新实现str.strip()，注意不能使用string.*strip()

代码实现：

```python
def leftStr(string, split=' '):
    startind = string.find(split)
    res = string
    while startind != -1 and startind ==0:
        res = res[startind+1:]
        startind = res.find(split)
    return res
```

## 4. 说明os,sys模块不同，并列举常用的模块方法？

+ os: 提供一种方便使用操作系统函数的方法

+ sys：提供访问由解释器使用或维护的变量和在与监视器交互使用到的函数

  + os常用方法：
    + os.remove()：删除文件
    + os.rename():重命名文件
    + **os.walk()：生成目录文件下的所有文件名称**
    + os.chdir()：改变目录
    + os.mkdir/makedirs：创建目录/多层目录
    + os.rmdir/removedirs：删除目录/多层目录
    + os.listdir()：列出指定目录文件
    + os.getcwd()：获取dangqian工作目录
    + os.chmod()：改变目录权限
    + os.path.basename()：去掉目录路径返回文件名
    + os.path.dirname()：去掉文件名，返回目录路径
    + os.path.join()：将分离的各部分组合成一个路径名
    + os.path.split()：返回元组（dirname，basename）
    + os.path.splitext()：返回元组（filename，extension）
    + os.path.getatime\ctime\mtime：返回最近访问/创建/修改/时间
    + os.path.getsize()：返回文件大小
    + os.path.exists()：是否存在
    + os.path.isabs()：是否为绝对路径
    + os.path.isdir()：是否是目录
    + os.path.isfile()：是否是文件
  + sys常用方法:yum:
    + sys.argv
    + sys.modules.keys()
    + sys.exc_info()
    + sys.exit()
    + sys.hexversion
    + sys.version
    + sys.maxunicode
    + sys.maxint
    + sys.modules
    + sys.path
    + sys.platform
    + sys.stdout
    + sys.stdin
    + sys.stderr
    + sys.exc_clear()
    + sys.exc_prefix
    + sys.byteorder
    + sys.copyright
    + sys.version_info

  ## 


## 6. os.path和sys.path的区别？

+ sys.path是由目录名构成的列表，Python 从中查找扩展模块( Python 源模块, 编译模块,或者二进制扩展). 启动 Python 时,这个列表从根据内建规则,PYTHONPATH 环境变量的内容, 以及注册表( Windows 系统)等进行初始化. 
+ os.path是module，包含了各种处理长文件名(路径名)的函数。

## 7. re模块中match和search方法的不同？

+ match() 函数从字符串的起始位置进行正则表达式匹配，返回`Match`对象或None，
+ 而 search() 则是扫描整个字符串，同样也是返回Match对象或None。

## 8.设计实现遍历目录与子目录，抓取.pyc文件?

.pyc文件是由.py文件经过编译后生成的字节码文件，其加载速度相对于之前的.py文件有所提高，而且还可以实现源码隐藏，以及一定程度上的反编译。比如，Python3.3编译生成的.pyc文件，Python3.4就别想着去运行啦！

.pyo文件也是优化（注意这两个字，便于后续的理解）编译后的程序（相比于.pyc文件更小），也可以提高加载速度。但对于嵌入式系统，它可将所需模块编译成.pyo文件以减少容量。

生成pyc文件：

```python
python -m py_compile /path/to/需要生成.pyc的脚本.py #若批量处理.py文件
                                                  #则替换为/path/to/{需要生成.pyc的脚本1,脚本2,...}.py
                                                  #或者/path/to/
                                                  
import py_compile
py_compile.compile(r'/path/to/需要生成.pyc的脚本.py') #同样也可以是包含.py文件的目录路径
                                                    #此处尽可能使用raw字符串，从而避免转义的麻烦。比如，这里不加“r”的话，你就得对斜杠进行转义
# 生成pyo文件
python -O -m py_compile /path/to/需要生成.pyo的脚本.py
```

**注意：**
以上无论是生成.pyc还是.pyo文件，都将在当前脚本的目录下生成一个含有字节码的文件夹**__pycache__**。



## 9、python的垃圾回收机制？

- python中主要通过引用计数（refreence     Counting）进行垃圾回收。

在Python中每一个对象的核心就是一个结构体PyObject，它的内部有一个引用计数（ob_refcnt）

- 引用的修改：

- - +1：创建，引用，作为参数传入函数，作为元素存储 

  - -1：对象别名被显示销毁del，对象别名被赋予新的对象，一个对象离开他的作用域，对象所在容器被销毁或是从容器中删除对象。

  - 获取引用计数：

  - - Sys.getrefcount(a)

  - 缺点：

  - - 逻辑简单实现麻烦，每个对象需要分配单独的空间来统计引用计数，加大了空间负担，维护容易出错。
    - 对一些场景，大数对象的释放，会消耗大量时间。如：字典
    - 循环引用：无解，致命问题

- 分代回收

- - 垃圾回收=垃圾检测+释放。

- 标记清除

- - 一个是root链表(root      object)，另外一个是unreachable链表。

## 10、 请至少列举5个 PEP8 规范？

- 缩进
- 每行最大长度79
- 类和top-level函数定义之间空两行，类中方法之间空一行
- 块注释 # 
- 右括号前不加空格
- 逗号、冒号、分号前不要加空格
- 函数的左括号前不要加空格。
- 序列的左括号前不要加空格。
- 操作符左右各加一个空格，不要为了对齐增加空格。

## 11、**在Python中如何实现单例模式**



- 单例模式：（解决全局变量的滥用）

- - 单例模式是一种设计模式，应用该模式的类只会生成一个实例

  - 单例模式保证在程序的不同位置可以且仅可以取到一个对象实例：如果不存在，则会创建一个实例，若存在则返回这个实例。

  - 单例模式的实现：

  - - https://www.cnblogs.com/huchong/p/8244279.html

1. 使用模块
   + 其实，**Python 的模块就是天然的单例模式**，因为模块在第一次导入时，会生成 `.pyc` 文件，当第二次导入时，就会直接加载 `.pyc` 文件，而不会再次执行模块代码。因此，我们只需把相关的函数和数据定义在一个模块中，就可以获得一个单例对象了。如果我们真的想要一个单例类，可以考虑这样做

 ```python
  # mysingleton.py
  class Singleton(object):
      def foo(self):
          pass
  singleton = Singleton()
  
  # 使用
  from mysingleton import singleton
 ```

  

2. 使用装饰器

 ```python
  def Singleton(cls):
      _instance = {}
  
      def _singleton(*args, **kargs):
          if cls not in _instance:
              _instance[cls] = cls(*args, **kargs)
          return _instance[cls]
  
      return _singleton
  
  # 测试
  @Singleton
  class A(object):
      a = 1
  
      def __init__(self, x=0):
          self.x = x
  
  
  a1 = A(2)
  a2 = A(3)
 ```

  

3. 使用类

- 基本实现

```python
class Singleton(object):

    def __init__(self):
        pass

    @classmethod
    def instance(cls, *args, **kwargs):
        if not hasattr(Singleton, "_instance"):
            Singleton._instance = Singleton(*args, **kwargs)
        return Singleton._instance
```

但该实现在多线程中存在问题，但线程执行太快时看不出问题。

```python
import threading

def task(arg):
    obj = Singleton.instance()
    print(obj)

for i in range(10):
    t = threading.Thread(target=task,args=[i,])
    t.start()
```

加入io操作后可以看出不同

```python
def __init__(self):
     import time
     time.sleep(1)
        
# output
#<__main__.Singleton object at 0x034A3410>
#<__main__.Singleton object at 0x034BB990>
#<__main__.Singleton object at 0x034BB910>
#<__main__.Singleton object at 0x034ADED0>
#<__main__.Singleton object at 0x034E6BD0>
#<__main__.Singleton object at 0x034E6C10>
#<__main__.Singleton object at 0x034E6B90>
#<__main__.Singleton object at 0x034BBA30>
#<__main__.Singleton object at 0x034F6B90>
#<__main__.Singleton object at 0x034E6A90>
```

+ 多线程中需要加锁

 ```python
  import time
  import threading
  class Singleton(object):
      _instance_lock = threading.Lock()
  
      def __init__(self):
          time.sleep(1)
  
      @classmethod
      def instance(cls, *args, **kwargs):
          if not hasattr(Singleton, "_instance"):
              with Singleton._instance_lock:
                  if not hasattr(Singleton, "_instance"):
                      Singleton._instance = Singleton(*args, **kwargs)
          return Singleton._instance
  
  
  def task(arg):
      obj = Singleton.instance()
      print(obj)
  for i in range(10):
      t = threading.Thread(target=task,args=[i,])
      t.start()
  time.sleep(20)
  obj = Singleton.instance()
  print(obj)
 ```

4. 使用__new__关键字实现单例

```python
import threading
class Singleton(object):
    _instance_lock = threading.Lock()

    def __init__(self):
        pass


    def __new__(cls, *args, **kwargs):
        if not hasattr(Singleton, "_instance"):
            with Singleton._instance_lock:
                if not hasattr(Singleton, "_instance"):
                    Singleton._instance = object.__new__(cls)  
        return Singleton._instance

obj1 = Singleton()
obj2 = Singleton()
print(obj1,obj2)

def task(arg):
    obj = Singleton()
    print(obj)

for i in range(10):
    t = threading.Thread(target=task,args=[i,])
    t.start()
```

5. 使用metaclass实现单例

+ 使用type创造类

```python
  class SingletonType(type):
      def __init__(self,*args,**kwargs):
          super(SingletonType,self).__init__(*args,**kwargs)
  
      def __call__(cls, *args, **kwargs): # 这里的cls，即Foo类
          print('cls',cls)
          obj = cls.__new__(cls,*args, **kwargs)
          cls.__init__(obj,*args, **kwargs) # Foo.__init__(obj)
          return obj
  
  class Foo(metaclass=SingletonType): # 指定创建Foo的type为SingletonType
      def __init__(self，name):
          self.name = name
      def __new__(cls, *args, **kwargs):
          return object.__new__(cls)
  
  obj = Foo('xx'）
```

## 12、不使用中间变量，交换两个变量`a`和`b`的值

 实现方法：

```
# 方法一
a = a ^ b
b = a ^ b
a = a ^ b
# 方法二
a, b = b, a
```

注：

ROT_TWO和ROT_THREE，实现的两个和三个变量的交换，三个以上的才叫做解包

## 13、假设你使用的是官方的CPython，说出下面代码的运行结果。

```python
a, b, c, d = 1, 1, 1000, 1000
print(a is b, c is d)

def foo():
    e = 1000
    f = 1000
    print(e is f, e is d)
    g = 1
    print(g is a)

foo()
```

结果：

```
True False
True False
True
```

注：CPython出于性能优化的考虑，把频繁使用的整数对象用一个叫`small_ints`的对象池缓存起来造成的。`small_ints`缓存的整数值被设定为`[-5, 256]`这个区间，也就是说，如果使用CPython解释器，在任何引用这些整数的地方，都不需要重新创建`int`对象，而是直接引用缓存池中的对象。如果整数不在该范围内，那么即便两个整数的值相同，它们也是不同的对象。

## 14、写一个删除列表中重复元素的函数，要求去重后元素相对位置保持不变。

```python
# 方法一：
def dedup(items):
    no_dup_items = []
    seen = set()
    for item in items:
        if item not in seen:
            no_dup_items.append(item)
            seen.add(item)
    return no_dup_items
# 方法二
def dedup(items):
    seen = set()
    for item in items:
        if item not in seen:
            yield item
            seen.add(item)
```

## 15、Lambda函数是什么，举例说明的它的应用场景。

Lambda函数其实最为主要的用途是把一个函数传入另一个高阶函数（如Python内置的`filter`、`map`等）中来为函数做解耦合，增强函数的灵活性和通用性。

```python
def multiply():
    return [lambda x: i * x for i in range(4)]

print([m(100) for m in multiply()])
```

运行结果：

```text
[300, 300, 300, 300]
```

注：需要注意的是这里有**闭包（closure）现象**，`multiply`函数中的局部变量`i`的生命周期被延展了，由于`i`最终的值是`3`

+ 如何避开闭包现象

  ```python
  # 使用偏函数，彻底避开闭包现象。
  from functools import partial
  from operator import __mul__
  
  def multiply():
      return [partial(__mul__, i) for i in range(4)]
  
  print([m(100) for m in multiply()])
  ```

## 16、**说说Python中的浅拷贝和深拷贝。**（`copy`模块中的`copy`和`deepcopy`函数）

浅拷贝通常只复制对象本身，而深拷贝不仅会复制对象，还会递归的复制对象所关联的对象。

深拷贝可能会遇到两个问题：

+ 一是一个对象如果直接或间接的引用了自身，会导致无休止的递归拷贝；
+ 二是深拷贝可能对原本设计为多个对象共享的数据也进行拷贝。

解决方法：`deepcopy`可以通过`memo`字典来保存已经拷贝过的对象，从而避免自引用递归问题

注：列表的切片操作`[:]`相当于实现了列表对象的浅拷贝

### 16-1. deepcopy 和 copy的区别？

+ copy:

  + 当最外层对象为可变类型时，copy后得到的对象指向新的内存空间，当最外层的对象为不可变类型时，copy后得到的对象指向原对象的内存空间（注意：浅拷贝的对象的最外层是否是可变类型）
+ deepcopy：除拷贝对象本身，还拷贝对象中引用的其他对象

  + 拷贝的内容中只要有一个是个可变类型，那么deepcopy一定是深拷贝

### 16-2. 常见案例

+ 常见的浅拷贝有：切片操作、工厂函数、对象的copy()方法、copy模块中的copy函数。

```python
>>> a = [1, 2, 3]
>>> b = list(a)
>>> print(id(a), id(b))          
# a和b身份不同
140601785066200 140601784764968
>>> for x, y in zip(a, b):       # 但它们包含的子对象身份相同...     
		print(id(x), id(y))
		... 
140601911441984 140601911441984
140601911442016 140601911442016
140601911442048 140601911442048

```

+ 深拷贝只有一种方式：copy模块中的deepcopy函数。

1、赋值：简单地拷贝对象的引用，两个对象的id相同。
2、浅拷贝：创建一个新的组合对象，这个新对象与原对象共享内存中的子对象。
3、深拷贝：创建一个新的组合对象，同时递归地拷贝所有子对象，新的组合对象与原对象没有任何关联。虽然实际上会共享不可变的子对象，但不影响它们的相互独立性。

## 17、python 的dict底层是什么？

- 3.6之前字典是无序的，3.7+中字典是有序的，字典也被称为关联数组，还称为哈希数组等。

- 字典底层维护的是一张哈希表，每一个哈希表中存储了哈希值、键、值（3.6）

- 3.7+的字典中还使用了indeices表进行辅助

- - 计算key的hash值【hash(key)】，再和mask做与操作【mask=字典最小长度（IndicesDictMinSize）      -      1】，运算后会得到一个数字【index】，这个index就是要插入的indices的下标位置（注：具体算法与Python版本相关，并不一定一样）
  - 得到index后，会找到indices的位置，但是此位置不是存的hash值，而是存的len(enteies)，表示该值在enteies中的位置
  - 如果出现hash冲突，则处理方式与老字典处理方式类似

- 哈希冲突解决方法：

- - 开放寻址
  - 再哈希
  - 链地址
  - 公共值溢出区
  - 装填因子

## 18、python的装饰器

functools.wraps的原理，atexit

- 装饰器是可以实现在不修改原函数的前提下对函数进行功能扩展的语法糖，符合“封闭开放”的开发原则。其原理是使用闭包的方法对函数进行“装饰”。

- atexit：退出处理器，定义了清理函数的注册和反注册函数

- - atexit.register(func,      *args, **kwargs)
  - atexit.unregister(func)

- wraps的原理

- - 将装饰器内部函数的元信息改为被装饰函数的元信息
  - 返回一个偏函数update_wrapper。该函数接收wrapper参数，也就是装饰器内部函数

## 19、列表生成式、字典生成式

+ 参考链接：https://www.cnblogs.com/nadech/p/8035035.html

1. 列表生成式

   1. `list(range(10))`
   2. `[i*i for i in range(10)]`

2. 字典生成器

   ```python
   #d = {key: value for (key, value) in iterable}
   d1 = {'x': 1, 'y': 2, 'z': 3}
   d2 = {k: v for (k, v) in d1.items()}
   print(d2)
   ```

3. 集合生成器

   ```
   s1={x for x in range(10)}
   print(s1)
   ```

## 20、使用Python代码实现遍历一个文件夹的操作。

Python标准库`os`模块的`walk`函数提供了遍历一个文件夹的功能，它返回一个生成器。

```python
import os

g = os.walk('/Users/Hao/Downloads/')
for path, dir_list, file_list in g:
    for dir_name in dir_list:
        print(os.path.join(path, dir_name))
    for file_name in file_list:
        print(os.path.join(path, file_name))
```

## 21. 闭包(closure)

**闭包无法修改外部函数的局部变量**。

**python循环中不包含域的概念**

闭包可以保存当前的运行环境，

闭包避免了使用全局变量，此外，闭包允许将函数与其所操作的某些数据（环境）关连起来。这一点与面向对象编程是非常类似的，在面对象编程中，对象允许我们将某些数据（对象的属性）与一个或者多个方法相关联。

所有函数都有一个 `__closure__`属性，如果这个函数是一个闭包的话，那么它返回的是一个由 cell 对象 组成的元组对象。cell 对象的cell_contents 属性就是闭包中的自由变量。、

简单的说，可以将闭包理解为**能够读取其他函数内部变量的函数**。

## 21.鸭子类型和猴子补丁

猴子补丁：

仅指在运行时动态改变类或模块，为的是将第三方代码打补丁在不按预期运行的bug或者feature上 。在运行时动态修改模块、类或函数，通常是添加功能或修正缺陷。猴子补丁在代码运行时内存中发挥作用，不会修改源码，因此只对当前运行的程序实例有效。因为猴子补丁破坏了封装，而且容易导致程序与补丁代码的实现细节紧密耦合，所以被视为临时的变通方案，不是集成代码的推荐方式。

热补丁。。。。。

鸭子类型：

鸭子类型是动态语言判断一个对象是否是某种类型时使用的方法，也叫做鸭子判定法。

Python语言中，有很多bytes-like对象（如：`bytes`、`bytearray`、`array.array`、`memoryview`）、file-like对象（如：`StringIO`、`BytesIO`、`GzipFile`、`socket`）、path-like对象（如：`str`、`bytes`），其中file-like对象都能支持`read`和`write`操作，可以像文件一样读写，这就是所谓的对象有鸭子的行为就可以判定为鸭子的判定方法。再比如Python中列表的`extend`方法，它需要的参数并不一定要是列表，只要是可迭代对象就没有问题。

## 22.面对对象设计原则

1. 单一职责原则（SRP）：一个类只有必要的属性和方法，一个函数只做好一件事情，高内聚
2. 开闭原则（OCP） -对推展开放，对修改关闭，抽象
3. 依赖倒转原则 （DIP）-面向抽象编程，面向接口编程-尽可能的使用抽象类型让系统有拓展性
4. 里氏替换（LSP） -能够用父类的地方一定可以使用字类型
5. 接口隔离（ISP） - 
6. 合成聚合复用原则（CARP） - 有限使用强关联而不是继承关系复用代码
7. 迪米特法则（LOD） -不要更陌生人讲话，低耦合

SOLID原则 = SRP + OCP + LSP + ISP + LoD

**面向对象：**面向对象编程简单来说就是基于对 类 和 对象 的使用，所有的代码都是通过类和对象来实现的编程就是面向对象编程！面向对象的三大特性：封装、继承、多态

~ 将数据和操作数据的方法从逻辑上变成一个整体 ---> 对象 

~ 类是对象的模板，通过类可以创建出对象，通过给对象发消息来解决问题.

~ 封装：隐藏实现细节，暴露简单的调用接口

~ 继承：从已有的类创建新类的过程，提供继承信息的叫父类（超类、基类），得到继承信息的叫子类（派生类）

~ 多态：给对象发出相同的消息，不同的对象会产生不同的行为。

## 23.python中的反射

在反射机制就是在运行时，动态的确定对象的类型，并可以通过字符串调用对象属性、方法、导入模块，是一种基于字符串的事件驱动。通过字符串的形式，去模块寻找指定函数，并执行。利用字符串的形式去对象（模块）中操作（查找/获取/删除/添加）成员。

Python是一门解释型语言，因此对于反射机制支持很好。在Python中支持反射机制的函数有getattr()、setattr()、delattr()、exec()、eval()、__import__，这些函数都可以执行字符串。

## 24.python中的自省的方法

**运行时能够获知对象的类型。**

type()，判断对象类型

dir()， 带参数时获得该对象的所有属性和方法；不带参数时，返回当前范围内的变量、方法和定义的类型列表

help() ,  用于查看函数或模块用途的详细说明

isinstance()，判断对象是否是已知类型

issubclass()，判断一个类是不是另一个类的子类

hasattr()，判断对象是否包含对应属性

getattr()，获取对象属性

setattr()， 设置对象属性

id(): 用于获取对象的内存地址

callable()：判断对象是否可以被调用。

## 25.python中的魔法方法

| 魔法方法                                                     | 作用               |
| ------------------------------------------------------------ | ------------------ |
| `__new__`,`__init__`,`__del__`                               | 创建和销毁对象相关 |
| `__add__`,`__sub__`,`__mul__`,<br />`__div__`,`__floordiv__`，`__mod__` | 算术运算符相关     |
| `__eq__`,`__ne__`,`__lt__`,<br />`__gt__`,`__le__`,`__ge__`  | 关系运算符         |
| `__pos__`,`__neg__`,`__invert__`                             | 一元运算符         |
| `_lshift__`,`__rshift__`,`__and__`<br />`__or__`,`__xor__`   | 位运算             |
| `__enter__`,`__exit__`                                       | 上下文管理器协议   |
| `__iter__`,`__next__`,`__reversed__`                         | 迭代器协议         |
| `__int__`,`__long__`,`__float__`<br />`__oct__`,`__hex__`    | 类型/进制转换      |
| `__str__`,`__repr__`,`__hash__`<br />`__dir__`               | 对象表述相关       |
| `__len__`,`__getitem__`,`__setitem__`<br />`__contains__`,`__missing__` | 序列相关           |
| `__copy__`,`__deepcopy__`                                    | 对象拷贝相关       |
| `__call__`,`__setattr__`,`__getattr__`<br />`__delattr__`    | 其他               |

## 26.函数参数`*arg`和`**kwargs`分别代表什么？

函数的参数分为位置参数、可变参数、关键字参数、命名关键字参数。

*arg代表可变参数，可以接受0个或任意多个参数，参数会被打包成一个元组

**kwargs代表关键字参数，可以接受用参数名=参数值的方式传入参数，传入的参数会打包成一个字典

## 27.python中变量的作用域

Python中有四种作用域，分别是局部作用域（**L**ocal）、嵌套作用域（**E**mbedded）、全局作用域（**G**lobal）、内置作用域（**B**uilt-in），搜索一个标识符时，会按照**LEGB**的顺序进行搜索，如果所有的作用域中都没有找到这个标识符，就会引发`NameError`异常。

## 28.random模块生成随机数、实现随机乱序和水机抽样

+ random.random()函数可以生成[0.0, 1.0)之间的随机浮点数。
+ random.uniform(a, b)函数可以生成[a, b]或[b, a]之间的随机浮点数。
+ random.randint(a, b)函数可以生成[a, b]或[b, a]之间的随机整数。
+ random.shuffle(x)函数可以实现对序列x的原地随机乱序。
+ random.choice(seq)函数可以从非空序列中取出一个随机元素。
+ random.choices(population, weights=None, *, cum_weights=None, k=1)函数可以从总体中随机抽取（有放回抽样）出容量为k的样本并返回样本的列表，可以通过参数指定个体权重，如果没有指定权重，个体被选中的概率均等。
+ random.sample(population, k)函数可以从总体中随机抽取（无放回抽样）出容量为k的样本并返回样本的列表。
+ 



## 29.举例说明什么情况下会出现`KeyError`、`TypeError`、`ValueError`。

举一个简单的例子，变量`a`是一个字典，执行`int(a['x'])`这个操作就有可能引发上述三种类型的异常。如果字典中没有键`x`，会引发`KeyError`；如果键`x`对应的值不是`str`、`float`、`int`、`bool`以及`bytes-like`类型，在调用`int`函数构造`int`类型的对象时，会引发`TypeError`；如果`a[x]`是一个字符串或者字节串，而对应的内容又无法处理成`int`时，将引发`ValueError`。

## 30.如何读取大文件，例如内存只有4G，如何读取一个大小为8G的文件？

很显然4G内存要一次性的加载大小为8G的文件是不现实的，遇到这种情况必须要考虑多次读取和分批次处理。在Python中读取文件可以先通过`open`函数获取文件对象，在读取文件时，可以通过`read`方法的`size`参数指定读取的大小，也可以通过`seek`方法的`offset`参数指定读取的位置，这样就可以控制单次读取数据的字节数和总字节数。除此之外，可以使用内置函数`iter`将文件对象处理成迭代器对象，每次只读取少量的数据进行处理，代码大致写法如下所示。

```python
with open('...', 'rb') as file:
    for data in iter(lambda: file.read(2097152), b''):
        pass
```

## 31.对模块和包的理解

每个python文件就是一个模块，而保存这些文件的文件夹就是一个包，包里包含一个`__init__.py`，包里还有子包的情况下子包中`__init__.py`不是必须的

## 32. 什么是lambda函数，他有什么好处？

lambda可快速定义单行最小函数，一个可以接收任意多个参数，并且返回单个表达式值得匿名函数。轻便即用即扔 。

## 33.是否遇到过python的模块间循环引用的问题，如何避免？

如果老是觉得碰到循环引用可能的原因有几点：

1. 可能是模块的分界线划错地方了
2. 可能是把应该在一起的东西硬拆开了
3. 可能是某些职责放错地方了
4. 可能是应该抽象的东西没抽象

总之微观代码规范可能并不能帮到太多，重要的是更宏观的划分模块的经验技巧，推荐uml，脑图，白板等等图形化的工具先梳理清楚整个系统的总体结构和职责分工

采取办法，从设计模式上来规避这个问题，比如:

1. 使用 `“__all__”` 白名单开放接口
2. 尽量避免 import