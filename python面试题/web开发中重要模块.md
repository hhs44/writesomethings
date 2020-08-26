

[TOC]



# 数据库模块

## 可用数据库方案

+ mysql
+ postgresql
+ mongodb

## 连接池

## 数据库并发控制

+ 存在的问题

  1. 胀读，A读取，并更改后，B读取，A不提交更改，
  2. 不可重复读，A读取并创建副本后，B从系统中删除了实体
  3. 幻读：

+ 悲观锁：实体在应用中存储的整个生命周期内，在数据库中被锁定。

  ```python
  # select_for_update():使数据库锁住对象直到事务完成
  with atomic():
      user = User.objects.select_for_update().first()
      user.save()
  ```

+ 乐观锁：在已知冲突率较低的情况下，不进行阻止，使用检测冲突，并在冲突发生时解决它
  1. 使用唯一标识符标记源数据
     1. 日期时间戳，增量计数器，用户id，有全局唯一代理键生成器生成的值
  2. 保留源数据的副本

+ 冲突解决策略：
  + 放弃
  + 显示问题，让用户决定
  + 合并改动
  + 记录冲突让后来的人决定
  + 无视冲突

### 不同策略的应用

| 表类型                | 示例                         | 推荐策略          |
| --------------------- | ---------------------------- | ----------------- |
| 实时高并发            | 账号系统                     | 1. 乐观锁2.悲观锁 |
| 实时低并发            | 过客，账单                   | 1.悲观锁2.乐观锁  |
| 日志                  | 访问日志、账号历史、事务记录 | 过度乐观锁        |
| 查找/引用（通常只读） | 付款方式                     | 过度乐观锁        |





## 数据库扩展

+ 纵向扩展，横向拓展
  + 纵向：增强单个数据服务能力：增加cpu，增加内存，增加存储空间，使用更加强大服务器
  + 横向，使用多个数据库服务组合起来，技术要求较高
+ 读写分离（横向）：
  + 主从复制时单向的从主机到从机，至少需要定义两个数据源，一个用于写，一个用于读。
  + binlog日志，binlog日志传入从机告知从机有新的binlog写入。
  + 从机->两个线程处理复制，
    + I/O线程连接主服务器读取二进制日志事件，复制到本地日志文件中（中继日志），
    + sql线程，从中继日志中读取事件，并在本地中执行。
  + 在django中的实现：
    + 配置多个数据库
    + 实现DefaultRouter类，定义db_for_read,db_for_write
    + setting中添加数据库路由DATABASE_ROUTRS
    + 多个从机实现读的负载均衡
      + 创建DNS记录，（内网DNS）
      + 策略：随机选择
    + using关键字指定使用的数据库
+ 垂直分库
  + 大型数据，分库分表才有意义

## mysql优化

+ explain()方法了解queryset的细节
+ 使用索引，对经常查询的字段加上索引
+ 计算上移，计算相关尽量上移到应用层，如：少用order_by()，不推荐使用存储过程，的hi有QuerySet的count()和exists()会更快。
+ 使用缓存，mysql默认开启查询缓存，而curdate，now，rand等不会使用缓存
+ 选择较好的存储引擎。一般使用innoDB
+ 获取想要的数据。使用QuerySet中的values()和values_list()方法指定想要查询的字段
+ 设计ID字段
+ 批量操作，操作多个对象时，使用bulk_create()方法，这会减少sql查询的数量
+ 使用utf8mb4编码
+ 尽可能一次性获取想要的数据，使用select_related()和prefetch_related()





# 缓存模块

## http缓存

通常会被缓存的页面

+ 200/301/404/206

http协议中的Cache-Control头，用于指定请求和响应的缓存机制

+ no-store
+ no-cache
+ public
+ private：
+ max-age：
+ must-revalidate



## django中的缓存

