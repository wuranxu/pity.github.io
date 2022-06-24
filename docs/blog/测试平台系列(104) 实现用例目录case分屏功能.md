> 大家好~我是`米洛`！<br/>
> 我正在从0到1打造一个开源的接口测试平台, 也在编写一套与之对应的`教程`，希望大家多多支持。<br/>
> 欢迎关注我的公众号`米洛的测开日记`，获取最新文章教程! 

### 回顾

  上一节，咱们编写了`系统设置`页面的相关功能，这一节呢，我们来优化一下用例目录和用例页面的显示。
  
  我们来研究下怎么实现一个`分屏功能`。
  
### 什么是分屏

  我们现在的`用例列表页`就不支持分屏，只因左右侧`卡片`的宽度都是相对固定的，也就是不能左右拖拽放大/缩小的。
  
![](https://static.pity.fun/picture/2022-1-23/1642912473432-image.png)

  如图所示，假设现在我的目录名字很长，那可能`显示效果`就会很差。（因为左边宽度不够用呀）
  
  再来看看分屏了的组件，我们用一个gif来展示:
  
![](https://static.pity.fun/picture/2022-1-23/1642913857667-gif3.gif)

### 组件调研

  起初踩了不少坑，也不知道该怎么在`github`搜索对应的组件，当然，要自己写是不可能的（毕竟不会）。
  
  后来在ant.design官网看到了一些推荐的组件:
  
![](https://static.pity.fun/picture/2022-1-23/1642913948880-image.png)

  **这就是生态的力量吗！**
  
  接着我们从里面找到了不起眼的分屏组件:
  
![](https://static.pity.fun/picture/2022-1-23/1642913980675-image.png)

### 看看官网的demo

![](https://static.pity.fun/picture/2022-1-23/1642914016463-image.png)

  官网的demo给的真的是，相当的`粗糙`，连import都没有写。这不禁让我想起某个star第一的Java操作excel的库，连maven/gradle依赖都没有给出来。
  
  简单讲下他的api:
  
- split

  分割的模式，有vertical和horizontal，即垂直和水平，即左右分屏和上下分屏，我们这里需要采用vertical。
  
- minSize

  最小尺寸，一般以左侧的div为主。
  
- defaultSize

  默认尺寸
  
- maxSize
  
  最大尺寸，也就是左边div可以拖到的最大宽度。
  
### 注意事项

  在图中，可以看到SplitPane包裹的2个div进行了分屏操作，那我们要对`antd`的Col进行同样的操作可行吗？
  
  博主亲自试了一下，不可以，这样会导致`页面错乱`。所以我们还是按照他的意思，使用div标签。
  
### 运用到pity中

1. 安装依赖

```shell
npm install --save react-split-pane
```

  我们主要是想让case和目录进行分离，所以我们找到目录那块jsx: src/pages/ApiTest/TestCaseDirectory.jsx调整以下内容:

2. 新增引用

```
import SplitPane from 'react-split-pane';

```

3. 修改case和用例视图


![](https://static.pity.fun/picture/2022-1-23/1642916657637-image.png)


  找到此处的Row，并在下面加入SplitPane组件，把2个Col的组件改为`div`。
  
  我们看下效果:
  
![](https://static.pity.fun/picture/2022-1-23/1642916738716-image.png)

  可以看到样式变了，但是实际上并未生效。
  
### 排查问题

  所以这就是我说官网的demo不是太好，因为我们还需要额外写一些css样式。
  
- 编写src/pages/ApiTest/TestCaseDirectory.less文件(没有则新建)

```css
.Resizer {
  background: #000;
  opacity: 0.2;
  z-index: 1;
  -moz-box-sizing: border-box;
  -webkit-box-sizing: border-box;
  box-sizing: border-box;
  -moz-background-clip: padding;
  -webkit-background-clip: padding;
  background-clip: padding-box;
}

.Resizer:hover {
  -webkit-transition: all 2s ease;
  transition: all 2s ease;
}

.Resizer.horizontal {
  height: 11px;
  margin: -5px 0;
  border-top: 5px solid rgba(255, 255, 255, 0);
  border-bottom: 5px solid rgba(255, 255, 255, 0);
  cursor: row-resize;
  width: 100%;
}

.Resizer.horizontal:hover {
  border-top: 5px solid rgba(0, 0, 0, 0.5);
  border-bottom: 5px solid rgba(0, 0, 0, 0.5);
}

.Resizer.vertical {
  width: 11px;
  margin: 0 -5px;
  border-left: 5px solid rgba(255, 255, 255, 0);
  border-right: 5px solid rgba(255, 255, 255, 0);
  cursor: col-resize;
}

.Resizer.vertical:hover {
  border-left: 5px solid rgba(0, 0, 0, 0.5);
  border-right: 5px solid rgba(0, 0, 0, 0.5);
}
.Resizer.disabled {
  cursor: not-allowed;
}
.Resizer.disabled:hover {
  border-color: transparent;
}

```

- 引入

![](https://static.pity.fun/picture/2022-1-23/1642916850097-image.png)


### 最终效果

![](https://static.pity.fun/picture/2022-1-23/1642918494628-gif3.gif)

  有了这个功能之后，我们就可以轻松编写`类似`的组件了，这样就可以轻松分割其他页面的div啦。
  
  

