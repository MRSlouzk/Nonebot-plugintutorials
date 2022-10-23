# 实例10 定时广播

难度：★★★

作者：[ubby](https://github.com/ubby)

### 功能

每天定时六点发一条信息

### 要点

- 如何让nonebot导入并识别apscheduler这个插件
- 设置时间

### 实现

> 本教程有待进一步完善。

#### 讲解

##### 如何让nonebot导入并识别apscheduler这个插件

可通过 `nonebot` 下的 `require` 来导入插件

> 注意: require导入的插件格式是这样：`nonebot_plugin_xxxx`

```python
timing = require("nonebot_plugin_apscheduler").scheduler
```

##### 设置时间

要注意定时器中有多个参数，比如hour，minute，second等，分别是时，分，秒
我们要在早晨六点发送也就是
hour = 06
minute = 00
second = 00

```python
@timing.scheduled_job("cron", hour='06', minute = '00' , second = '00' ,id="dingshi")
async def _():
    #这里是获取bot对象
    (bot,) = nonebot.get_bots().values()
    await bot.send_msg(
        message_type="group",
        group_id=768176998,
        message="起床了！！！"
    )
```

#### 最终实现

```python
import nonebot
from nonebot import require

# 设置一个定时器
timing = require("nonebot_plugin_apscheduler").scheduler
@timing.scheduled_job("cron", hour='06', minute = '00' , second = '00' ,id="dingshi")
async def _():
    #这里是获取bot对象
    (bot,) = nonebot.get_bots().values()
    await bot.send_msg(
        message_type="group",
        group_id=768176998,
        message="起床了！！！"
    )

```
