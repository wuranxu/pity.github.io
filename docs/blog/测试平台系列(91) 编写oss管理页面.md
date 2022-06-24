> 大家好~我是`米洛`！<br/>
> 我正在从0到1打造一个开源的接口测试平台, 也在编写一套与之对应的`教程`，希望大家多多支持。<br/>
> 欢迎关注我的公众号`米洛的测开日记`，获取最新文章教程! 

### 回顾

  上一节我们编写好了`oss`相关的crud接口，那这一节我们就得为oss数据的管理编写一个新的页面了。
  
  即将做的是一个极度精简的`文件管理页面`。
  
### 效果图

  因为我每次都是写完一段代码，然后编写对应教程，所以效果图这种东西自然是不在话下:
  
![](https://static.pity.fun/picture/2021-11-29/1638196757075-image.png)

  图片可以点击下载，也可以`删除`。
  
### 编写oss下载接口

  在此之前还是先搞定下之前的`作业`。

- 编写随机获取文件名的方法

![](https://static.pity.fun/picture/2021-11-29/1638196914641-image.png)

  根据传入的文件名，配合pity的打乱顺序+当前时间戳，基本可以`保证百分之90`的文件名冲突问题。
  
- 实现阿里云的下载文件方法

![](https://static.pity.fun/picture/2021-11-29/1638196986128-image.png)

  调用get_object_to_file即可，并返回新的文件路径。
  
- 封装下载文件的方法

  
![这里用到了BackgroundTask](https://static.pity.fun/picture/2021-11-29/1638197215729-image.png)

  
- 编写下载文件接口

```python
@router.get("/download")
async def download_oss_file(filepath: str):
    """
    更新oss文件，路径不能变化
    :param filepath:
    :return:
    """
    try:
        client = OssClient.get_oss_client()
        # 切割获取文件名
        filename = filepath.split("/")[-1]
        path = client.download_file(filepath, filename)
        return PityResponse.file(path, filename)
    except Exception as e:
        return PityResponse.failed(f"下载失败: {e}")
```

  注意: 由于oss里面的`文件名`都是带路径的（目录），所以我们split一下取出`文件名`。
  
### 看看下载的效果

![](https://static.pity.fun/picture/2021-11-29/1638197276913-case%E6%8B%96%E6%8B%BD2.gif)

### 前端页面部分

  前端页面表格那块自然是`传统手艺`，不需要多说了。
  
  值得注意的是下面这几个地方:
  
- 上传文件表单

  上传文件表单里面有个Upload组件，这个比较特殊，我们需要手动上传，我也踩了很久的坑。
  
  这里附上源码:

![](https://static.pity.fun/picture/2021-11-29/1638197537829-image.png)

  这些几乎都是根据antd官网的例子来的，其中Form.Item套了2层，demo就是这么写的，我试了下去掉一个还不行。
  
  valuePropName固定要是`fileList`。

![](https://static.pity.fun/picture/2021-11-29/1638197615868-image.png)

- 上传文件的http请求

  由于前端未使用axios这样的http请求库，而是umi-request（自己封装的）。
  
  所以这边还是说明一下怎么使用:
  
![](https://static.pity.fun/picture/2021-11-29/1638197722174-image.png)

  这是文件上传的方法，第一是要制造`FormData`对象，并把文件数据append进去。
  
  第二个关键就是requestType要指定为`form`，这2点我摸索了挺久。
  
- 下载文件

  用window.open或者a标签链接到我们的download接口接口:
  
  `window.open(`${CONFIG.URL}/oss/download?filepath=${record.key}`)`

- 全部代码

```js
import {PageContainer} from "@ant-design/pro-layout";
import {Button, Card, Col, Divider, Form, Input, Modal, Row, Table, Upload} from "antd";
import {InboxOutlined, PlusOutlined} from "@ant-design/icons";
import {CONFIG} from "@/consts/config";
import {useEffect, useState} from "react";
import {connect} from 'umi';
import moment from "moment";

const Oss = ({loading, dispatch, gconfig}) => {

  const [form] = Form.useForm();
  const {ossFileList, searchOssFileList} = gconfig;
  const [visible, setVisible] = useState(false);
  const [value, setValue] = useState('');

  const onDeleteFile = key => {
    dispatch({
      type: 'gconfig/removeOssFile',
      payload: {
        filepath: key
      }
    })
  }

  const listFile = () => {
    dispatch({
      type: 'gconfig/listOssFile',
    })
  }

  const columns = [
    {
      title: '文件路径',
      key: 'key',
      dataIndex: 'key',
      render: key => <a href={`${CONFIG.URL}/oss/download?filepath=${key}`} target="_blank">{key}</a>
    },
    {
      title: '大小',
      key: 'size',
      dataIndex: 'size',
      render: size => `${Math.round(size / 1024)}kb`
    },
    {
      title: '更新时间',
      key: 'last_modified',
      dataIndex: 'last_modified',
      render: lastModified => moment(lastModified * 1000).subtract(moment().utcOffset() / 60 - 8, 'hours').format('YYYY-MM-DD HH:mm:ss')

    },
    {
      title: '操作',
      key: 'ops',
      render: (record) => <>
        <a onClick={() => {
          window.open(`${CONFIG.URL}/oss/download?filepath=${record.key}`)
        }}>下载</a>
        <Divider type="vertical"/>
        <a onClick={() => {
          onDeleteFile(record.key);
        }}>删除</a>
      </>
    },
  ]
  const normFile = (e) => {
    if (Array.isArray(e)) {
      return e;
    }
    return e && e.fileList;
  };

  const onUpload = async () => {
    const values = await form.validateFields();
    const res = dispatch({
      type: 'gconfig/uploadFile',
      payload: values,
    })
    if (res) {
      setVisible(false)
      setValue('')
      listFile();
    }
  }

  useEffect(() => {
    if (value === '') {
      dispatch({
        type: 'gconfig/save',
        payload: {searchOssFileList: ossFileList}
      })
    } else {
      dispatch({
        type: 'gconfig/save',
        payload: {searchOssFileList: ossFileList.filter(v => v.key.toLowerCase().indexOf(value.toLowerCase()) > -1)}
      })
    }
  }, [value])


  useEffect(() => {
    listFile();
  }, [])


  return (
    <PageContainer title="OSS文件管理" breadcrumb={false}>
      <Card>
        <Modal width={600} title="上传文件" visible={visible} onCancel={() => setVisible(false)} onOk={onUpload}>
          <Form form={form} {...CONFIG.LAYOUT}>
            <Form.Item label="文件路径" name="filepath"
                       rules={[{required: true, message: '请输入文件要存储的路径, 目录用/隔开'}]}>
              <Input placeholder="请输入文件要存储的路径, 目录用/隔开"/>
            </Form.Item>
            <Form.Item label="文件" required>
              <Form.Item name="files" valuePropName="fileList" getValueFromEvent={normFile} noStyle
                         rules={[{required: true, message: '请至少上传一个文件'}]}>
                <Upload.Dragger name="files" maxCount={1}>
                  <p className="ant-upload-drag-icon">
                    <InboxOutlined/>
                  </p>
                  <p className="ant-upload-text">点击或拖拽文件到此区域上传🎉</p>
                </Upload.Dragger>
              </Form.Item>
            </Form.Item>
          </Form>
        </Modal>
        <Row gutter={[8, 8]} style={{marginBottom: 12}}>
          <Col span={6}>
            <Button type="primary" onClick={() => setVisible(true)}><PlusOutlined/>添加文件</Button>
          </Col>
          <Col span={12}/>
          <Col span={6}>
            <Input placeholder="输入要查找的文件名" value={value} onChange={e => {
              setValue(e.target.value);
            }}/>
          </Col>
        </Row>
        <Table rowKey={record => record.key} dataSource={searchOssFileList} columns={columns}
               loading={loading.effects['gconfig/listOssFile']}/>
      </Card>
    </PageContainer>
  )
}

export default connect(({loading, gconfig}) => ({loading, gconfig}))(Oss);

```



  今天的内容就介绍到这里，如今我们已经可以通过oss去管理我们的文件了，那我们要怎么运用呢？
  
  下一节给大家介绍，http请求支持`文件上传`功能。
  
  

