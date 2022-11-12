# 实例9 猜数字

难度：★★★★

作者：[XieXiLin](https://github.com/XieXiLin3)

### 功能

当一个人发送`/猜数字`时，系统随机生成一个数字，之后此人回答。bot会提示"大了"或者"小了"，直至猜对。

### 要点

- 持续性会话
- 由 QQ 和时间生成随机数
- 时间戳的保存

### 实现

#### 讲解

##### 持续性会话

我们可以利用 [`pause()`](https://nb2.baka.icu/docs/tutorial/plugin/matcher-operation#pause) 进行等待回复。

> 官方解释:
> 向用户回复一条消息（可选），并立即结束当前事件处理依赖并等待接收一个新的事件后进入下一个事件处理依赖。
> 类似于 receive 的行为但可以根据事件来决定是否接收新的事件。

如果答案不对，我们可以用 [`reject()`](https://nb2.baka.icu/docs/tutorial/plugin/matcher-operation#reject) 进行答案拒绝，然后等待下一个答案发送再进行此事件的重试。

> 官方解释:
> 向用户回复一条消息（可选），并立即结束当前事件处理依赖并等待接收一个新的事件后再次执行当前事件处理依赖。
> 通常用于拒绝当前 receive 接收的事件或 got 接收的参数（如：不符合格式或标准）。

```python
from nonebot import on_command
from nonebot.adapters.onebot.v11 import MessageEvent, MessageSegment

num = on_command("guessnumber", aliases={"guessnum", "gusnum", "gnum", "猜数字"})

@num.handle()
async def num_handle_start(event: MessageEvent):
    await num.pause(MessageSegment.text("Please Start"))

@num.handle()
async def num_handle_guess(event: MessageEvent):
    await num.send(MessageSegment.text("Your Message Is:"))
    await num.send(event.message)
    if event.message.extract_plain_text() is "cancel":
        await num.finish(MessageSegment.text("Exit"))
    else:
        await num.reject(MessageSegment.text("Is Wrong, Try Again"))
```

##### 由 QQ 和时间生成随机数

用 `event.user_id` 获取用户的 QQ 号，然后通过时间戳搭配合成一个 `seed` 进行随机数生成。

可以用 `random` 库的 `seed()` 进行生成。

```python
from nonebot import on_command
from nonebot.adapters.onebot.v11 import MessageEvent, MessageSegment
import random

num = on_command("guessnumber", aliases={"guessnum", "gusnum", "gnum", "猜数字"})

@num.handle()
async def num_handle_start(event: MessageEvent):
    random.seed(event.user_id)
    await num.pause(MessageSegment.text("Please Start"))

@num.handle()
async def num_handle_guess(event: MessageEvent):
    await num.send(MessageSegment.text("Your Message Is:"))
    await num.send(event.message)
    if event.message.extract_plain_text() is random.randint(1, 1000):
        await num.finish(MessageSegment.text("Exit"))
    else:
        await num.reject(MessageSegment.text("Is Wrong, Try Again"))
```

> 代码以最终 `代码展示` 环节的代码为准。

##### 时间戳保存

但是你如果自己先测试的时候发现，`seed` 每次重新响应都会变化，这个时候就需要找方法暂存 `seed` 了。

我在这里使用的是 `json` 储存。

可以直接保存 `seed` 方便处理（下面的代码正是如此）。

```python
from nonebot import on_command
from nonebot.params import CommandArg
from nonebot.adapters.onebot.v11 import MessageEvent, Message, MessageSegment
import random, json, time

num=on_command("num")
@num.handle()
async def num_handle(event: MessageEvent, args: Message = CommandArg()):
    num_seed={"user": event.user_id, "seed": event.user_id + int(time.time())}
    with open("num.json", "wb", encoding="utf-8") as f:
        f.write(json.dumps(num_seed, ensure_ascii=False, indent=2))
    await num.reject(MessageSegment.reply(event.message_id) + MessageSegment.text(str(random.randint(1, 1000))))

@num.handle()
async def num_handle_for(event: MessageEvent):
    with open("num.json", "r", encoding="utf-8") as f:
        num_seed_reads=json.loads(f.read())
    num_seed=num_seed_reads['seed']
    if event.message.extract_plain_text() is str(num_seed):
        await num.finish(MessageSegment.reply(event.message_id) + MessageSegment.text("Good, it's right! Goodbye!"))
    else:
        await num.reject(MessageSegment.reply(event.message_id) + MessageSegment.text(str(random.randint(1, 1000))))
```

#### 代码展示

```python
from nonebot import on_command
from nonebot.params import CommandArg
from nonebot.adapters.onebot.v11 import MessageEvent, Message, MessageSegment
import random, json, time


num=on_command("num")
@num.handle()
async def num_handle(event: MessageEvent, args: Message = CommandArg()):
    num_seed={"user": event.user_id, "seed": event.user_id + int(time.time())}
    with open("num.json", "wb", encoding="utf-8") as f:
        f.write(json.dumps(num_seed, ensure_ascii=False, indent=2))
    await num.reject(MessageSegment.reply(event.message_id) + MessageSegment.text(str(random.randint(1, 1000))))

@num.handle()
async def num_handle_for(event: MessageEvent):
    with open("num.json", "r", encoding="utf-8") as f:
        num_seed_reads=json.loads(f.read())
    random.seed(num_seed_reads['seed'])
    if event.message.extract_plain_text() is str(random.randint(1, 1000)):
        await num.finish(MessageSegment.reply(event.message_id) + MessageSegment.text("Good, it's right! Goodbye!"))
    elif int(event.message.extract_plain_text()) > int(random.randint(1, 1000)):
        await num.reject(MessageSegment.reply(event.message_id) + MessageSegment.text("Too Big")
    else:
        await num.reject(MessageSegmeny.reply(event.message_id) + MessageSegment.text("Too Small"))
```

有个不好的地方，要是别人也用的话种子就会变，可以等待未来我想通了再修改这个，或者等待你的 `Pull Request` 。
