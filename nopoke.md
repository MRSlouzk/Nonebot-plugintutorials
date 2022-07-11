# 实例1 不要乱戳！

------

难度：★★

### 功能

当bot被戳一戳时，如果对方在允许列表里，则发送蹭蹭的图片，若不在则@戳的人并且让他不要再戳了。

### 要点

- 戳一戳检测
- 判断是否在列表里
- 发送图片

### 实现

##### 戳一戳检测

```python
from nonebot import on_notice
from nonebot.adapters.onebot.v11 import PokeNotifyEvent

def _check(event: PokeNotifyEvent):
    return event.user_id==event.self_id

poke=on_notice(rule=_check)

@poke.handle()
async def _(event: PokeNotifyEvent):
    pass
```

检测“戳一戳”事件调用的是Nonebot中的[on_notice](https://v2.nonebot.dev/docs/api/plugin/on#on_notice)(通知事件)。当bot接收到一个`notice`事件时，会先根据`rule`参数当中的规则函数`_check`判断是否为戳一戳事件*(注：因为规则函数传参已经规定了必须是`PokeNotifyEvent`(戳一戳事件)，所以如果传入的不是这个参数默认判定为False)*，然后在函数体里面判断被戳的人是不是bot自己，如果是的话才会返回True。

##### 判断是否在列表里

```python
agree_list=[123,112]

@poke.handle()
async def _(event: PokeNotifyEvent):
    if(event.user_id in agree_list):
        pass
```

这个是最简单的，熟悉Python的列表(List)类型变量的操作就行。`in`关键字用于判断列表中是否有某一变量(必须是同一种类型！比如`1 in ["1"]`会返回False)。

##### 发送图片

```python
await poke.finish(MessageSegment.record(file=r"file:///E:\Code\plugins\poke\resources\1.mp3"))
```

