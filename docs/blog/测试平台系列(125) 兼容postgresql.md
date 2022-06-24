!!! Abstract 大家好~我是`米洛`！<br/>
> 我正在从0到1打造一个开源的接口测试平台, 也在编写一套与之对应的`教程`，希望大家多多支持。<br/>
> 欢迎关注我的公众号`米洛的测开日记`，一起交流学习! 

### 回顾

  上一节我们给自己代码做了一次review，在review的过程中我注意到我们的`数据库`还没有完全支持postgresql，所以今天我们就来试试看，切换底层`数据库`，pity能不能顶得住。
  
### 定义枚举

  如我所说，对于具体的sql_type，我们应该用枚举类给定义起来，所以我们来定义一下枚举:
  
![](https://files.mdnice.com/user/11504/67e51b46-10ad-41f5-83a0-22a4288868b1.png)

  接着我们去替换对应的地方，也就是上一节所说的magic number:
  
![](https://files.mdnice.com/user/11504/4e5cdd7c-1b8c-4b53-ba43-8157c750fccd.png)

  这里如果不满足我们的条件，则直接`抛异常`，提示用户没有这个数据库类型。
  
### 安装pg

  [pg官网](https://www.postgresql.org/download/)
  
  进入下载页，下载对应操作系统的pg即可。傻瓜式安装后，如果没有启动，则需要在安装目录进行如下操作:
  
```bash
// 在bin目录
// 数据路径是pg在磁盘的存储数据路径

pg_ctl -D 数据存储路径 start
```

![](https://files.mdnice.com/user/11504/75364a1b-b8c2-4289-a351-33f3954ae7a0.png)


  启动之后我们可以用DataGrip连接数据库，然后创建`pity`数据库。这一步类似于mysql的
  
```sql
create database pity;
```

![](https://files.mdnice.com/user/11504/23cff4b6-e764-4dec-91a4-aee70faaacba.png)

  一切都是傻瓜式的，嘿嘿。

### 下载异步pg引擎

  经过sqlalchemy官网的指示，我们下载asyncpg引擎:
  
```bash
pip install asyncpg
```

### 修改jdbc

![](https://files.mdnice.com/user/11504/20553d5c-24d1-45e1-bfd9-2bbc1263a24f.png)

  因为我们是临时配置测试一下，所以直接写死了，这些都是我本地的数据库账号密码和端口等数据。
  
  **本次测试并不是要切换到pg，而是给只能选择pg（比如我上家公司就没有mysql）的人一个选择，顺便也看看pity能不能无缝切换到pg。**
  
### 重新建表

  创建好数据库以后，我们第一步就是要测试数据表是否能够建立。我们启动一下pity试试。

![](https://files.mdnice.com/user/11504/1d10ede8-584d-43b6-af0c-3f0df73312bb.png)

  一上来就给咱一个猛的报错，可以看到，我们的ddl里面包含的是datetime，但pg里面并没有`datetime`，确实，我之前使用的时候也都用的`timestamp`。
  
  所以这意味着，如果我们要兼容pg，则需要把datetime类型全部换成TIMESTAMP，换是可以换，但我就怕我们现在的代码会受影响。
  
  这里我已经批量替换过了，发现sql里面反射出来的依旧是datetime类型（Python的datetime），那就说明代码没啥问题。
  
  ![](https://files.mdnice.com/user/11504/97f5d565-f7e7-41f5-b556-28b3f99905c2.png)
  
  **pg也没有tinytext和longtext，所以我们需要统一改成text兼容pg。**
  
  总体来说改动并不大，这就是我们用sqlalchemy（orm）的好处。
  
---

  来看看改造后，是否好用：

![](https://files.mdnice.com/user/11504/b2f0c5dd-4e06-4a67-82cf-50bbe9cef05e.png)

  可以看到，数据表基本都建好了，但是少了一只：`apscheduler`。忘了这个硬茬了！！！
  
### 适配apscheduler

![](https://files.mdnice.com/user/11504/cd5e567b-29af-4583-842f-a900243c2745.png)

  总体来说，由于apscheduler的存在，我们还是得弄一套`同步session`，至于pg的同步session，我们需要安装psycopg2:
  
```bash
pip install psycopg2
```

![](https://files.mdnice.com/user/11504/930a6a6d-10c9-4e5f-a819-f1925c2a8a25.png)

  再次启动服务，发现表格建好了！
  
### 试试pity吧

#### 问题一: deleted_at

  出师不太利，一上来就报了个deleted_at的错，原来我有的deleted_at的字段定义的是timestamp，但实际上我查询的时候用的是否等于0。
  
  
  所以我统一把deleted_at改成bigint了。

#### 问题二: date_format
  
  接着又遇到了一个date_format的错，我们试着解决下吧：
  
![](https://files.mdnice.com/user/11504/09371da1-cb9b-4f9c-8b1c-d315bffdd277.png)

  对于mysql这句话是没毛病的，格式化日期，但是pg不太支持呀，所以我们得换通用点的方式。
  
![](https://files.mdnice.com/user/11504/45f2dcc8-7642-4fbd-bcd5-3ccea85e03bb.png)

  这里我们就不采用sql的func了，直接用Python的方法，这样都兼容，按照图里这么改。改好了之后mysql继续测试一下，发现没毛病。
  
#### 问题三: timestamp不支持直接比较字符串

  我们之前比较日期，用between或者大于小于的时候，传入字符串的时间格式都莫得问题，但是换pg之后她就`傲娇`了。原因是，它不能把text直接转成timestamp，所以我们得手动转一下，MySQL倒是2种都支持。

![](https://files.mdnice.com/user/11504/d9b85874-ea74-4fdf-96df-f26ef18a31b7.png)

### 问题四: pg的数据传入id字段，哪怕为None insert的时候会把id置为0

  为了解决这个问题，我们就不赋予id属性了：
  
![](https://files.mdnice.com/user/11504/273d72db-4dc1-4c6e-96b2-ff301ea0eb36.png)

  也许跟用的异步pg引擎有关，我试了下这个也不影响MySQL，所以就迁就下吧。
  
---

  以上就是我总结的，数据库切换引发的一系列的问题，希望对大家`有所帮助`。
  
> 我是米洛，一直陪伴各位学习！免费的`小黄心`，帮我点一个吧！
>
> 项目地址: [https://github.com/wuranxu/pity](https://github.com/wuranxu/pity)
  
  
  