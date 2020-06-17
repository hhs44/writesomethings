# 如何开始一个django项目

使用django前的准备

+ 安装python环境，这里使用的python环境是python3.7.7

+ 使用pip安装django，这里直接使用最新版本django3

本次的django项目的最终目标是看见让人满意的小火箭。如图下：

![image-20200525082253714](https://i.loli.net/2020/05/25/UXEmLCrMuBI4GYs.png)

关于部署，暂时以django自带的wsgi进行本地部署。

## 生成一个django项目

命令：

```toml
# 查看django版本
python -m django --version
# 创建Django项目
django-admin startproject demo
# 创建django的第一个应用
python manage.py startapp first
# 启动django项目
python manage.py runserver （可选 host：port）
```

使用以上命令就能开启一个默认版本的django项目，这里访问的页面 是django自带的一个小火煎页面，但目前没有进行其他配置，看到的页面是全英文，并且时区也不匹配。

注：这里没有对新建的first应用进行注册使用。

### 修改配置文件

配置问及那，在项目名下的settings.py中进行设置，主要修改的项目包含时区和语言分别对应文件中的字段是LANGUAGE_CODE和TIME_ZONE，分别修改为`zh-hans`和  'Asia/shanghai'

## 使mysql数据库运行起来

需要安装mysqlclient，如果使用pymysql会因为pymysql版本过旧无法使用，在修改版本判断文件后才能支持mysql数据库的操作。（mysqlclient是pymysql的升级版）