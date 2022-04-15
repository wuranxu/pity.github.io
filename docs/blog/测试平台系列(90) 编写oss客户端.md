<div class="markdown-body" style="padding: 0 10px; word-spacing: 0px; word-wrap: break-word; text-align: left; font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, 'PingFang SC', Cambria, Cochin, Georgia, Times, 'Times New Roman', serif; line-height: 1.6; letter-spacing: .034em; color: rgb(63, 63, 63); font-size: 16px; word-break: all;"><blockquote data-tool="mdnice-plus编辑器" style="border: none; padding: 15px 20px; line-height: 27px; background-color: #FBF9FD; border-left: 4px solid #409EFF; display: block;">
<p style="padding-bottom: 8px; margin: 0; padding-top: 1em; line-height: 1.75em; padding: 0px; font-size: 15px; color: rgb(89, 89, 89);">大家好~我是<code style="font-size: 14px; word-wrap: break-word; padding: 2px 4px; border-radius: 4px; margin: 0 2px; background-color: rgba(27,31,35,.05); font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; word-break: break-all; color: #409EFF;">米洛</code>！<br>
我正在从0到1打造一个开源的接口测试平台, 也在编写一套与之对应的<code style="font-size: 14px; word-wrap: break-word; padding: 2px 4px; border-radius: 4px; margin: 0 2px; background-color: rgba(27,31,35,.05); font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; word-break: break-all; color: #409EFF;">教程</code>，希望大家多多支持。<br>
欢迎关注我的公众号<code style="font-size: 14px; word-wrap: break-word; padding: 2px 4px; border-radius: 4px; margin: 0 2px; background-color: rgba(27,31,35,.05); font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; word-break: break-all; color: #409EFF;">米洛的测开日记</code>，获取最新文章教程!</p>
</blockquote>
<h3 data-tool="mdnice-plus编辑器" style="margin-bottom: 15px; padding: 0px; margin-top: 1.2em; font-size: 17px; font-weight: bold; display: block; margin-left: 8px; color: #409EFF;"><span>📒 </span>回顾</h3>
<p data-tool="mdnice-plus编辑器" style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;">上一节我们编写了<code style="font-size: 14px; word-wrap: break-word; padding: 2px 4px; border-radius: 4px; margin: 0 2px; background-color: rgba(27,31,35,.05); font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; word-break: break-all; color: #409EFF;">在线执行测试计划</code>功能，并稍微改了下报告页面。那其实我们之前的内容都是有很多<strong style="font-weight: bold; line-height: 1.75em; color: rgb(74, 74, 74);">坑</strong>在里面的。</p>
<p data-tool="mdnice-plus编辑器" style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;">比如http请求只支持了json和form，没有支持<code style="font-size: 14px; word-wrap: break-word; padding: 2px 4px; border-radius: 4px; margin: 0 2px; background-color: rgba(27,31,35,.05); font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; word-break: break-all; color: #409EFF;">文件上传</code>的请求，甚至有一些crud的功能都没有太完善。</p>
<p data-tool="mdnice-plus编辑器" style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;">不过不要紧，我的想法还是先<code style="font-size: 14px; word-wrap: break-word; padding: 2px 4px; border-radius: 4px; margin: 0 2px; background-color: rgba(27,31,35,.05); font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; word-break: break-all; color: #409EFF;">创造</code>，再完善。当然也不是盲目创造，也得提前预判好后面的走向。</p>
<h3 data-tool="mdnice-plus编辑器" style="margin-bottom: 15px; padding: 0px; margin-top: 1.2em; font-size: 17px; font-weight: bold; display: block; margin-left: 8px; color: #409EFF;"><span>📒 </span>为什么要用oss</h3>
<p data-tool="mdnice-plus编辑器" style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;">pity里面oss打算用在2个地方，第一个就是一些静态资源图片。比方说<code style="font-size: 14px; word-wrap: break-word; padding: 2px 4px; border-radius: 4px; margin: 0 2px; background-color: rgba(27,31,35,.05); font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; word-break: break-all; color: #409EFF;">项目图片</code>，<code style="font-size: 14px; word-wrap: break-word; padding: 2px 4px; border-radius: 4px; margin: 0 2px; background-color: rgba(27,31,35,.05); font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; word-break: break-all; color: #409EFF;">用户头像</code>。</p>
<p data-tool="mdnice-plus编辑器" style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;">另一个地方就是上文说的，测试文件上传接口的时候，我们需要<code style="font-size: 14px; word-wrap: break-word; padding: 2px 4px; border-radius: 4px; margin: 0 2px; background-color: rgba(27,31,35,.05); font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; word-break: break-all; color: #409EFF;">测试文件</code>。这些文件怎么来，怎么管理？都得借助oss来完成。</p>
<p data-tool="mdnice-plus编辑器" style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;">目前以我熟悉的oss为例，打算支持以下几种oss:</p>
<ul class="contains-task-list" data-tool="mdnice-plus编辑器" style="margin-top: 8px; margin-bottom: 8px; padding-left: 25px; color: black; list-style-type: none;">
<li class="task-list-item">
<p style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;"><input type="checkbox" checked disabled> 阿里云oss ✔</p>
</li>
<li class="task-list-item">
<p style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;"><input type="checkbox" checked disabled> <del style="font-style: italic; color: black;">腾讯云cos</del>(无账号) 💊</p>
</li>
<li class="task-list-item">
<p style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;"><input type="checkbox" checked disabled> 七牛云(免费，我能给展示demo)</p>
</li>
<li class="task-list-item">
<p style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;"><input type="checkbox" checked disabled> gitee(免费)</p>
<p style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;">其中阿里云和腾讯云由于都是<code style="font-size: 14px; word-wrap: break-word; padding: 2px 4px; border-radius: 4px; margin: 0 2px; background-color: rgba(27,31,35,.05); font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; word-break: break-all; color: #409EFF;">付费产品</code>，博主还是买不起的。所以可能需要有对应的测试账号，在此感谢小右(Mini-Right)的帮助，给了我一个阿里云的测试账号，保证了crud能正常进行。</p>
<p style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;"><strong style="font-weight: bold; line-height: 1.75em; color: rgb(74, 74, 74);">如果需要我实现其他oss客户端，请带上对应的测试账号私信我哈。</strong></p>
<p style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;">测试网站的数据估计会用<code style="font-size: 14px; word-wrap: break-word; padding: 2px 4px; border-radius: 4px; margin: 0 2px; background-color: rgba(27,31,35,.05); font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; word-break: break-all; color: #409EFF;">gitee</code>或者<code style="font-size: 14px; word-wrap: break-word; padding: 2px 4px; border-radius: 4px; margin: 0 2px; background-color: rgba(27,31,35,.05); font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; word-break: break-all; color: #409EFF;">七牛云</code>。</p>
</li>
</ul>
<h3 data-tool="mdnice-plus编辑器" style="margin-bottom: 15px; padding: 0px; margin-top: 1.2em; font-size: 17px; font-weight: bold; display: block; margin-left: 8px; color: #409EFF;"><span>📒 </span>说明</h3>
<p data-tool="mdnice-plus编辑器" style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;">关于文件管理这块，由于时间关系，我<code style="font-size: 14px; word-wrap: break-word; padding: 2px 4px; border-radius: 4px; margin: 0 2px; background-color: rgba(27,31,35,.05); font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; word-break: break-all; color: #409EFF;">暂时不会</code>落一张表与oss数据进行关联，主要目的是为了<code style="font-size: 14px; word-wrap: break-word; padding: 2px 4px; border-radius: 4px; margin: 0 2px; background-color: rgba(27,31,35,.05); font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; word-break: break-all; color: #409EFF;">节省时间</code>。</p>
<p data-tool="mdnice-plus编辑器" style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;">表关联可以作为二期工程。</p>
<h3 data-tool="mdnice-plus编辑器" style="margin-bottom: 15px; padding: 0px; margin-top: 1.2em; font-size: 17px; font-weight: bold; display: block; margin-left: 8px; color: #409EFF;"><span>📒 </span>编写oss基类</h3>
<p data-tool="mdnice-plus编辑器" style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;">新建<code style="font-size: 14px; word-wrap: break-word; padding: 2px 4px; border-radius: 4px; margin: 0 2px; background-color: rgba(27,31,35,.05); font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; word-break: break-all; color: #409EFF;">app.middleware.oss.oss_file.py</code></p>
<p data-tool="mdnice-plus编辑器" style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;">由于咱们支持多种oss客户端，所以包装好一个抽象类（类似go的interface）等着各个<code style="font-size: 14px; word-wrap: break-word; padding: 2px 4px; border-radius: 4px; margin: 0 2px; background-color: rgba(27,31,35,.05); font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; word-break: break-all; color: #409EFF;">oss客户端</code>去实现之。</p>
<pre data-tool="mdnice-plus编辑器" style="margin-top: 10px; margin-bottom: 10px;"><code class="hljs language-python" style="overflow-x: auto; padding: 0.5em; color: #abb2bf; background: #282c34; display: -webkit-box; font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; border-radius: 0px; font-size: 12px; -webkit-overflow-scrolling: touch;"><span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">from</span> abc <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">import</span> ABC, abstractmethod
<span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">from</span> typing <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">import</span> ByteString


