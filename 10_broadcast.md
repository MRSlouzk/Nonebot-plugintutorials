# 实例8 全群广播

难度：★★

作者：[XieXiLin](https://github.com/XieXiLin3)

### 功能

读取bot加的所有群聊，并且在管理员发送全局广播指令后将管理员的消息发到所有群。

### 要点

- 读取群列表
- 遍历所有群并发送广播消息

### 实现

> 本教程有待进一步完善。

#### 讲解

##### 获取群列表

可通过 `Bot` 下的 `get_group_list()` 获取机器人已在的群列表。

> 注意: 列表内的参数还是 `get_group_info()` 批量获取到的群信息，你需要通过提取其中的 `group_id` 获取群号。

```python
from nonebot import on_command
from nonebot.permission import SUPERUSER
from nonebot.params import CommandArg
from nonebot.log import logger
from nonebot.adapters.onebot.v11 import Message, MessageEvent, MessageSegment, Bot

bc=on_command("broadcast", aliases={"bc", "广播", "全局广播", "notice"}, permission=SUPERUSER)

@bc.handle()
async def bc_handle(event: MessageEvent, bot: Bot):
    logger.info(await bot.get_group_list())
```

##### 遍历所有群并发送广播消息

获取群列表之后通过 `for` 循环将 `i` 逐次变换为每个群的信息，再提取 `group_id` 作为群号即可。

> Tips: 在这里 `args` 为 `Message` 类，所以可直接放入 `message` 参当消息段发送。
> （碰巧看源码看到了，顺手说一声）

```python
from nonebot import on_command
from nonebot.permission import SUPERUSER
from nonebot.params import CommandArg
from nonebot.adapters.onebot.v11 import Message, MessageEvent, MessageSegment, Bot

@bc.handle()
async def bc_send(event: MessageEvent, bot: Bot, args: Message=CommandArg()):
    for i in await bot.get_group_list():
        await bot.send_group_msg(group_id=i['group_id'], message=args)
        await bc.finish(MessageSegment.reply(event.message_id) + MessageSegment.text(f"成功发送。"))
```

#### 最终实现

```python
from nonebot import on_command
from nonebot.permission import SUPERUSER
from nonebot.params import CommandArg
from nonebot.adapters.onebot.v11 import Message, MessageEvent, MessageSegment, Bot

cancel_word=["取消", "取消发送", "cancel", "Cancel"]

bc=on_command("broadcast", aliases={"bc", "广播", "全局广播", "notice"}, permission=SUPERUSER)
@bc.handle()
async def bc_wait(event: MessageEvent, bot: Bot):
    await bc.pause(MessageSegment.reply(event.message_id) + MessageSegment.text("请发送需要广播的消息"))

@bc.handle()
async def bc_send(event: MessageEvent, bot: Bot):
    if event.message.extract_plain_text() in cancel_word:
        await bc.finish(MessageSegment.reply(event.message_id) + MessageSegment.text("已取消发送。"))
    else:
        count=0
        for i in await bot.get_group_list():
            await bot.send_group_msg(group_id=i['group_id'], message=event.message)
            count+=1
        await bc.finish(MessageSegment.reply(event.message_id) + MessageSegment.text(f"成功发送到 {count} 个群。"))
```

在这里我们使用了 [`pause()`](https://nb2.baka.icu/docs/tutorial/plugin/matcher-operation#pause) ，可以分开发送，更清晰明了。

`permission=SUPERUSER` 即为只有 `SUPERUSER` 才可使用命令。