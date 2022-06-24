!!! Abstract 大家好~我是`米洛`！<br/>
> 我正在从0到1打造一个开源的接口测试平台, 也在编写一套与之对应的`教程`，希望大家多多支持。<br/>
> 欢迎关注我的公众号`米洛的测开日记`，一起交流学习! 

### 回顾

  上一节我们`review`了一下代码，这一节我们来进入正轨，聊聊sqlalchemy彻底`异步化`。
  
### 历史问题

  由于`pity`支持了在线编写/执行sql语句的功能，所以为了更加友好，我们特意提供了读取数据库表/字段的功能，看看下图:
  
![](https://files.mdnice.com/user/11504/537f864f-5864-4e0a-8b71-ac24e6b90075.png)

  这样做的目的，并不是`炫技`，而是给大家一个更舒适的编写sql的环境，如果写sql的时候，数据库表和字段都给你`联想`了，那写起来就更轻松了。
  
  但这么个功能呢~却不是那么好做，因为据我所知，这个功能需要用到Metadata，接着利用其的反射方法，获取到对应的`表`和`字段`信息。而异步engine又无法获取metadata相关数据，包括`自动建表`。
  
  为此我一直保持着`同步session`和`异步session`混用的状态，当然这肯定不是很好的方案。
  
  于是我今天突发奇想，还真被我找到了答案。
  
### 解决问题

  具体issue: [https://github.com/sqlalchemy/sqlalchemy/issues/6121](https://github.com/sqlalchemy/sqlalchemy/issues/6121)
  
  
![](https://files.mdnice.com/user/11504/a32ff6ae-12bf-4cb9-a4c9-94741aa1e022.png)

  看看他的思路，其实很简单，`异步engine`提供了run_sync方法，这个很厉害，也就是说可以直接用异步engine去执行同步engine的方法，针对create_all（建表）+获取表信息，就很有用了，我们直接`学`过来，并给他一个小`红心`。
  
- 修改create_all

![](https://files.mdnice.com/user/11504/25edc36c-b291-4977-ad5e-a05454f54875.png)

  在底部定义create_table方法，由于是异步的，所以我们不能直接运行，但我们可以考虑放入startup里面，随着系统启动而执行。
  
- 修改main.py

![](https://files.mdnice.com/user/11504/e5d9a4fb-e462-43be-82d8-4411b999b983.png)

- 修改获取表的方法

  先编写load_table方法，这里面由于需要`展示树`，所以多了很多额外的变量，核心的部分就是run_sync
  
![](https://files.mdnice.com/user/11504/973bb935-16b4-4121-9dd2-dafdb47f7bb1.png)

- 修改调用的部分

![](https://files.mdnice.com/user/11504/53afc48b-8d91-4eee-ad49-06a595cd4803.png)

  其实是把原先的逻辑，放入了一部分到table里面，好处是可以只遍历1次哦！
  
### 放心去掉mysqlconnector

![](https://files.mdnice.com/user/11504/9ad3defa-f5f1-4541-be81-08e2fc9cb9d7.png)


![](https://files.mdnice.com/user/11504/6003e17d-93e1-41e2-8601-017af27c2f0b.png)

  这里引擎记得改为pymysql，实在没办法，我突然想到apscheduler不支持异步session，所以这个同步session还得继续用，但我们可以只用pymysql作为驱动了（支持异步和同步），所以我们把jdbc的驱动改为pymysql。
  
  这样来看的话，基本也是做了一个完整的清理工作，说实话，速度还可以，有点小起飞。
  
### 忍不住压测了一下

  找了个查询网关地址的接口，其中包括了`鉴权`，`db`操作，20个线程可以到300rps，还算满意，服务也不卡，对于一个`测试平台`来说，这样也就`够了`。
  
  无法避免的是，我花了1个多小时，把所有同步session的操作都换成了异步。
  
![](https://files.mdnice.com/user/11504/97b7f506-42f8-4656-b797-8d3ccd1c0cb5.png)

> 我是米洛，一直陪伴各位学习！免费的`小黄心`，帮我点一个吧！
>
> 项目地址: [https://github.com/wuranxu/pity](https://github.com/wuranxu/pity)