<span class="hljs-class" style="line-height: 26px;"><span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">class</span> <span class="hljs-title" style="line-height: 26px; color: #e6c07b;">OssFile</span>(<span class="hljs-params" style="line-height: 26px;">ABC</span>):</span>

<span class="hljs-meta" style="color: #61aeee; line-height: 26px;">    @abstractmethod</span>
    <span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">def</span> <span class="hljs-title" style="color: #61aeee; line-height: 26px;">create_file</span>(<span class="hljs-params" style="line-height: 26px;">self, filepath: <span class="hljs-built_in" style="color: #e6c07b; line-height: 26px;">str</span>, content: ByteString</span>):</span>
        <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">pass</span>

<span class="hljs-meta" style="color: #61aeee; line-height: 26px;">    @abstractmethod</span>
    <span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">def</span> <span class="hljs-title" style="color: #61aeee; line-height: 26px;">update_file</span>(<span class="hljs-params" style="line-height: 26px;">self, filepath: <span class="hljs-built_in" style="color: #e6c07b; line-height: 26px;">str</span>, content: ByteString</span>):</span>
        <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">pass</span>

<span class="hljs-meta" style="color: #61aeee; line-height: 26px;">    @abstractmethod</span>
    <span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">def</span> <span class="hljs-title" style="color: #61aeee; line-height: 26px;">delete_file</span>(<span class="hljs-params" style="line-height: 26px;">self, filepath: <span class="hljs-built_in" style="color: #e6c07b; line-height: 26px;">str</span></span>):</span>
        <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">pass</span>

