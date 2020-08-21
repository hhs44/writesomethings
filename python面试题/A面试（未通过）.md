

## python相关

1. 实现一个单例模式

```python

def Singleton(cls):
    _instance = {}
    
    def _singleton(*args, **kargs):
        if cls not in _instance:
            _instance[cls] = cls(*args, **kargs)
         return _instance[cls]
   return _singleton

import threading
class Singleton():
    _instance_lock = threading.Lock()
    
    def __new__(cls, *args, **kwargs):
        if not hasattr(Singleton,"_instance"):
            with Singlenton,_instace_lock:
                if not hasattr(Singleton, "_instance"):
                    Singleton._instance = object.__new__(cls)
         return Singleton._instance
    
```



1. 实现一个装饰器

   ```python
   def f(func):
       
       def wrapper(x, *args, **kwargs):
           res = []
          	return res
      	return wrapper
   ```

## 数据库相关

1. 数据库的数据是实时更新的吗？每点击一次，数据库数据修改一次？ 

   1. 

2. **Redis hash的个数**

   1. Hashes类型:键值对个数最多为2^32-1个
   2. string类型：值最大可以存储512M
   3. Lists类型：个数最多2^31-1个
   4. Sets类型：。。。

3. 如何修改Redis数据库的库的个数？

   1. 设置数据库数量：databases 16

4. Redis数据库如何实现持久化

   如果开启了AOF持久化功能，服务器则会优先使用AOF文件来还原数据库状态。

   1. RDB持久化

      + 数据安全性相对弱

      + 通过异步将内存数据库的快照写入持久化文件来实现数据的持久化。

      + 注重性能，妥协安全

      + Redis提供了两个命令：`SAVE`和`BGSAVE`来创建RDB快照文件。这两个命令的最大区别是：`SAVE`命令在生成RDB文件的时候会阻塞Redis的进程；而`BGSAVE`会创建一个子进程来生成RDB文件，不会阻塞服务器的进程。

      + #### 定时生成快照

        + ```
          # 只要满足下列条件之一，则会执行bgsave命令
          save 900 1 # 在900s内存在至少一次写操作
          save 300 10
          save 60 10000
          # 禁用RBD持久化，可在最后加 save ""
          
          # 当备份进程出错时主进程是否停止写入操作
          stop-writes-on-bgsave-error yes  
          # 是否压缩rdb文件 推荐no 相对于硬盘成本cpu资源更贵
          rdbcompression no
          ```

      + 优缺点

        + #### 优点

          1. 生成RDB快照文件的过程是异步的，所以在持久化过程中对服务器性能影响小。
          2. RDB文件存储的是内存数据库的快照，采用紧凑的二进制文件存储。通过RDB文件进行数据库恢复的时候速度快。

        + #### 缺点

          1. 由于RDB文件是异步进行备份的，所以存在数据安全性弱的弊端：当系统发生故障导致内存数据库数据丢失的时候，从RDB文件中只能恢复创建RDB快照那一刻的数据，在最近一次创建RDB快照那一刻到服务器宕机之间的数据将永久性的丢失了。数据恢复的完整程度依赖于RDB快照创建的频率。
          2. 由于RDB快照是将整个内存数据库备份下来，所以当内存数据库很大的时候创建RDB文件需要耗费更久的时间。

   2. AOF持久化

      + 持久化发送到Redis服务器的写命令

      + AOF持久化机制会把所有引起内存数据库数据变化的写命令都保存下来，通过文件追加（Append）的方式保存到AOF文件中。

      + 牺牲一定的性能来换取数据的安全性

      + 配置

        ```
        # 默认关闭AOF，若要开启将no改为yes 
        appendonly no 
        # append文件的名字 
        appendfilename "appendonly.aof" 
        # 每隔一秒将缓存区内容写入文件 默认开启的写入方式 
        appendfsync everysec  
        # 当AOF文件大小的增长率大于该配置项时自动开启重写（这里指超过原大小的100%）。 
        auto-aof-rewrite-percentage 100 
        # 当AOF文件大小大于该配置项时自动开启重写 
        auto-aof-rewrite-min-size 64mb
        ```

        

      + Redis通过设置`appendfsync`选项来控制持久化的数据安全程度。该参数提供了三个可选的AOF持久化选项值：`always`、`everysec`、`no`。分别对应不同的数据安全级别。

      | 选项值           | 作用                                                         |
      | ---------------- | ------------------------------------------------------------ |
      | always           | 每执行一次命令就进行AOF文件同步                              |
      | everysec（默认） | 每隔一秒进行一次AOF文件同步。由于同步AOF文件是阻塞操作，所以当前一次同步操作耗时超过1秒的时候，下一次同步操作将会到等上一次同步完成以后才进行，因此最差情况下会造成延迟2秒的同步 |
      | no               | Redis不主动进行AOF文件同步，而是交给操作系统定时进行文件同步 |

      + ### 优缺点

        #### 优点

        1. 数据的安全性更高，当服务crash以后丢失的数据更少。
        2. AOF文件的可读性更好。
        3. 通过append方式追加命令，访问磁盘的效率高。

        #### 缺点

        1. 由于AOF持久化会在一定程度上进行磁盘同步处理操作，这个过程是阻塞的（虽然很短），所以对服务器处理命令的性能会产生影响。
        2. AOF文件记录的是写命令，恢复的时候需要重放命令来得到内存数据库的状态，即使有AOF重写机制，恢复速度上和RDB相比也会有差距。

5. Redis数据库支持的数据类型

   1. string（字符串），hash（哈希），list（列表），set（集合）及zset(sorted set：有序集合)。

