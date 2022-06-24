> 大家好~我是`米洛`！<br/>
> 我正在从0到1打造一个开源的接口测试平台, 也在编写一套与之对应的`教程`，希望大家多多支持。<br/>
> 欢迎关注我的公众号`米洛的测开日记`，获取最新文章教程! 

### 回顾

  上一节我们编写了`har`文件转RequestInfo相关的内容，但就在进行`下一步`的时候，我发现我们的前置条件还`并不支持`http请求，这就导致我们只能把拆出来的`多个接口`转为`多个用例`，这显然是不好的做法。
  
  所以这一节我们需要让前置条件支持`http请求`，并稍微做出一些样式上的调整。
  
### 前端部分

  我们这次不同于以往，由于http请求其实也是前置条件的`一种`，所以理论上来说，后端的增删改查接口都是有的。
  
  我们先改造下对应的页面，再到后端支持`Http执行器`即可。
  
  我们先来看看现有的前后置条件:

![](https://static.pity.fun/picture/20220611165401.png)

  可以看到这个页面确实不太好看，为啥呢，我感觉主要是这样:
  
- 有些表单不需要那么长的输入框，但是却独占了一行，导致整个页面很松散

  所以我们需要进行一定的整合，我们来看看整合后的:

![](https://static.pity.fun/picture/20220611170029.png)

  不能说好看了，但是起码一个页面内能够放下这些内容了，少了一个滚动条。

  其实我做的就是，把有些字段不需要那么多输入框的，给合并到一行显示了，这样起码不会那么`占高度`了。
  
  接着来看看SQL前置的对比:

- 改造前

![](https://static.pity.fun/picture/20220611170209.png)

- 改造后

![](https://static.pity.fun/picture/20220611170224.png)

  首先是左侧的菜单`拉高`了，感官上会比前者好一点。
  
  最后我们来看看新版的http请求组件，由于其他地方也有用到，所以我稍微复用了一下（总体来说组件设计的不是太好，勉勉强强改造了，后续肯定需要重新设计一番的）

![](https://static.pity.fun/picture/20220611170412.png)

  前端上面的改造就介绍到这里，其实也就是复用了一下http相关的组件，我们来看看`后端部分`。
  
### 后端部分

  后端部分，我们只需要补充对应的Constructor即可。
  
  新增app/core/constructor/http_constructor.py
  
```python
import json

from app.core.constructor.constructor import ConstructorAbstract
from app.crud.config.AddressDao import PityGatewayDao
from app.middleware.AsyncHttpClient import AsyncRequest
from app.models.constructor import Constructor


class HttpConstructor(ConstructorAbstract):

    @staticmethod
    async def run(executor, env, index, path, params, req_params, constructor: Constructor, **kwargs):
        try:
            executor.append(f"当前路径: {path}, 第{index + 1}条{HttpConstructor.get_name(constructor)}")
            data = json.loads(constructor.constructor_json)
            url = data.get("url")
            if data.get("base_path"):
                base_path = await PityGatewayDao.query_gateway(env, data.get("base_path"))
                url = f"{base_path}{url}"
            headers = json.loads(data.get("headers"))
            client = await AsyncRequest.client(url=url, body_type=data.get("body_type"), headers=headers,
                                               body=data.get("body"))
            resp = await client.invoke(data.get("request_method"))
            executor.append(f"当前{ConstructorAbstract.get_name(constructor)}类型为http, url: {url}")
            if constructor.value:
                params[constructor.value] = resp
            executor.append(
                f"当前{ConstructorAbstract.get_name(constructor)}返回变量: {constructor.value}\n返回值:\n {resp}\n")
        except Exception as e:
            raise Exception(
                f"{path}->{constructor.name} 第{index + 1}个{HttpConstructor.get_name(constructor)}执行失败: {e}")

```

  我们继承自ConstructorAbstract类，并实现里面的run方法。通过反序列化constructor_json获取http请求的相关数据:

- url
- request_method
- headers
- base_path
- body

  获取到了之后就可以发送http请求了，最后把请求结果返回写入params字典，和其他类型的数据构造器`差不多`。

![](https://static.pity.fun/picture/20220611192603.png)

  我们在获取数据构造器这里加上http类型，接着就可以嚣张地等待奇迹出现了:

![](https://static.pity.fun/picture/20220611192822.png)

  可以看到，http请求已经被顺利获取并完成了，至此，除了mq相关的前后置条件，其他内容我们基本上都已经完成了。
  
  下一节我们还是的将思绪拉回到`如何提取接口间的依赖数据`。
