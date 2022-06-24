> 大家好~我是`米洛`！<br/>
> 我正在从0到1打造一个开源的接口测试平台, 也在编写一套与之对应的`教程`，希望大家多多支持。<br/>
> 欢迎关注我的公众号`米洛的测开日记`，获取最新文章教程! 

### 回顾

  上一节我们介绍了`mitmproxy`，这节我们就来轻松愉快地实现一个篡改`HTTP返回`的功能。
  
#### 安装证书

  正好我的电脑`重装了系统`，所以证书嘛的，我也得重新安装一遍，顺便记录下整个过程。
  
  首先我们开启mitmproxy.exe，接着打开浏览器: 输入mitm.it
  
![](https://static.pity.fun/picture/20220608003202.png)

  他提示我们`没有通过代理`访问这个网站，所以我们需要配置下自己的代理服务器（Mac的用户还请自己百度一下，目前我的机器是Windows）
  
  按住Windows键，搜索`代理`。
  
![](https://static.pity.fun/picture/20220608003313.png)

  进入代理设置页面，输入127.0.0.1+8080端口，接着保存即可。

![](https://static.pity.fun/picture/20220608003424.png)

  再次访问mitm.it页面:
  
![](https://static.pity.fun/picture/20220608003440.png)

  页面发生了变化，我们直接下载Windows下的证书，下载完成后打开，一直点下一步即可。
  
![](https://static.pity.fun/picture/20220608003554.png)

![](https://static.pity.fun/picture/20220608003609.png)

  最后点击确定即可。
  
  其实安装证书主要是为了让`https`接口能够正常运转，不然我们录制到https接口的时候会提示我们经过了代理，从而`被拦截`。
  
#### 打开网站测试一下

![](https://static.pity.fun/picture/20220608003826.png)

  我们打开`掘金`，可以看到有很多个录制到的接口，而且页面基本上也是无感知的，不会报啥`安全问题`。
  
  （大家可以多开几个网站试试看，看会不会有什么问题）
  
#### 篡改百度首页

  我们今天的任务是写一个简单的`mock脚本`，让大家感受一下mitmproxy的强大。
  
  当用户访问`https://www.baidu.com`的时候，给他返回`百度一下，你就知道`。
  
  话不多说，开造。
  
- 编写一个最基础的脚本

  我们需要先编写一个baidu.py，接着写入以下内容:
  
```python
from mitmproxy import http


class Baidu(object):

    def request(self, flow):
        if flow.request.pretty_url.startswith("https://www.baidu.com"):
            flow.response = http.Response.make(
                200,  # (optional) status code
                "百度一下，你就知道",  # (optional) content
                {"Content-Type": "application/json"}  # (optional) headers
            )

addons = [
    Baidu()
]
```

  这里我们定义了一个百度类，接着在每次request请求进来的时候，我们判断他的url是不是baidu.com，是的话，我们就给他设置一下response，其中`HTTP状态码`为200，返回内容为`百度一下，你就知道`。
  
  接着我们把这个类加入addons数组，并在mitmproxy.exe启动的时候挂载之。
  
- 挂载addons

  执行一下命令（需要把mitmpdump.exe放入环境变量, 也可以pip全局安装mitmproxy）
  
```BASH
mitmdump -s baidu.py
```

  接着我们挂上8080的代理，开始访问百度，这时候就应该展现gif图了！
  
![](https://static.pity.fun/picture/%E5%8A%A8%E7%94%BB.gif)

  这样我们就完成了一个简单的`篡改请求`，虽然这不是用例录制相关的内容，但我还是想展示一下mitmproxy的强大。
  
---

  今天的内容就讲到这里，下一节我们继续介绍用例生成相关内容。