<span class="hljs-meta" style="color: #61aeee; line-height: 26px;">    @abstractmethod</span>
    <span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">def</span> <span class="hljs-title" style="color: #61aeee; line-height: 26px;">list_file</span>(<span class="hljs-params" style="line-height: 26px;">self</span>):</span>
        <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">pass</span>

<span class="hljs-meta" style="color: #61aeee; line-height: 26px;">    @abstractmethod</span>
    <span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">def</span> <span class="hljs-title" style="color: #61aeee; line-height: 26px;">download_file</span>(<span class="hljs-params" style="line-height: 26px;">self, filepath</span>):</span>
        <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">pass</span>

</code></pre>
<p data-tool="mdnice-plus编辑器" style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;">抽象类没有具体的实现，只有方法的定义，目的是为了<code style="font-size: 14px; word-wrap: break-word; padding: 2px 4px; border-radius: 4px; margin: 0 2px; background-color: rgba(27,31,35,.05); font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; word-break: break-all; color: #409EFF;">限制子类的方法，即必须实现基类中定义的方法。</code></p>
<p data-tool="mdnice-plus编辑器" style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;">总结了一下，必须有<code style="font-size: 14px; word-wrap: break-word; padding: 2px 4px; border-radius: 4px; margin: 0 2px; background-color: rgba(27,31,35,.05); font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; word-break: break-all; color: #409EFF;">增删改查下载</code>5种方法。</p>
<h3 data-tool="mdnice-plus编辑器" style="margin-bottom: 15px; padding: 0px; margin-top: 1.2em; font-size: 17px; font-weight: bold; display: block; margin-left: 8px; color: #409EFF;"><span>📒 </span>编写AliyunOss实现</h3>
<p data-tool="mdnice-plus编辑器" style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;">新建<code style="font-size: 14px; word-wrap: break-word; padding: 2px 4px; border-radius: 4px; margin: 0 2px; background-color: rgba(27,31,35,.05); font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; word-break: break-all; color: #409EFF;">app.middleware.oss.aliyun.py</code></p>
<pre data-tool="mdnice-plus编辑器" style="margin-top: 10px; margin-bottom: 10px;"><code class="hljs language-python" style="overflow-x: auto; padding: 0.5em; color: #abb2bf; background: #282c34; display: -webkit-box; font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; border-radius: 0px; font-size: 12px; -webkit-overflow-scrolling: touch;"><span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">from</span> typing <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">import</span> ByteString

