# 实例2 入群审批

难度：★★★

作者：[MRSlouzk](https://github.com/MRSlouzk)

> [Nonebot2插件编写教程 EP5-处理加群请求](https://www.bilibili.com/video/BV1WW4y1i7As) *([大家的谢谢](https://space.bilibili.com/495468749))*

### 功能

有人发送入群申请时如果验证信息答案在允许列表里，则同意，否则在群里发送消息提醒。

### 要点

- 如何检测有人发了加群申请
- 获取验证信息
- 同意加群申请

### 实现

##### 检测有人发了加群申请

```python
from nonebot.adapters.onebot.v11 import GroupRequestEvent, Event
from nonebot import on_request

def _check(event: Event):
    return isinstance(event, GroupRequestEvent)

request=on_request(rule=_check)
```

代码如上，检测加群申请使用的是`on_request`，即[请求事件](https://github.com/botuniverse/onebot-11/blob/master/event/request.md)。Onebot当中的请求事件只有两种：加群申请(GroupRequestEvent)与加好友申请(FriendRequestEvent)，两者从字面上已经能读懂含义，在此不再说明。

上述代码当中，`_check`规则函数用于检测事件是否为**加群申请**事件而非**加好友申请**事件，其中`isinstance()`函数用于判断某个对象是否为一个已知类，比如上述代码的`isinstance(event, GroupRequestEvent)`就是判断传入的`event`参数是否为`GroupRequestEvent`，如果是就返回`True`，规则函数通过，以此过滤掉好友申请，方便区分。

##### 获取验证信息

获取验证信息是通过解析`GroupRequestEvent`当中的数据获取的，首先通过下段代码获取请求中的信息

```python
async def _(bot: Bot,event: GroupRequestEvent):
    if(event.group_id in group_to_use):
        raw = json.loads(event.json())
```

可以看到，raw即是通过`event.json`获取到的json数据转化成的变量，类型为字典(Dict)。里面包含很多字段，包括但不仅限于收到申请的时间(time)、收到申请消息的bot的qq(self_id)、群号(group_id)、请求子类型(sub_type)等，具体见[文档](https://github.com/botuniverse/onebot-11/blob/master/event/request.md)。而我们所需要的验证信息就是其中的"comment字段"，可以通过`comment = raw['comment']`提取出来。**特别注意！！！验证消息提取出来后并不单纯是申请人填入文字框中的信息**，而是<u>"问题：XXX 答案：YYY"</u>的形式，其中YYY才是我们真正想要的内容。可以通过`word = re.findall(re.compile('答案：(.*)'), comment)[0]`来提取(正则表达式yyds)，至此我们就拿到了验证信息。

![request_app1](https://github.com/MRSlouzk/Nonebot-plugintutorials/blob/main/imgs/request_app1.png?raw=true)

而要验证验证信息是否正确也很简单，原理跟第一个实例很像，直接检测列表中是否有这个字符串(验证信息)就行，即使用`in`方法。

##### 同意加群申请

同意加群申请调用的是Onebot中Bot类提供的一个方法(API)，名为`set_group_add_request`，这个方法有以下参数

`flag`，类型为字符串，相当于网络请求中的`token`令牌，每一个加群请求都对应唯一一个flag值。还不懂的话就记住这东西就相当于钥匙就行，处理加群申请看作一把锁，要成功处理必须使用这个"钥匙"来开锁。`flag`是QQ服务器直接提供的，不需要自己生成。

`sub_type`，事件子类型，有"add"和"invite"两种，分别对应"主动加群"和"邀请加群"。该参数必须与请求当中的sub_type相同，不然无法执行处理申请的方法。

`approve`，是否同意加群申请，默认为True，即同意加群，False为不同意。

`reason`，拒绝加群的理由，仅在`approve`为False时生效。

代码实现*(不必要换行,换行只是让参数看起来更清晰)*

```python
bot.set_group_add_request(
                    flag=flag, //填入的是json数据中提取到的flag值
                    sub_type=sub_type,
                    approve=True,
                    reason=" ", //选填
                )
```

------

最后来看一下完整代码

```python
from nonebot import on_request
from nonebot.log import logger
from nonebot.adapters.onebot.v11 import Bot, GroupRequestEvent
import re
import json

notice=on_request(priority=1)

group_to_use=[] //填入需要开启自动审批的群
approve_message_1=[""] //填入想要的字符串列表

@notice.handle()
async def _(bot: Bot,event: GroupRequestEvent):
    if(event.group_id in group_to_use):
        raw = json.loads(event.json())
        gid = str(event.group_id)
        flag = raw['flag']
        sub_type = raw['sub_type']
        if sub_type == 'add':
            comment = raw['comment']
            word = re.findall(re.compile('答案：(.*)'), comment)[0]
            uid = event.user_id
            if(str(word) in approve_message_1):
                logger.info(f'同意{uid}加入群 {gid},验证消息为 “{word}”') #控制台日志输出加群信息!logger类的使用参见																		2.5章(未做)
                await bot.set_group_add_request(
                    flag=flag,
                    sub_type=sub_type,
                    approve=True,
                    reason=" ",
                )
            else:
                await notice_1.finish("有人要加群!但答案不对呀!")
    await notice_1.finish()
```