6. Redis使用AOF方式持久化，aof文件不断增大，如何处理？

   ```
   # 当AOF文件大小的增长率大于该配置项时自动开启重写（这里指超过原大小的100%）。 auto-aof-rewrite-percentage 100 
   # 当AOF文件大小大于该配置项时自动开启重写 
   auto-aof-rewrite-min-size 64mb
   ```

7. Redis数据库如何设置密码

   1. 临时修改：config set requirepass password
   2. 永久修改:配置文件requirepass password

8. hash表是如何生成的

   1. 

9. MySQL数据库如何使用sql语句插入一条数据

   1. 

10. MySQL数据库的慢查询有了解过吗

    1. 

11. MySQL数据库如何进行查询优化

12. 如何很多请求同时对Redis的同一个键进行访问，如何保证数据安全

13. 说说Redis的淘汰机制

14. 我的MySQL数据库每天晚上12点进行全备份。第二天有员工在9点钟误删除了一个数据库，但在10点钟才被发现。问如何进行恢复被误删除的数据库并同时保留9点到10点钟新增的数据同时不影响业务的正常运行?

15. 当数据越来越多，如何避免hash槽中key出现相同的情况?

16. **MongoDB在哪些场合使用过？**

    单个数据大 + 数据量多(海量数据) + 低价值，主要解决海量数据的访问效率问题。

    + 网站数据：mongo非常适合实时的插入，更新与查询，并具备网站实时数据存储所需的复制及高度伸缩性。
    + 缓存：由于性能很高，mongo也适合作为信息基础设施的缓存层。在系统重启之后，由mongo搭建的持久化缓存可以避免下层的数据源过载。
    + 大尺寸、低价值的数据：使用传统的关系数据库存储一些数据时可能会比较贵，在此之前，很多程序员往往会选择传统的文件进行存储。
    + 高伸缩性的场景：mongo非常适合由数十或者数百台服务器组成的数据库。
    + 用于对象及JSON数据的存储：mongo的BSON数据格式非常适合文档格式化的存储及查询。

不适合的场景：

​	a.高度事物性的系统：例如银行或会计系统。传统的关系型数据库目前还是更适用于需要大量原子性复杂事务的应用程序。

​	b.传统的商业智能应用：针对特定问题的BI数据库会对产生高度优化的查询方式。对于此类应用，数据仓库可能是更合适的选择。

​	c.需要SQL的问题。

16. 





# 项目部署相关

1. 大家都说Nginx快？快的原因是什么？
2. 对RPC了解吗?
3. 如何在服务器上设置业务进程数？
4. 说说正向代理和反向代理

# linux相关

1. 如何查看剩余内存
2. 如何查看端口是否被占用
3. 如何查看一个程序的PID以及它的所有子进程
4. 如何为一个目录下的所有文件添加权限
5. 如果你对一个目录具有写权限，那么你是否具有对这个目录下的所有文件具有删除权限？
6. 对Linux多路复用的理解
7. 修改IP地址的方法

## 前端相关

1. 对前端HTML CSS 和 JS了解多少？熟悉吗？
2. 对React和bootstrap了解吗?
3. 如何进行http优化？(响应头设置Content-Encoding: gzip)

# 网络编程相关

说一下实现TCP建立连接的过程以及当时进入了什么状态？为什么建立连接只需要3次，断开连接需要4次？为什么断开连接时第二次和第三次要分开，不能合在一起吗？

## 项目相关

1. 说一下一个请求过来到返回response的过程
2. 如何实现单点登录
3. JWT token是如何进行生成和校验的
4. 了解过哪些后端框架？Tornado了解吗?
5. 了解过webapp2吗
6. Django如何实现csrf攻击保护
7. 说说你项目中遇到的困难以及如何解决
8. 说说你认为自己最有成就感或最深刻的项目
9. 对KAFKA了解吗？用过哪些消息队列？使用过RabbitMQ吗?
10. 项目团队几个人？开发多长时间？

## 版本控制相关

1. 如何从远程仓库拉取分支到本地
2. 如何进行版本回退

## 其他

1. Celery的原理和应用场景
2. Elasticsearch 的原理
3. 平时是如何学习的?有关注哪些技术?
4. Docker的了解，常用命令，如何暴露端口
5. 对ERP了解吗？Odoo了解吗?





## 语言层面

1.精通Python语言，了解Python高级特性，了解设计模式，能够读懂开源框架代码。
2.前端要熟悉HTML/CSS/JS，了解ES6特性，至少会使用一个前端框架，例如JQuery或者Vue。

## 数据库

1.关系型数据库Mysql、Postgresql，性能调优

2. 非关系型数据库Mongodb（可选）
3. 缓存型数据库Redis （必备）

## Python框架

掌握Flask、Django、Tornado或其他Web框架，熟悉或者精通其中任意一个即可，能够了解这些框架的底层实现原理和机制。

## Web

熟悉TCP/UDP/HTTP协议等基础理论知识。
熟悉web常见的验证方式，如Basic Authentication、Token Authentication和JWT验证，熟悉第三方登录如OAuth2.0。
熟悉RESTful API的设计理念，熟悉CRUD基本操作
了解Web Sockets

## 搜索引擎

了解或者熟悉ElasticSearch、Solr、Sphinx

## 消息队列

了解RabbitMQ、Kafka

## 其他

熟悉Docker，能够使用Docker部署项目

## 系统

熟悉Linux的常见操作，熟悉云计算平台如阿里云、腾讯云、AWS，熟悉Nginx或者Apache的常见配置，能够熟练部署项目到Linux服务器上