<span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">import</span> oss2

<span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">from</span> app.middleware.oss.files <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">import</span> OssFile


<span class="hljs-class" style="line-height: 26px;"><span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">class</span> <span class="hljs-title" style="line-height: 26px; color: #e6c07b;">AliyunOss</span>(<span class="hljs-params" style="line-height: 26px;">OssFile</span>):</span>

    <span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">def</span> <span class="hljs-title" style="color: #61aeee; line-height: 26px;">__init__</span>(<span class="hljs-params" style="line-height: 26px;">self, access_key_id: <span class="hljs-built_in" style="color: #e6c07b; line-height: 26px;">str</span>, access_key_secret: <span class="hljs-built_in" style="color: #e6c07b; line-height: 26px;">str</span>, endpoint: <span class="hljs-built_in" style="color: #e6c07b; line-height: 26px;">str</span>, bucket: <span class="hljs-built_in" style="color: #e6c07b; line-height: 26px;">str</span></span>):</span>
        auth = oss2.Auth(access_key_id=access_key_id,
                         access_key_secret=access_key_secret)
        <span class="hljs-comment" style="color: #5c6370; font-style: italic; line-height: 26px;"># auth = oss2.AnonymousAuth()</span>
        self.bucket = oss2.Bucket(auth, endpoint, bucket)

    <span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">def</span> <span class="hljs-title" style="color: #61aeee; line-height: 26px;">create_file</span>(<span class="hljs-params" style="line-height: 26px;">self, filepath: <span class="hljs-built_in" style="color: #e6c07b; line-height: 26px;">str</span>, content: ByteString</span>):</span>
        self.bucket.put_object(filepath, content)

    <span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">def</span> <span class="hljs-title" style="color: #61aeee; line-height: 26px;">update_file</span>(<span class="hljs-params" style="line-height: 26px;">self, filepath: <span class="hljs-built_in" style="color: #e6c07b; line-height: 26px;">str</span>, content: ByteString</span>):</span>
        self.bucket.put_object(filepath, content)

    <span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">def</span> <span class="hljs-title" style="color: #61aeee; line-height: 26px;">delete_file</span>(<span class="hljs-params" style="line-height: 26px;">self, filepath: <span class="hljs-built_in" style="color: #e6c07b; line-height: 26px;">str</span></span>):</span>
        self.bucket.delete_object(filepath)

    <span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">def</span> <span class="hljs-title" style="color: #61aeee; line-height: 26px;">list_file</span>(<span class="hljs-params" style="line-height: 26px;">self</span>):</span>
        ans = []
        <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">for</span> obj <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">in</span> oss2.ObjectIteratorV2(self.bucket):
            ans.append(<span class="hljs-built_in" style="color: #e6c07b; line-height: 26px;">dict</span>(key=obj.key, last_modified=obj.last_modified,
                            size=obj.size, owner=obj.owner))
        <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">return</span> ans

    <span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">def</span> <span class="hljs-title" style="color: #61aeee; line-height: 26px;">download_file</span>(<span class="hljs-params" style="line-height: 26px;">self, filepath</span>):</span>
        <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">if</span> <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">not</span> self.bucket.object_exists(filepath):
            <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">raise</span> Exception(<span class="hljs-string" style="color: #98c379; line-height: 26px;">f"oss文件: <span class="hljs-subst" style="color: #e06c75; line-height: 26px;">{filepath}</span>不存在"</span>)
        <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">return</span> self.bucket.get_object(filepath)
