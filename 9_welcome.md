# 实例7 入群欢迎

难度：★

作者：

### 功能

当有新成员入群时发送欢迎语。

### 要点

- 入群事件判定

### 实现

#### 讲解

##### 入群事件判定

加群事件为 [`GroupIncreaseNoticeEvent()`](https://onebot.adapters.nonebot.dev/docs/api/v11/event#GroupIncreaseNoticeEvent) ，是一个 [`on_notice()`](https://nb2.baka.icu/docs/api/plugin/on#on_notice) (通知事件) 。

我们可以使用 `isinstance()` 判定一个事件是不是某个事件类型。

```python
from nonebot import on_notice
from nonebot.adapters.onebot.v11 import Event, GroupIncreaseNoticeEvent

def _rule(event: Event):
    return isinstance(event, GroupIncreaseNoticeEvent)

join=on_notice(rule=_rule)
@join.handle()
async def group_increase_handle():
    await join.finish()
```

最后我们向群里返回一条欢迎消息就完成了。

#### 代码展示

```python
from nonebot import on_notice
from nonebot.adapters.onebot.v11 import Event, GroupIncreaseNoticeEvent, MessageSegment

def _rule(event: Event):
    return isinstance(event, GroupIncreaseNoticeEvent)

join=on_notice(rule=_rule)
@join.handle()
async def group_increase_handle(event: GroupIncreaseNoticeEvent):
    await join.finish(MessageSegment.text("欢迎新成员 ") + MessageSegment.at(event.user_id) + MessageSegment.text(" 加入我们的大家族!"))
```