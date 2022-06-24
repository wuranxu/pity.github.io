!!! Abstract 大家好~我是`米洛`！<br/>
> 我正在从0到1打造一个开源的接口测试平台, 也在编写一套与之对应的`教程`，希望大家多多支持。<br/>
> 欢迎关注我的公众号`米洛的测开日记`，获取最新文章教程! 

### 回顾

  上一节我们聊了grpc相关的实现思路，但也只是提供了思路和`对应的库`，但读者们需要时间消化，所以我们来转换一下思维，来聊聊日志这块内容。
  
### 现有日志

  虽然现有的`日志`已经足够咱们使用了，但随着时间的推移，日志的切割，这些都得考虑进来，否则你将拥有一个好几百MB的日志文件。考虑到企业内部很多都是基于es、cat等日志系统，logbook用起来就有点底气不足了。
  
  **其实也不算吧，主要是觉得loguru这玩意挺不错的，彩色的东西是我的最爱，我觉得比单调的控制台好不少。**
  
  虽然logbook差不多能满足我的需求，但我还是想研究下loguru。
  
### loguru

  [github地址](https://github.com/Delgan/loguru): https://github.com/Delgan/loguru

  loguru是一款非常便捷的日志管理工具，里面自带了`切割日志`，`彩色输出`等一系列操作，但人家api相对比较友好，不需要像`logging`那样配置很多很多内容，而且本身是线程安全/进程安全的（当然这是它自己说的）。下面我们就来看一看它的基本用法，用一张官网的gif:
  
![](https://static.pity.fun/picture/2022-2-7/1644221384548-demo.gif)

  这个gif演示的demo非常丰富，有打印到控制台的，有输出到log文件的，还有日志分片和格式以及异常等内容，但是确实只写了`极少量`的代码。

  老实说这也是我被他吸引到的地方，我也亲自体验了一下，感觉还不错。那我们该如何进行改造呢？从哪里入手呢？
  
  我会把本文分为2个内容，第一是替换fastapi自带的日志句柄，第二则是用到我们自身的日志板块中。
  
### 替换fastapi默认的输出

  在windows下，如果我们不采用loguru，可以看到日志的输出大概是这样的:
  
![](https://static.pity.fun/picture/2022-2-7/1644223125428-image.png)

  非常朴实无华，和蔼可亲。
  
  再来看看改造后的效果:
  
![](https://static.pity.fun/picture/2022-2-7/1644223221572-image.png)

  多的不说，就一个`时间`就已经很有作用了，有时候你不知道服务啥时候启动的，我们就可以通过日志的时间来判断。更何况这个日志输出如此美腻呢~
  
  话不多说，甘蔗！
  
#### 在app/__init__.py新增以下内容

```python
class InterceptHandler(logging.Handler):
    """
    Default handler from examples in loguru documentaion.
    See https://loguru.readthedocs.io/en/stable/overview.html#entirely-compatible-with-standard-logging
    """

    def emit(self, record: logging.LogRecord):
        # Get corresponding Loguru level if it exists
        try:
            level = logger.level(record.levelname).name
        except ValueError:
            level = record.levelno

        # Find caller from where originated the logged message
        frame, depth = logging.currentframe(), 2
        while frame.f_code.co_filename == logging.__file__:
            frame = frame.f_back
            depth += 1

        logger.opt(depth=depth, exception=record.exc_info).log(
            level, record.getMessage()
        )


def format_record(record: dict) -> str:
    """
    这里的代码是copy的，记录日志格式的
    Custom format for loguru loggers.
    Uses pformat for log any data like request/response body during debug.
    Works with logging if loguru handler it.
    Example:
    # >>> payload = [{"users":[{"name": "Nick", "age": 87, "is_active": True}, {"name": "Alex", "age": 27, "is_active": True}], "count": 2}]
    # >>> logger.bind(payload=).debug("users payload")
    # >>> [   {   'count': 2,
    # >>>         'users': [   {'age': 87, 'is_active': True, 'name': 'Nick'},
    # >>>                      {'age': 27, 'is_active': True, 'name': 'Alex'}]}]
    """

    format_string = LOGURU_FORMAT
    if record["extra"].get("payload") is not None:
        record["extra"]["payload"] = pformat(
            record["extra"]["payload"], indent=4, compact=True, width=88
        )
        format_string += "\n<level>{extra[payload]}</level>"

    format_string += "{exception}\n"
    return format_string


def make_filter(name):
    # 过滤操作，当日志要选择对应的日志文件的时候，通过filter进行筛选
    def filter_(record):
        return record["extra"].get("name") == name

    return filter_


def init_logging():
    loggers = (
        logging.getLogger(name)
        for name in logging.root.manager.loggerDict
        if name.startswith("uvicorn.")
    )
    for uvicorn_logger in loggers:
        uvicorn_logger.handlers = []

    # 这里的操作是为了改变uvicorn默认的logger，使之采用loguru的logger
    # change handler for default uvicorn logger
    intercept_handler = InterceptHandler()
    logging.getLogger("uvicorn").handlers = [intercept_handler]
    # set logs output, level and format
    # logger.add(sys.stdout, level=logging.DEBUG, format=format_record, filter=make_filter('stdout'))
    # 为pity添加一个info log文件，主要记录debug和info级别的日志
    pity_info = os.path.join(Config.LOG_DIR, "pity_info.log")
    # 为pity添加一个error log文件，主要记录warning和error级别的日志
    pity_error = os.path.join(Config.LOG_DIR, "pity_error.log")
    logger.add(pity_info, enqueue=True, rotation="20 MB", level="DEBUG", filter=make_filter("pity_info"))

    logger.add(pity_error, enqueue=True, rotation="10 MB", level="WARNING", filter=make_filter("pity_error"))

    # 配置loguru的日志句柄，sink代表输出的目标
    logger.configure(
        handlers=[
            {"sink": sys.stdout, "level": logging.DEBUG, "format": format_record},
            {"sink": pity_info, "level": logging.INFO, "format": format_record, "filter": make_filter("pity_info")},
            {"sink": pity_error, "level": logging.WARNING, "format": format_record, "filter": make_filter("pity_error")}
        ]
    )
    return logger
```

  这里的代码看似很复杂，实际上`InterceptHandler`这个类是用于定义uvicorn输出的类，可以不用细看，因为代码我也是copy的（其实里面大部分都是注释）。
  
  format_record则是控制日志输出格式的，包括时间，文件名，以及传入字典的时候对字典做了一个额外处理。
  
  我们重点看init_logging方法，这个里面前半部分是过滤出uvicorn的默认日志句柄，接着替换。后面是新增了pity的2种业务日志（info和error），一共3个handler。
  
  最后配置logger的输出，分别输出到sys.stdout（控制台）和pity_info.log以及pity_error.log。
  
  （pity_error和info里面都有个filter自带，这个是为了配合make_filter用的，因为日志句柄很多，而logger又是一个全局对象，所以日志到底要`输出`到`哪个日志文件`，就要通过filter来过滤了。）
  
### 改写main.py

![](https://static.pity.fun/picture/2022-2-7/1644223820997-image.png)

  在main.py里面引入init_logging即可。接着重启服务，控制台就变成了上面的样子。
  
#### 新增参数获取

  有时候我们想获取到请求的json参数或者form参数，以便于报错的时候方便找到历史的请求数据，所以我们可以写一个中间件:
  
```python
async def request_info(request: Request):
    logger.bind(name=None).info(f"{request.method} {request.url}")
    try:
        body = await request.json()
        logger.bind(payload=body, name=None).debug("request_json: ")
    except:
        body = await request.body()
        if len(body) != 0:
            # 有请求体，记录日志
            logger.bind(payload=body, name=None).debug(body)


pity.include_router(user.router)
pity.include_router(project.router, dependencies=[Depends(request_info)])
pity.include_router(http.router, dependencies=[Depends(request_info)])
pity.include_router(testcase_router, dependencies=[Depends(request_info)])
pity.include_router(config_router, dependencies=[Depends(request_info)])
pity.include_router(online_router, dependencies=[Depends(request_info)])
pity.include_router(oss_router, dependencies=[Depends(request_info)])
pity.include_router(operation_router, dependencies=[Depends(request_info)])
pity.include_router(msg_router, dependencies=[Depends(request_info)])
```

  这里我过滤出了name=None（即stdout）的句柄，并打印出请求方法和url。接着判断是不是json数据，如果不是则判断有没有body，如果有json或者有body，则打印出来。
  
  接着我们在每个router都加入这个依赖的中间件（request_info），用户路由没有加是因为`用户登录的时候会传入明文的账号密码（js加密这里还没做 暂时也不想做）`，所以如果被有心之人发现的话就难受了，所以我直接去掉了。
  
#### 测试一下请求参数

  因为await request.json()会返回一个Python里面的对象（大概率是dict），所以打印会按照字典的方式打印出来，大家这里需要理解一下。
  
![](https://static.pity.fun/picture/2022-2-7/1644224132458-image.png)

  这个功能就很`nice`了！！！但缺陷就是目前是打印的字典，如果能打印json就更好了，这个读者有兴趣可以自己去实现。
  
### 改造原本的Log类

```python
import inspect
import os

from loguru import logger

from config import Config


# 注意这里
# @SingletonDecorator
class Log(object):
    business = None

    def __init__(self, name='pity'):  # Logger标识默认为app
        """
        :param name: 业务名称
        """
        # 如果目录不存在则创建
        if not os.path.exists(Config.LOG_DIR):
            os.mkdir(Config.LOG_DIR)
        self.business = name
        self.pity_info = logger.bind(name="pity_info")
        self.pity_error = logger.bind(name='pity_error')

    def info(self, message: str):
        file_name, line, func, _, _ = inspect.getframeinfo(inspect.currentframe().f_back)
        self.pity_info.info(f"func: {func} line: {line} module: {self.business} message: {message}")

    def error(self, message: str):
        file_name, line, func, _, _ = inspect.getframeinfo(inspect.currentframe().f_back)
        self.pity_error.error(f"func: {func} line: {line} module: {self.business} message: {message}")

    def warning(self, message: str):
        file_name, line, func, _, _ = inspect.getframeinfo(inspect.currentframe().f_back)
        self.pity_error.warning(f"func: {func} line: {line} module: {self.business} message: {message}")

    def debug(self, message: str):
        file_name, line, func, _, _ = inspect.getframeinfo(inspect.currentframe().f_back)
        self.pity_info.debug(f"func: {func} line: {line} module: {self.business} message: {message}")

```

  为了不影响之前的日志模块，所以我们没有大改，由于`系统日志`都是app/utils/logger.py文件的方法，所以导致自定义的日志都会出现: `显示不出来真正调用log.info的文件`这个问题。
  
  所以我这用了inspect自带的模块，获取到info/error等方法的调用方，一起来看看:
  
![](https://static.pity.fun/picture/2022-2-7/1644224332006-image.png)

  比如websocket这里，xxx连接上了，我们看看日志怎么展示的:
  
![](https://static.pity.fun/picture/2022-2-7/1644224407902-image.png)

  因为日志的调用一直是logger文件下的info方法，所以每次都会打印app.utils.logger.info，而我们用了inspect之后就不一样了，能获取到真实的方法:
  
  `图中可以看到是connect方法，44行，由于文件名巨长，所以我没放进去，其实也是可以获取到的。感兴趣的朋友可以自己改装下。`
  
  这块内容就当作业留给大家了，咱们下期再见！
  

  