</code></pre>
<p data-tool="mdnice-plus编辑器" style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;">AliyunOss<strong style="font-weight: bold; line-height: 1.75em; color: rgb(74, 74, 74);">继承</strong>了OssFile，构造方法获取阿里云的身份信息，并验证。最后读取bucket，这bucket我理解的是一块<code style="font-size: 14px; word-wrap: break-word; padding: 2px 4px; border-radius: 4px; margin: 0 2px; background-color: rgba(27,31,35,.05); font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; word-break: break-all; color: #409EFF;">区域</code>，你的文件都存储在这块区域里面，我们就叫他F盘吧。</p>
<p data-tool="mdnice-plus编辑器" style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;">其他的方法很简单，基本上是调用对应的api，去做crud操作。oss没有<code style="font-size: 14px; word-wrap: break-word; padding: 2px 4px; border-radius: 4px; margin: 0 2px; background-color: rgba(27,31,35,.05); font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; word-break: break-all; color: #409EFF;">文件夹</code>的概念，都是统一用路径来存储文件地址的，比如:</p>
<p data-tool="mdnice-plus编辑器" style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;">woody/github.txt</p>
<p data-tool="mdnice-plus编辑器" style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;">这个文件路径可以理解为，woody目录下的<code style="font-size: 14px; word-wrap: break-word; padding: 2px 4px; border-radius: 4px; margin: 0 2px; background-color: rgba(27,31,35,.05); font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; word-break: break-all; color: #409EFF;">github.txt</code>文件。</p>
<h3 data-tool="mdnice-plus编辑器" style="margin-bottom: 15px; padding: 0px; margin-top: 1.2em; font-size: 17px; font-weight: bold; display: block; margin-left: 8px; color: #409EFF;"><span>📒 </span>编写获取客户端方法</h3>
<p data-tool="mdnice-plus编辑器" style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;"><code style="font-size: 14px; word-wrap: break-word; padding: 2px 4px; border-radius: 4px; margin: 0 2px; background-color: rgba(27,31,35,.05); font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; word-break: break-all; color: #409EFF;">app.middleware.oss._\_init__.py</code></p>
<pre data-tool="mdnice-plus编辑器" style="margin-top: 10px; margin-bottom: 10px;"><code class="hljs language-python" style="overflow-x: auto; padding: 0.5em; color: #abb2bf; background: #282c34; display: -webkit-box; font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; border-radius: 0px; font-size: 12px; -webkit-overflow-scrolling: touch;"><span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">from</span> app.core.configuration <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">import</span> SystemConfiguration
<span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">from</span> app.middleware.oss.aliyun <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">import</span> AliyunOss
<span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">from</span> app.middleware.oss.files <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">import</span> OssFile


<span class="hljs-class" style="line-height: 26px;"><span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">class</span> <span class="hljs-title" style="line-height: 26px; color: #e6c07b;">OssClient</span>(<span class="hljs-params" style="line-height: 26px;"><span class="hljs-built_in" style="color: #e6c07b; line-height: 26px;">object</span></span>):</span>
    _client = <span class="hljs-literal" style="color: #56b6c2; line-height: 26px;">None</span>

<span class="hljs-meta" style="color: #61aeee; line-height: 26px;">    @classmethod</span>
    <span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">def</span> <span class="hljs-title" style="color: #61aeee; line-height: 26px;">get_oss_client</span>(<span class="hljs-params" style="line-height: 26px;">cls</span>) -&gt; OssFile:</span>
        <span class="hljs-string" style="color: #98c379; line-height: 26px;">"""
        通过oss配置拿到oss客户端
        :return:
        """</span>
        <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">if</span> OssClient._client <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">is</span> <span class="hljs-literal" style="color: #56b6c2; line-height: 26px;">None</span>:
            cfg = SystemConfiguration.get_config()
            oss_config = cfg.get(<span class="hljs-string" style="color: #98c379; line-height: 26px;">"oss"</span>)
            access_key_id = oss_config.get(<span class="hljs-string" style="color: #98c379; line-height: 26px;">"access_key_id"</span>)
            access_key_secret = oss_config.get(<span class="hljs-string" style="color: #98c379; line-height: 26px;">"access_key_secret"</span>)
            bucket = oss_config.get(<span class="hljs-string" style="color: #98c379; line-height: 26px;">"bucket"</span>)
            endpoint = oss_config.get(<span class="hljs-string" style="color: #98c379; line-height: 26px;">"endpoint"</span>)
            <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">if</span> oss_config <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">is</span> <span class="hljs-literal" style="color: #56b6c2; line-height: 26px;">None</span>:
                <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">raise</span> Exception(<span class="hljs-string" style="color: #98c379; line-height: 26px;">"服务器未配置oss信息, 请在configuration.json中添加"</span>)
            <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">if</span> oss_config.get(<span class="hljs-string" style="color: #98c379; line-height: 26px;">"type"</span>).lower() == <span class="hljs-string" style="color: #98c379; line-height: 26px;">"aliyun"</span>:
                <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">return</span> AliyunOss(access_key_id, access_key_secret, endpoint, bucket)
            <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">raise</span> Exception(<span class="hljs-string" style="color: #98c379; line-height: 26px;">"不支持的oss类型"</span>)
        <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">return</span> OssClient._client

