# 实例2 入群审批

难度：★★★

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

上述代码当中，`_check`规则函数用于检测事件是否为**加群申请**事件而非**加好友申请**事件，其中`isinstance()`函数用于判断某个对象是否为一个已知类，比如上述代码的`isinstance(event, GroupRequestEvent)`就是判断传入的`event`参数是否为`GroupRequestEvent`，如果是就返回`True`，规则函数通过。

##### 获取验证信息