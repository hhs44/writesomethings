[TOC]



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
print len(fp.read()), "bytes"
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

+ match() 函数只检查 RE 是否在字符串开始处匹配，而 search() 则是扫描整个字符串。

## 8.设计实现遍历目录与子目录，抓取.pyc文件?