</code></pre>
<p data-tool="mdnice-plus编辑器" style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;">我们在configuration.json配置oss信息，接着每次都从OssClient获取客户端即可。</p>
<p data-tool="mdnice-plus编辑器" style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;"><img src="http://oss.pity.fun/picture/2021-11-27/1638022147065-image.png" alt="我们根据这个类型来判断oss的类型" class="medium-zoom-image" style="display: block; margin: 0 auto; max-width: 100%; border-radius: 4px; margin-bottom: 25px;"></p>
<h3 data-tool="mdnice-plus编辑器" style="margin-bottom: 15px; padding: 0px; margin-top: 1.2em; font-size: 17px; font-weight: bold; display: block; margin-left: 8px; color: #409EFF;"><span>📒 </span>编写后端接口</h3>
<pre data-tool="mdnice-plus编辑器" style="margin-top: 10px; margin-bottom: 10px;"><code class="hljs language-python" style="overflow-x: auto; padding: 0.5em; color: #abb2bf; background: #282c34; display: -webkit-box; font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; border-radius: 0px; font-size: 12px; -webkit-overflow-scrolling: touch;"><span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">from</span> fastapi <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">import</span> APIRouter, File, UploadFile

<span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">from</span> app.handler.fatcory <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">import</span> PityResponse
<span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">from</span> app.middleware.oss <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">import</span> OssClient

router = APIRouter(prefix=<span class="hljs-string" style="color: #98c379; line-height: 26px;">"/oss"</span>)


<span class="hljs-meta" style="color: #61aeee; line-height: 26px;">@router.post(<span class="hljs-params" style="line-height: 26px;"><span class="hljs-string" style="color: #98c379; line-height: 26px;">"/upload"</span></span>)</span>
<span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">async</span> <span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">def</span> <span class="hljs-title" style="color: #61aeee; line-height: 26px;">create_oss_file</span>(<span class="hljs-params" style="line-height: 26px;">filepath: <span class="hljs-built_in" style="color: #e6c07b; line-height: 26px;">str</span>, file: UploadFile = File(<span class="hljs-params" style="line-height: 26px;">...</span>)</span>):</span>
    <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">try</span>:
        file_content = <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">await</span> file.read()
        <span class="hljs-comment" style="color: #5c6370; font-style: italic; line-height: 26px;"># 获取oss客户端</span>
        client = OssClient.get_oss_client()
        client.create_file(filepath, file_content)
        <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">return</span> PityResponse.success()
    <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">except</span> Exception <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">as</span> e:
        <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">return</span> PityResponse.failed(<span class="hljs-string" style="color: #98c379; line-height: 26px;">f"上传失败: <span class="hljs-subst" style="color: #e06c75; line-height: 26px;">{e}</span>"</span>)


<span class="hljs-meta" style="color: #61aeee; line-height: 26px;">@router.get(<span class="hljs-params" style="line-height: 26px;"><span class="hljs-string" style="color: #98c379; line-height: 26px;">"/list"</span></span>)</span>
<span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">async</span> <span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">def</span> <span class="hljs-title" style="color: #61aeee; line-height: 26px;">list_oss_file</span>():</span>
    <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">try</span>:
        client = OssClient.get_oss_client()
        files = client.list_file()
        <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">return</span> PityResponse.success(files)
    <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">except</span> Exception <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">as</span> e:
        <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">return</span> PityResponse.failed(<span class="hljs-string" style="color: #98c379; line-height: 26px;">f"获取失败: <span class="hljs-subst" style="color: #e06c75; line-height: 26px;">{e}</span>"</span>)


