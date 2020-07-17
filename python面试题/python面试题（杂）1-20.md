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

  ## 5. deepcopy 和 copy的区别？

+ copy:
  
  + 当最外层对象为可变类型时，copy后得到的对象指向新的内存空间，当最外层的对象为不可变类型时，copy后得到的对象指向原对象的内存空间（注意：浅拷贝的对象的最外层是否是可变类型）
+ deepcopy：除拷贝对象本身，还拷贝对象中引用的其他对象
  
  + 拷贝的内容中只要有一个是个可变类型，那么deepcopy一定是深拷贝

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