+ 缓存

  + 时间和空间是不可调和的矛盾，软件和硬件在逻辑上是等效

  + 缓存是典型的空间换时间策略，池化技术（线程池、连接池等）也是典型的空间换时间策略

  + 例如：数据库连接池就是通过提前创建和保留数据库连接，减少创建释放连接时TCP三次握手四次挥手的开销

    ```
    1. 声明式缓存
    @cache_page(timeout=...) ---> FBV
    @method_decorator(cache_page(timeout=...), name='list') ---> CBV
    2. 编程式缓存
    django.core.cache.caches['...'] ---> set / get
    django_redis.get_redis_connection ---> Redis ---> 几乎所有命令
    可能需要手动序列化数据 pickle / json
    ```

    

+ 自定义缓存

## 缓存替换策略

缓存替换算法（Least Recently Used，LRU）

```python
class ListNode:
    def __init__(self, key=None, value=None):
        self.key = key
        self.value = value
        self.prev = None
        self.next = None


class LRUCache:
    def __init__(self, capacity: int):
        self.capacity = capacity
        self.dic = {}
        self.head = ListNode()
        self.tail = ListNode()
        
        self.head.next = self.tail
        self.tail.prev = self.head

    def move_node_to_tail(self, key):
        node = self.dic[key]
        node.prev.next = node.next
        node.next.prev = node.prev
        node.prev = self.tail.prev
        node.next = self.tail
        self.tail.prev.next = node
        self.tail.prev = node

    def get(self, key: int) -> int:
        if key in self.dic:
            # 如果已经在链表中了久把它移到末尾（变成最新访问的）
            self.move_node_to_tail(key)
        res = self.dic.get(key, -1)
        if res == -1:
            return res
        else:
            return res.value

    def put(self, key: int, value: int) -> None:
        if key in self.dic:
            # 如果key本身已经在哈希表中了就不需要在链表中加入新的节点
            # 但是需要更新字典该值对应节点的value
            self.dic[key].value = value
            # 之后将该节点移到末尾
            self.move_node_to_tail(key)
        else:
            if len(self.dic) == self.capacity:
                # 去掉哈希表对应项
                self.dic.pop(self.head.next.key)
                # 去掉最久没有被访问过的节点，即头节点之后的节点
                self.head.next = self.head.next.next
                self.head.next.prev = self.head
            # 如果不在的话就插入到尾节点前
            new = ListNode(key, value)
            self.dic[key] = new
            new.prev = self.tail.prev
            new.next = self.tail
            self.tail.prev.next = new
            self.tail.prev = new

```

其他：先进先出（FIFO），后进先出（LIFO），随机替换（RR）

### redis的缓存穿透和雪崩

+ 缓存穿透：缓存未起到保护后端存储系统，如：查询缓存时没有命中，则查询数据存储系统，当某个数据不存在，每次查询都不会命中，都会访问缓存和数据库

  + 解决：使用布隆过滤器来应对缓存穿透的问题

+ 雪崩：当大量缓存同一时间失效时，多个进程参与重新构建缓存，会对系统造成大量压力

  解决：

  + 缓存失效后通过加锁或者队列来控制读数据库和写缓存的线程数量
  + 为不同的缓存设置不同的过期时间，让缓存失效的时间点尽量均匀

### 其他

+ 缓存预热
+ 缓存更新
+ 缓存降级

# Redis相关

https://zhuanlan.zhihu.com/p/131517931

## Redis的过期键的删除策略

+ 定时过期：每个设置过期时间的key都需要创建一个定时器，到过期时间就会立即清除。该策略可以立即清除过期的数据，对内存很友好；但是会占用大量的CPU资源去处理过期的数据，从而影响缓存的响应时间和吞吐量。
  - 惰性过期：只有当访问一个key时，才会判断该key是否已过期，过期则清除。该策略可以最大化地节省CPU资源，却对内存非常不友好。极端情况可能出现大量的过期key没有再次被访问，从而不会被清除，占用大量内存。
  - 定期过期：每隔一定的时间，会扫描一定数量的数据库的expires字典中一定数量的key，并清除其中已过期的key。该策略是前两者的一个折中方案。通过调整定时扫描的时间间隔和每次扫描的限定耗时，可以在不同情况下使得CPU和内存资源达到最优的平衡效果。
  - (expires字典会保存所有设置了过期时间的key的过期时间数据，其中，key是指向键空间中的某个键的指针，value是该键的毫秒精度的UNIX时间戳表示的过期时间。键空间是指该Redis集群中保存的所有键。)