<span class="hljs-meta" style="color: #61aeee; line-height: 26px;">@router.get(<span class="hljs-params" style="line-height: 26px;"><span class="hljs-string" style="color: #98c379; line-height: 26px;">"/delete"</span></span>)</span>
<span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">async</span> <span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">def</span> <span class="hljs-title" style="color: #61aeee; line-height: 26px;">delete_oss_file</span>(<span class="hljs-params" style="line-height: 26px;">filepath: <span class="hljs-built_in" style="color: #e6c07b; line-height: 26px;">str</span></span>):</span>
    <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">try</span>:
        client = OssClient.get_oss_client()
        client.delete_file(filepath)
        <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">return</span> PityResponse.success()
    <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">except</span> Exception <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">as</span> e:
        <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">return</span> PityResponse.failed(<span class="hljs-string" style="color: #98c379; line-height: 26px;">f"删除失败: <span class="hljs-subst" style="color: #e06c75; line-height: 26px;">{e}</span>"</span>)


<span class="hljs-meta" style="color: #61aeee; line-height: 26px;">@router.post(<span class="hljs-params" style="line-height: 26px;"><span class="hljs-string" style="color: #98c379; line-height: 26px;">"/update"</span></span>)</span>
<span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">async</span> <span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">def</span> <span class="hljs-title" style="color: #61aeee; line-height: 26px;">update_oss_file</span>(<span class="hljs-params" style="line-height: 26px;">filepath: <span class="hljs-built_in" style="color: #e6c07b; line-height: 26px;">str</span>, file: UploadFile = File(<span class="hljs-params" style="line-height: 26px;">...</span>)</span>):</span>
    <span class="hljs-string" style="color: #98c379; line-height: 26px;">"""
    更新oss文件，路径不能变化
    :param filepath:
    :param file:
    :return:
    """</span>
    <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">try</span>:
        client = OssClient.get_oss_client()
        file_content = <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">await</span> file.read()
        client.update_file(filepath, file_content)
        <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">return</span> PityResponse.success()
    <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">except</span> Exception <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">as</span> e:
        <span class="hljs-keyword" style="color: #c678dd; line-height: 26px;">return</span> PityResponse.failed(<span class="hljs-string" style="color: #98c379; line-height: 26px;">f"删除失败: <span class="hljs-subst" style="color: #e06c75; line-height: 26px;">{e}</span>"</span>)
</code></pre>
<p data-tool="mdnice-plus编辑器" style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;">方法很简单，文件路径在url参数里边，文件的话，利用fastapi里面的File和UploadFile皆可获取到上传的文件。</p>
<p data-tool="mdnice-plus编辑器" style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;"><strong style="font-weight: bold; line-height: 1.75em; color: rgb(74, 74, 74);">注意，必须先安装python-multipart库配合文件上传</strong></p>
<h3 data-tool="mdnice-plus编辑器" style="margin-bottom: 15px; padding: 0px; margin-top: 1.2em; font-size: 17px; font-weight: bold; display: block; margin-left: 8px; color: #409EFF;"><span>📒 </span>测试一下</h3>
<p data-tool="mdnice-plus编辑器" style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;">成功读取到了oss的文件信息，但下载接口</p>
<p data-tool="mdnice-plus编辑器" style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;"><img src="http://oss.pity.fun/picture/2021-11-27/1638022289631-image.png" alt class="medium-zoom-image" style="display: block; margin: 0 auto; max-width: 100%; border-radius: 4px; margin-bottom: 25px;"></p>
<p data-tool="mdnice-plus编辑器" style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;">好像忘记下载文件相关内容了，那么参考我的<code style="font-size: 14px; word-wrap: break-word; padding: 2px 4px; border-radius: 4px; margin: 0 2px; background-color: rgba(27,31,35,.05); font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; word-break: break-all; color: #409EFF;">上一篇FsatApi下载文件的文章</code>吧。</p>
<p data-tool="mdnice-plus编辑器" style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;"><a href="https://juejin.cn/post/7029591719997865998" style="word-wrap: break-word; font-weight: bold; color: #409EFF; text-decoration: none; border-bottom: 1px solid #1d7dfa;">FastApi下载文件</a></p>
<p data-tool="mdnice-plus编辑器" style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;">或者直接去github查看源码~</p>
<p data-tool="mdnice-plus编辑器" style="font-size: 16px; padding-bottom: 8px; margin: 0; padding-top: 1em; color: rgb(74, 74, 74); line-height: 1.75em;">今天的内容就介绍到这里了，下一节卷oss的用途。</p><!--6--></div>