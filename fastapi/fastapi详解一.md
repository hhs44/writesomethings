# fastapi入门学习（一）

![image-20200702200828276](https://i.loli.net/2020/07/02/Np1dL9W36kqBzQJ.png)

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

### curd操作

在该案例中对数据库主要有查询，更改和删除。查询条件包含时间范围，和对输入框的内容在车牌和车主姓名里进行查询。更改操作，完成对数据受理信息的更改，删除操作只是删除record记录，car表的信息不做改变。（在实际案例中，删除操作是对删除信息字段进行改变）。

下面代码中实现了五个接口：

```python
from typing import List, Any

from pydantic.schema import date
from sqlalchemy import and_, or_
from sqlalchemy.orm import Session

import pagnation
from models import Car, Record

PAGE_SIZE = 5

# 查询所有车辆信息
def read_car(db: Session):
    """

    :param db:
    :return:
    """
    cars: List[Any] = db.query(Car).all()
    return cars

# 查询所有记录信息，不包含车辆信息
def read_search_record(db: Session):
    """
    :param db:
    :return:
    """
    return db.query(Record).all()

# 按照条件进行查询
def read_all_record(db: Session, carno: str = '', start: date = None, end: date = None, page: int = 1):
    """
    :param db:
    :param carno:
    :param start:
    :param end:
    :param page:
    :return:
    """
    url = "/api/record/?"
    records = db.query(Record)
    if carno:
        records = records.join(Car).filter(or_(
            Car.carno.like("%" + carno + "%"),
            Car.owner.like("%" + carno + "%")))
        url += f"carno={carno}&"
    if all([start, end]):
        records = records.filter(and_(Record.makedate <= end, Record.makedate >= start))
        url += f"start={start}&end={end}"
    count = (len(records.all()) + PAGE_SIZE - 1) / PAGE_SIZE
    if page:
        records = records.limit(PAGE_SIZE).offset((page - 1) * PAGE_SIZE)
    records = records.all()
    return pagnation.pagenation(records, url, page, count)

# 更新record中的 dealt字段为True
def update_dealt(db: Session, record_id: int):
    """
    :param db:
    :param record_id:
    :return:
    """
    try:
        record = db.query(Record).get(record_id)
        record.dealt = True
        db.add(record)
        db.commit()
        return {'code': 10000, 'msg': "Success update car's record "}
    except Exception as e:
        print(e)
    return {'code': 10001, 'msg': " Fail update car's record "}

# 删除一条已受理的record记录
def del_record(db: Session, record_id: int):
    """
    :param db:
    :param record_id:
    :return:
    """
    try:
        record = db.query(Record).get(record_id)
        if record.dealt:
            db.delete(record)
            db.commit()
            return {'code': 10000, 'msg': " Success del car's record "}
    except Exception as e:
        print(e)
    return {'code': 10001, 'msg': " Fail del car's record "}
```

curd操作完成后，必不可少，会考虑到分页的需求。实现思路主要是，路由参数参数传递中添加page字段，传出参数中需要以下5个字段：

+ count：总共页数
+ currpage：当前页码
+ nexturl：下一页接口
+ preurl：前一页接口
+ record：查询的记录数据

分页器实现代码

```
def pagenation(record, url, currage, count: int):
    nextnum = int(count // 1) if (currage + 1) >= count else currage + 1
    prenum = 1 if (currage - 1) == 0 else currage - 1
    nexturl = url + f"page={nextnum}"
    preurl = url + f"page={prenum}"
    if currage == 0:
        currage = 1
    if currage >= count:
        currage = count
    latest = {
        'count': count,
        'currpage': currage,
        'nexturl': nexturl,
        'preurl': preurl,
        "records": record,
    }
    return latest

```

### schemas模块的实现

这个对于初学着看着有点摸不着头脑。这里简单描述：该部分完成的是对于路由函数（main.py里的函数）的接受和返回的数据进行规范。

对下面代码进行简单描述：

+ CarBase包含了从car中需要查询到的字段，并规定了每个字段的类型
+ Car在CarBase基础上添加了orm_mode,可以实现orm模型

```python
from typing import List

from pydantic import BaseModel
from pydantic.schema import date


class CarBase(BaseModel):
    carno: str
    owner: str


class Car(CarBase):
    class Config:
        orm_mode = True


class RecordBase(BaseModel):
    reason: str
    makedate: date
    punish: str
    dealt: bool

# 这里car实现外键数据的Car化
class Record(RecordBase):
    id: int
    car: Car

    class Config:
        orm_mode = True

# latest是查询后接口返回的数据格式，recods这里获取到的数据是一个列表
class Latest(BaseModel):
    count: int
    currpage: int
    nexturl: str
    preurl: str
    records: List[Record]

    class Config:
        orm_mode = True


class Dealt(BaseModel):
    code: int
    msg: str
```

这里定义完成后的调用的关键字字段是response_model。

### 最终的mian.py

以上的工作完成的都是fastapi的周边工作，fastapi主要完成的是一个接口的实现，接口文档的自动生成（生成风格是swagger UI），在该文档中还可以实现接口的测试。

主要实现的步骤如下：

+ 定义Fsatapi对象

+ 在对象中以get，post，delete，patch +路由的形式定义接口

  例子：get（url ，name=“文档中的名字”，description=“具体的描述”，response_model=“schemas模块中的定义”）

```python
from fastapi import FastAPI, Depends
from pydantic.schema import date

# 使用sqlalchemy对数据库进行crud操作
from sqlalchemy.orm import Session

# 导入支持访问静态文件的相关包
from starlette.requests import Request
from starlette.responses import FileResponse
from starlette.staticfiles import StaticFiles

import crud
import schemas
from database import SessionLocal


def get_db():
    """
    获取事务
    """
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()


app = FastAPI()

app.mount("/static", StaticFiles(directory="static"))


@app.get("/")
async def index(request: Request):
    """

    :param request:
    :return:
    """
    return FileResponse('./static/index.html')


@app.get("/api/car", name="获取车辆信息", description="获取所有车辆信息")
async def get_car(db: Session = Depends(get_db)):
    """

    :param db:
    :return:
    """
    car = crud.read_car(db)
    return car


@app.get("/api/record/", name="获取所有违章记录", description="获取所有违章记录", response_model=schemas.Latest)
async def get_all_record(page: int = 1, carno: str = None, start: date = None, end: date = None,
                         db: Session = Depends(get_db)):
    """
    :param page:
    :param carno:
    :param start:
    :param end:
    :param db:
    :return:
    """
    latest = crud.read_all_record(carno=carno, start=start, end=end, page=page, db=db)
    return latest


@app.patch("/api/record/{record_id}", name="update car record", description="处理违章记录", response_model=schemas.Dealt)
async def dealt_record(record_id: int, db: Session = Depends(get_db)):
    """
    :param record_id:
    :param db:
    :return:
    """
    dealt = crud.update_dealt(db=db, record_id=record_id)
    return dealt


@app.delete("/api/record/{record_id}", name="del car record", description="删除违章记录", response_model=schemas.Dealt)
async def dealt_record(record_id: int, db: Session = Depends(get_db)):
    """
    :param record_id:
    :param db:
    :return:
    """
    dealt = crud.del_record(db=db, record_id=record_id)
    return dealt


if __name__ == '__main__':
    import uvicorn

    # uvicorn main: app --reload
    uvicorn.run(app, host='127.0.0.1', port=8000)
```

### 想看到页面的必经之路

![image-20200702195733568](https://i.loli.net/2020/07/02/ZnK54EYtmGTx3gN.png)

fastapi本身是做接口的，就像数据操作需要sqlalchemy来实现，这里实现使用到的库是starlette==0.13.4。

接口的页面的静态文件在我的仓库里，后端开发不需要为自己写前端文件啦。

项目源代码在[这里](https://github.com/hhs44/fastapi_example)。

