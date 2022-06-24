!!! Abstract 大家好~我是`米洛`！<br/>
> 我正在从0到1打造一个开源的接口测试平台, 也在编写一套与之对应的`教程`，希望大家多多支持。<br/>
> 欢迎关注我的公众号`米洛的测开日记`，获取最新文章教程! 

### 回顾

  上一节我们小小地改造了一下`断言模块`，使得JSONPath能够更深入pity心。这一节我们就来聊聊har文件转pity用例的上半部分：har转RequestInfo
  
### Har文件分析

  har文件，相信经常用Chrome浏览器的都知道，浏览器录制的数据可以转换为har文件，来看下面一张图:

![](https://static.pity.fun/picture/20220610212333.png)

  上面是我录制了`掘金`平台的相关接口，我们在任意处鼠标右键，或者点击这个下载按钮，都能够把这些请求保存为har文件。

![](https://static.pity.fun/picture/20220610212437.png)

  我们知道写用例是一件很麻烦的事情，假设一个`链路`有5个接口，假设你在测试一个用户领券买东西的流程，你可能会经历以下步骤:
  
1. 用户登录
2. 用户去领券中心领取100-2的优惠券
3. 用户搜索符合优惠券的商品，并将它加入购物车
4. 用户去购物车结算，生成订单（并使用此优惠券）
5. 用户支付

  可以看到，这个链路其实就不太短，如果我们在不熟悉这几个接口参数的情况下，我们要对照接口文档，去造对应的参数，甚至可能造了一会发现case还是没跑通。
  
  那如果以上场景，你在`浏览器操作`的时候，就能帮你汇总这些接口，并抓取下来，你写用例的时候只需要把`固定的参数`比如账号，商品id等参数化，那会不会非常友好，高效，而且`链路`肯定是通的，因为你在浏览器已经完成过这样的流程了。
  
  所以这就是为什么，我们要支持用例录制的原因，因为雀氏方便，雀氏蟀啊！
  
  好像有点扯远了，我们继续聊这个har的事情。总之har就是chrome tools帮我们录制好我们发出的请求，剩下的筛选，生成用例就得靠我们自己了。
  
  来看看har文件里面的内容吧:

![](https://static.pity.fun/picture/20220610213335.png)

  可以看到，他其实是一个JSON数据，所以我们用JSON库解析它就好了。
  
### XHR和Fetch

  虽然js/css/img等等请求也算http请求，但一般来说与服务端交互的还得是xhr和fetch。fetch可以理解为二代xhr（ajax）。
  
  所以我们只需要获取这2种类型的数据即可。

![](https://static.pity.fun/picture/20220610213635.png)

  而且它把request和response的数据都分开了，我们可以很方便地`读取`。
  
### 编写Convetor

- app/core/request/convertor.py

```python
__author__ = "woody"

from typing import List

from app.core.request.request import RequestInfo

"""
request转换器，支持har到
"""


class Convertor(object):

    @staticmethod
    def convert(file, regex: str = None) -> List[RequestInfo]:
        raise NotImplementedError

```

  内容很简单，并且支持正则，目的是为了过滤不要的url，或者说是为了筛选出想要的接口。
  
  当然这都是`抽象接口`，接着我们来写具体的实现。
  
### 编写HarConvetor

- app/core/request/har_convertor.py

```python
import json
import re
from typing import List

from app.core.request.convertor import Convertor
from app.core.request.request import RequestInfo
from app.excpetions.convert.ConvertException import HarConvertException


class HarConvertor(Convertor):
    @staticmethod
    def convert(file, regex: str = None) -> List[RequestInfo]:
        try:
            with open(file, "r", encoding="utf-8") as f:
                ans = []
                flag = None
                if regex is not None:
                    flag = re.compile(regex)
                # 加载har请求数据
                data = json.load(f)
                entries = data.get("log", {}).get("entries")
                if not entries:
                    raise HarConvertException("entries数据为空")
                for entry in entries:
                    # 如果是fetch或xhr接口，说明是http请求（暂不支持js)
                    if entry.get("_resourceType").lower() in ("fetch", "xhr"):
                        info = RequestInfo()
                        request_data = entry.get("request")
                        info.url = request_data.get("url")
                        if flag is not None and not re.findall(flag, info.url):
                            # 由于不符合预期的url，所以过滤掉
                            continue
                        response_data = entry.get("response")
                        info.body = HarConvertor.get_body(request_data)
                        info.status_code = response_data.get("status")
                        info.request_method = request_data.get("method")
                        info.request_headers = HarConvertor.get_kv(request_data)
                        info.response_headers = HarConvertor.get_kv(response_data)
                        info.cookies = HarConvertor.get_kv(response_data, "cookies")
                        info.request_cookies = HarConvertor.get_kv(request_data, "cookies")
                        info.response_content = response_data.get("content", {}).get("text")
                        ans.append(info)
                return ans
        except HarConvertException as e:
            raise HarConvertException(f"har文件转换异常: {e}")
        except Exception as e:
            raise HarConvertException(f"har文件转换失败: {e}")

    @staticmethod
    def get_kv(request_data: dict, key: str = "headers") -> dict:
        """
        通过response/request获取header信息
        :param key:
        :param request_data:
        :return:
        """
        headers = request_data.get(key)
        result = dict()
        for h in headers:
            result[h.get("name")] = h.get("value")
        return result

    @staticmethod
    def get_body(request_data: dict):
        data = request_data.get("postData", {})
        return data.get("text")


if __name__ == "__main__":
    requests = HarConvertor.convert("./pity.fun.har", "juejin.cn")
    print(requests)

```

  这里主要做了2个事情，读取+转换，首先我们将读取的har利用json.load转为Python数据。
  
  接着去找到对应的请求数组: entries, 遍历之。
  
  接着就是拼接为我们自己的`RequestInfo`了，到这里我们的转换工作就完成了，我们还自定义了HarConvertException, 是为了更好地区别`异常类型`。
  
  这样设计的好处是，以后如果要支持hab, hba, hxr各种格式的时候，我们只需要实现convert方法即可。

  接着就是我们最重要的`转换环节`了，可能需要博主费点脑子了。大家做好心理准备~
  