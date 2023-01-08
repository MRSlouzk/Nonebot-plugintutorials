# 如何调用Gocq-Api

本节我们讲述如何在Nonebot2的插件中调用gocqhttp的API

### [Gocq-Api](https://docs.go-cqhttp.org/api/#%E5%9F%BA%E7%A1%80%E4%BC%A0%E8%BE%93)

Gocq当中有70+种API，其中有一部分已经被Onebot协议所实现，例如`send_msg`、`send_group_msg`等，可以直接通过`bot.`来调用这些API。全部已实现API参看[Onebot文档](https://github.com/botuniverse/onebot-11/blob/master/api/public.md)。不过本教程并不是讲如何通过Onebot的实现去调用gocq的api，而是直接访问。

其实方法很简单，代码如下

```python
from nonebot import on_command
from nonebot.adapters.onebot.v11 import Bot, GroupMessageEvent

sign = on_command("打卡")

@sign.handle()
async def sig(event: GroupMessageEvent, bot: Bot):
    await bot.call_api("send_group_sign", group_id=event.group_id)
    await sign.finish("打卡成功!")
```

也可以

```python
from nonebot import on_command
from nonebot.adapters.onebot.v11 import Bot, GroupMessageEvent

sign = on_command("打卡")

@sign.handle()
async def sig(event: GroupMessageEvent, bot: Bot):
    data = {"group_id": event.group_id}
    await bot.call_api("send_group_sign", data=data)
    await sign.finish("打卡成功!")
```

上述代码首先通过依赖注入的方式获取Bot类，之后调用Bot类下的`call_api`方法，即可实现API的调用。以下是源码当中对`call_api`的注释信息

> ```
> 调用 OneBot 协议 API。
> 
> 参数:
>     api: API 名称
>     data: API 参数
> 
> 返回:
>     API 调用返回数据
> 
> 异常:
>     nonebot.adapters.onebot.exception.NetworkError: 网络错误
>     nonebot.adapters.onebot.exception.ActionFailed: API 调用失败
> ```

可以看到第一个参数传入的是API的名字，类型为str。API的名字以及传参方法在[gocq API文档](https://docs.go-cqhttp.org/api/#%E5%8F%91%E9%80%81%E7%A7%81%E8%81%8A%E6%B6%88%E6%81%AF)当中均可找到。如上例当中的`send_group_sign`为gocq的一个API，用于群签到。

**那么该如何传入API所需要的参数呢？**

方法也很简单，直接在`call_api`内使用`所需参数名=所需参数`的方式传入，上例当中的`group_id=event.group_id`就是如此，因为按文档所述`send_group_sign`需要传入`group_id`，即 群号 参数。当然了，也可以构造参数字典来进行参数传递，见上述第二个例子。

其它gocq API的使用方法完全可以举一反三，不再赘述。