# 消息队列（削峰和异步化）

+ 监听
+ 发送
+ 存储

## celery框架

### 任务类

任务类定义了调用任务的行为（发送消息），也定义了接受消息时的行为。每个任务类必须有独一无二的名字。

### celery任务的6中状态

+ PENDING:表示任务处于等待执行或者未知状态
+ STARTED:表示任务已经开始
+ SUCEESS:表示任务执行成功
+ FAILURE:表示任务执行失败
+ RETRY:表示任务正在被重试
+ REVOKED:表示任务被撤销

celery中的定时任务

### 任务路由

创建两个队列

### 任务工作流

通过创建签名封装任务，

+ signature：创建签名
+ group：并行执行多个任务，返回GroupResult对象
+ link：进行回调
+ chain签名，任务链接
+ chord签名任务完成后调用回调函数
+ map签名：接受一系列任务并返回

### 最佳实践

+ 忽略不需要的结果。
  + 设置ignore_result
+ 避免启动同步子任务。
  + 可以使用回调机制来编排任务调度
+ 合理使用celery的异常处理机制。
  + retry()
+ 任务中不要传入数据库对象作为参数。
+ 添加监控
  + flower
  + 

# 文件上传

# 安全问题

## 中间件安全

+ 跨站点脚本（xss）防护：点击某按钮时，恶意执行脚本
+ 跨站点伪造请求防护（CSRF）：
+ sql注入防护：
+ 点击劫持
+ 访问白名单

## 数据安全

+ 哈希加密
+ https建立安全通信
+ 请求签名



# 认证和访问控制

## RBAC

```python
"""
模型案例 + 中间件模板
"""
# 更丰富的案例
# 资源
class Resource(models.Model):
    name = models.CharField(max_length=255)


# 操作
class Operation(models.Model):
    name = models.CharField(max_length=255)


# 规则
class Rule(models.Model):
    name = models.CharField(max_length=255)
    resource_id = models.IntegerField(max_length=20)
    resource = models.CharField(max_length=255)
    operation_id = models.IntegerField(max_length=255)
    operation = models.CharField(max_length=255)


# 角色
class Role(models):
    name = models.CharField(max_length=255)
    rules = models.ManyToManyField(Rule)


# 用户角色
class UserRole(models.Model):
    user = models.ForeignKey(User)
    role = models.ForeignKey(Role)
    
# 中间件模板
def rbac_authorize(func):
    @wraps(func)
    def _decorator(request, *args, **kwargs):
        # 获取资源、操作、和用户
        resource, operation, user = get_basic_info(request)
        # 获取规则
        rule = get_rule_by_resource_operation(resource, operation)
        # 获取拥有规则的角色
        rule_list_a = get_role_list_by_rule(rule)
        # 获取用户的角色
        rule_list_b = get_role_by_user(user)
        # 如果角色集合有交集，则通过验证，否者拒绝请求
        if have_in_common(role_list_a, role_list_b):
            return func(request, *args, **kwargs)
        else:
            return HttpResponse("Unauthorized", status_code=403)
    return _decorator
```



# 部署

# 负载均衡

# 日志

## logging

+ logger（记录器）
+ handler（处理器）
+ 过滤器（filter）
+ 格式器（formatter）

级别：

+ debug、info、warnning、error、critical

# 监控

# 其他

## 生成csv/pdf



+ 使用csv库,使用reportlab库
+ 响应头中设置内容类型是text/csv(application/pdf)
+ 设置Content-Disposition为attachment：表示以附件形式展示文件
+ 传入响应对象生成writer，65536是单个sheet的上限
+ 写入文件内容
+ 返回响应

## 中间件技术

### 接口访问限流

```python
# 修改REST_FRAMEWOKR配置
'DEFAULT_THROTTLE_CLASSES': (
'rest_framework.throttling.AnonRateThrottle',
'rest_framework.throttling.UserRateThrottle',
),
'DEFAULT_THROTTLE_RATES': {
'anon': '30/min',
'user': '10000/day',
}
# FBV ---> @throttle_classes((...))
# CBV ---> throttle_classes = (...)
```

