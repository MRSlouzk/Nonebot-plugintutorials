# 消息的处理

### NB平台扮演的角色

以OneBot适配器举例，在机器人后台消息接收时会经过如下流程：首先gocqhttp会从QQ服务器获取消息，然后通过反向WebSocket（如果用的是的话）将消息传输给正在运行的Onebot平台，平台将消息处理后发出响应再发还给gocqhttp端，最后gocqhttp根据响应向QQ服务器发出消息，这就是消息处理的整个流程。

简单来说gocqhttp就是沟通Nonebot和QQ服务器之间通信的桥梁，也就是眼睛耳朵鼻子或者说身体的各个部位，而Nonebot就是相当于"大脑"，负责处理接收到的信息并且发送指令给手、脚这种身体部位让他们行动，这就是形象化的过程。

### 由关键字触发体验简单的消息处理

首先先看一段代码

```python
from nonebot import on_keyword
from nonebot.adapters.onebot.v11 import Message

word=on_keyword({"你是谁"})

@word.handle()
async def _():
    await word.finish(Message("我是bot!"))
```

这段代码实现的功能是：如果有人发送（无论是与机器人的私聊还是群聊里）任一包含"你是谁"这三个关键字的消息的时候，机器人会回复消息：“我是bot”。功能是很简单，但是里面的基础知识点不少，下面逐一分析一下

------



```python
from nonebot import on_keyword
from nonebot.adapters.onebot.v11 import Message
```

这段经典的Import代码的意思就是从nonebot库中引入`on_keyword`方法，再从Onebot V11库中引入`Message`方法。这两个方法具体用法我们之后再谈，接下来先聊聊这个import。

Nonebot2中的很多功能都需要通过import导入后才能使用，虽然你也可以通过无脑`from nonebot import *`来一劳永逸，但不保证这样会不会出一些奇奇怪怪的问题，所以我的建议是需要什么方法就导入什么。**那么我怎么知道从哪些库里引入我想要的函数呢？**这里我列举了一些常见库里包含的常见函数，可以供大家参考

| 库                          | 常用函数/方法                                                | 说明       |
| --------------------------- | ------------------------------------------------------------ | ---------- |
| nonebot                     | on_message，on_notice，on_request，on_keyword，on_command，on_regex | 基础库     |
| nonebot.config              | Config                                                       | 配置文件库 |
| nonebot.matcher             | Match                                                        | 时间响应器 |
| nonebot.params              | Arg，State，CommandArg，RegexMatched                         | 参数库     |
| nonebot.permission          | SUPERUSER，Permission                                        | 权限库     |
| nonebot.log                 | logger，default_format                                       | 日志库     |
| nonebot.adapters.onebot.v11 | Bot，Message，MessageSegment，Event，PRIVATE，GROUP等        | Onebot库   |

(注：OneBot V11的库可以通过`nonebot.adapters.onebot.v11`直接导入，也可以用诸如`nonebot.adapters.onebot.v11.message`等对应方式导入，详见[官方文档](https://onebot.adapters.nonebot.dev/docs/api/v11))

------

```python
word=on_keyword({"你是谁"})
```

这行代码的含义是“<u>注册一个名为word，类型为on_keyword的事件响应器</u>”，那么这句话是什么意思呢？

首先是“注册”，注册其实类似于C语言里面的函数声明，函数必须在声明后才可以使用，事件响应器（具体是什么不急着谈）也是如此。简单来说你想建个房子，但你建房子之前必须要指定一块地说我要在这里建房子才行，而不能直接就建房子，响应器注册就是指定一个变量（土地）来建房子（响应器）。并且这里面的“word”也是类似于“函数的名字”一样的存在，所以可以在符合Python变量命名法的前提下随便取名，这个变量在之后使用事件响应器时会用到。

之后是`on_keyword`，前面说过on_keyword是NoneBot库中的一个方法，至于方法这个词具体是什么意思请去看《面向对象编程》。在这你可以简单理解成一个函数就行，这个函数会返回一个事件响应器，那它有什么参数呢？先来看看文档中的说明：

`on_keyword(keywords, rule=..., *, permission=..., handlers=..., temp=..., priority=..., block=..., state=...)`*<u>注册一个消息事件响应器，并且当消息纯文本部分包含关键词时响应</u>*

第一个参数是keywords，类型为集合(Set)，传入的是用来触发该响应器工作的关键词集合(用{}包起来的)。

第二个参数是rule，传入的是bool类型规则函数或者Rule参数。

第三个参数是permission，传入的是bool类型的权限函数或者Permission参数。

第四个参数是handlers，传入的参数是事件响应器列表。

第五个参数是temp，传入的是bool类型参数，若为True则此响应器只使用一次。

第六个参数是priority，传入的是int型参数，用于设定响应器的优先级。注意！此参数越大优先级越低。

第七个参数是block，传入的是bool类型的参数，若为True则不会注册优先级小于此响应器的响应器，相当于消息被阻断在这里了。

最后一个参数是state，传入的是字典类型参数。

目前需要了解的只有keywords，permission，priority和block这四个参数，其他参数我们之后才会讲到，所以可以先放着。