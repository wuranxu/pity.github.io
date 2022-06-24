> 大家好~我是`米洛`！<br/>
> 我正在从0到1打造一个开源的接口测试平台, 也在编写一套与之对应的`教程`，希望大家多多支持。<br/>
> 欢迎关注我的公众号`米洛的测开日记`，获取最新文章教程! 

### 回顾

  上一节我们把oss运用到了`实战之中`，并且完成了http请求支持文件上传的需求。由于下一节的内容我还没有想好，想着先把免费的`gitee` oss搞定再说。
  
  但在编写代码的过程中发现许多问题或者说`坑`，让我有些怀疑这块是不是该收手了。
  
### 了解gitee api

  gitee这样相对比较大的平台，一般都会对外提供api操作对应的仓库数据。
  
  他们的文档地址: [https://gitee.com/api/v5/swagger#/getV5ReposOwnerRepoStargazers?ex=no](https://gitee.com/api/v5/swagger#/getV5ReposOwnerRepoStargazers?ex=no)
  
![](https://static.pity.fun/picture/2021-12-2/1638452094546-image.png)

  我们可以看到，它的api甚至都已经是第五个版本了（v5代表的是api的版本）。
  
  所有的api都可以从这个网站查询到，话不多说，我们赶紧找找自己想要的api: `查询仓库的文件`，`下载文件`, `新增文件`, `删除文件`。
  
  找了一圈却发现，**gitee没有提供类似os.walk或者能够获取到所有目录/文件的api，只有下面这种:**

![也就是说只能获取到一层目录的数据，如果目录套了目录，就得继续调用这个api](https://static.pity.fun/picture/2021-12-2/1638452432125-image.png)

  也就是说，如果我们是`深层次目录`，可能要`不断遍历`下去。
  
  但如果我们限制下用户使用的时候，不嵌套那么多目录，应该也`还能凑合用`，毕竟没有什么比免费更香了。
  
### 编写客户端获取部分

- 研究下gitee配置和aliyun的不同

  在gitee里面，只用3个数据就能确定一个项目:
  
  1. access_token(用来判断你是哪个用户)
  2. repo(仓库)
  3. owner(用户)
  
![](https://static.pity.fun/picture/2021-12-2/1638452859351-image.png)

  所以我们把endpoint这个置空，毕竟用不到，其他的一次为`access_token`, `owner`和`repo`。
  
- 改写oss获取客户端的方法

![](https://static.pity.fun/picture/2021-12-2/1638452946361-image.png)

  如果类型是GITEE，则返回GiteeOss，当然此时GiteeOss里面具体的方法还没有实现。
  
  **注意这里我在Config里头加入了GITEE和ALIYUN变量，如果报找不到变量的错，是因为我封装了一下，希望大家能够自己发现。**
  
  由于教程不是纯新手教程，在`发现我留的坑`或者`代码悄悄改动`的时候，也是大家进步的时候，因为不需要作业，天然给你留了作业。（机智的我）
  
### 实现GiteeOss中的方法

- 初始化方法


![](https://static.pity.fun/picture/2021-12-2/1638459134640-image.png)

  值得注意的是，我设置了个base_path，是为了能够把pity搞的oss文件全部放入pity文件夹。其他的就是gitee的基础api地址。
  
- 创建文件

```python
    @aioify
    async def create_file(self, filepath: str, content: bytes):
        gitee_url = self._base_url.format(self.user, self.repo, self._base_path, filepath)
        data = base64.b64encode(content).decode()
        json_data = {"access_token": self.token, "message": "pity create file", "content": data}
        r = requests.post(url=gitee_url, json=json_data)
        if r.status_code == 400:
            raise Exception("文件重复，请先删除旧文件")
        if r.status_code != 201:
            raise Exception("上传文件到gitee失败")
```

  注意这里有个`aioify`装饰器，这需要额外安装aioify包，然后这个方法就能被`异步执行`，还是挺好用的。
  
  其实没有很复杂的内容，就是调用gitee api的接口，去上传文件，如果http状态码是400说明文件重复，201说明成功（虽然我也很`纳闷`为什么不是`200`）。
  
- 删除文件

```python
    @aioify
    async def delete_file(self, filepath: str):
        filepath, sha = filepath.split("$")
        gitee_url = self._base_url.format(self.user, self.repo, self._base_path, filepath)
        r = requests.delete(gitee_url, params=dict(access_token=self.token, message="delete file", sha=sha))
        if r.status_code != 200:
            raise Exception("刪除文件失败")
```

- 获取文件列表

```python

    @aioify
    async def list_file(self):
        ans = list()
        await self.query_file_by_path("", ans)
        return ans
        
    @aioify
    async def query_file_by_path(self, filepath: str, ans: List[dict]):
        gitee_url = self._base_url.format(self.user, self.repo, self._base_path, filepath)
        r = requests.get(gitee_url, params=dict(access_token=self.token))
        file_list = r.json()
        for f in file_list:
            if f.get("type") == "file":
                ans.append(dict(key=f.get('path')[5:], download_url=f.get("download_url"),
                                size=0, sha=f.get("sha"),
                                last_modified=None, type='gitee'))
                continue
            await self.query_file_by_path(f"{filepath}/{f.get('name')}", ans)
```

  这边首先获取`根目录`下的所有文件，接着判断文件列表，如果是目录则继续获取该目录下的文件，否则把文件数据放入`ans`，最终返回ans即可。
  
  要注意的是，gitee的文件下载需要用到`sha`字段，而且gitee不支持查看`最后编辑时间`和`文件大小`，这2点也是鸡肋的点。
  
- 下载文件

![](https://static.pity.fun/picture/2021-12-2/1638459098905-image.png)

  大家可以看看调用下载目录后，返回的data数据。我这里简单讲下，他其实是一个json，并不是文件流。
  
```json
{"content": "文件base64编码的byte字符串", "size": "文件大小"}
```

  所以我们取出来数据后要继续用base64decode，最后写入我们生成的临时文件，通过fastapi的FileResponse提供下载功能。
  
### 总结

  gitee最大特点就是免费，作为oss实在是不方便，作为图床倒是不错，目前也没有`防盗链`这一说。
  
- 获取文件不方便
- 不能获取文件大小，除非你下载文件
- 没有更新时间
  
  所以最后咱们oss的管理页面长这样:
  
![](https://static.pity.fun/picture/2021-12-2/1638459393264-image.png)

  但其实是我们偷懒，如果在文件上传的时候我们`记录一份文件相关数据`到本地，那这些都不是问题。那应该还是香的吧！（但又可能存在数据不同步问题）
  
  怎么说呢，一开始是懒，看来还是付出代价了~这个坑先放着，以后再填吧~
  
  