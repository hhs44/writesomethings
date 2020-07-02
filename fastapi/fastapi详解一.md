# fastapi入门学习（一）



FastAPI框架是一个高性能，易于学习，高效编码，生产可用的python，web开发框架。

但对于刚开始学习python不久的同学来说仅仅从官文学习，有一定的难度，并咩有那么易学。

本篇主要从一个车辆违章查询的项目，以项目来驱动fastapi的学习。

项目实现效果如下：

查询页面展示：

![image-20200701200412814](https://i.loli.net/2020/07/01/x7ySB1hqZTn2U8C.png)

接口swagger ui界面：

![image-20200701200445894](https://i.loli.net/2020/07/01/SU2o63MtXNkJCEW.png)

### fastapi的安装和简单案例

fastapi框架的安装

```python
pip install fastapi
```

此外还需要一个能让项目跑起来的服务器，并且需要的是一个ASGI服务器

```python
pip install uvicorn
```

### 最简单的fastapi只需要一个文件

创建main.py：这里使用官方代码：

```python
from typing import Optional

from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root():
    return {"Hello": "World"}


@app.get("/items/{item_id}")
def read_item(item_id: int, q: Optional[str] = None):
    return {"item_id": item_id, "q": q}
```

保存好后在命令行运行服务器：

```
uvicorn main:app --reload
```

访问路径 http://127.0.0.1:8000/docs/，就能看到你的第一个接口的接口文档，如下

![image-20200701202452760](https://i.loli.net/2020/07/01/gsBI6yUh4mjzRfl.png)

### 车辆查询项目的项目结构

本人也是对fastapi只是入门，对结构的理解暂时如下，该结构并不是一个完整web项目的需求。以后有机会再深入研究。

![image-20200701202647366](https://i.loli.net/2020/07/01/soqQpX2JkVczlCr.png)

一一介绍每个部分：

+ curd ：数据库的操作，使用sqlalchmey完成，curd即是创建（create），更新（update），读取（read），删除（delete）。
+ database：完成对数据库的连接。
+ main：路由函数的位置，可以和flask类比。
+ models：对数据库表结构类化，和curd，database部分的实现主要参考sqlalchmey。
+ pagnation：自定义的分页器，完成查询后数据的分页。
+ schemas：规范约束，响应的或者查询到数据。



### 完成数据库连接和模型构建部分

这里使用的是mysql，并且使用mysqlclient库，如果使用的pymysql可参见第二种写法，这里创建了database.py文件。

```python

from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
# 使用mysqlclient的情况
SQLALCHEMY_DATABASE_URL = "mysql://username:password@localhost:3306/数据库名?charset=utf8"
# 使用pymysql
# SQLALCHEMY_DATABASE_URL = "mysql+pymysql://username:password@localhost:3306/数据库名?charset=utf8"
from sqlalchemy.orm import sessionmaker

engine = create_engine(SQLALCHEMY_DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

Base = declarative_base()

```

创建models.py，该项目中主要使用了record和car两个表，record关联到car的id上。

实现如下：

```python
from sqlalchemy import Boolean, Column, ForeignKey, Integer, String, Date, MetaData
from sqlalchemy.orm import relationship

from database import Base, engine

metadata = MetaData(engine)


class Car(Base):
    __tablename__ = "car"

    id = Column(Integer, primary_key=True, index=True, autoincrement=True)
    carno = Column(String(15), index=True, )
    owner = Column(String(20), index=True, )
    brand = Column(String(20))
# 这里的配置存在疑惑，也是初次使用sqlalchemy。
    records = relationship('Record', back_populates='car')


class Record(Base):
    __tablename__ = "record"

    id = Column(Integer, primary_key=True, autoincrement=True)
    reason = Column(String(255))
    makedate = Column(Date)
    punish = Column(String(255))
    dealt = Column(Boolean, default=0)
    car_id = Column(Integer, ForeignKey('tb_car.id'))

    car = relationship("Car", back_populates="records")

```

### 创