# Matcher响应器类的使用

> 之前我们讲到的响应器创建后一般使用的都是装饰器标注、连续性对话、消息发送与结束等操作，那除此以外响应器还能够调节哪些参数？
>
> 用到响应器的函数前都要进行标注，若响应器处理函数当中有子函数也需要调用响应器，那么该如何？并且只有一个函数调用还好，多个响应器的处理函数同时调用一个子函数就会出问题，即不知道是哪个响应器调用的，我们该怎么解决这个问题呢？

### 一、作为参数传递

先看一个实际需求

```python
from nonebot import on_command
from nonebot.adapters.onebot.v11 import Message

add_data = on_command("添加数据")
del_data = on_command("删除数据")

@add_data.handle()
async def _():
    permission_check() #权限效验以及处理
    #下接数据处理函数

@del_data.handle()
async def _():
    permission_check()
    #下接数据处理函数
```

上述代码实现的是数据的添加与删除，当然具体的代码可以有多种实现，这里就省略了。不过此代码的重点是当中的`permission_check()`函数，这是一个两个命令共有的，进行权限检测和处理的函数，里面具体内容我们下面会讲。

这时候有人会问：权限管理的话`Permission`也可以，`Rule`也可以，写在响应器创建的时候不更好？确实是这样，但是这样做会有个问题：**你怎么让那个人知道他的权限不足？**换句话说就是怎么让Bot发出消息提醒他权限不足，这点使用`Permission`是不好做的。<u>*(理论上来说`Rule`里可以实现消息的发送，但本段代码只是个演示用的示例，不要纠结这么多问题)*</u>。

那还有人会说：这个不简单吗，直接在函数里面写个权限组判定然后输出指定内容不就完了？话是这么说，但是如果你的响应器函数越来越多，每次都要写这部分内容，会让代码出现大量的冗余，之后的维护会变得非常困难。因此我们就需要把这部分内容封装成一个函数。

**<u>那这个函数如何获取到原事件的响应器</u>**？这时候就需要使用到`Matcher`类了，使用方法如下

```python
async def permission_check(event: GroupMessageEvent, matcher: Matcher):
    if(event.user_id not in admin_list): #此处可自己根据情况定义,懂意思就行
        await matcher.finish("您的权限不足,无法继续操作")
```

传入的matcher参数使用方法与一般的响应器用法一模一样，类比响应器就行，在此不再赘述。

### 二、响应器属性

事件响应器拥有非常多的属性和方法，常用的属性/方法如下

| 名字               | 作用            |
| ------------------ | --------------- |
| get_arg            | 获取Got消息     |
| get_receive        | 获取Receive消息 |
| permission_updater | 权限提升        |
| state              | 响应器状态      |
| expire_time        | 响应器存续时间  |

下面是存续时间的一个用例。

```python
import datetime

from nonebot import on_command

sch = on_command("定时结束")
@sch.handle()
async def _():
    sch.expire_time = datetime.datetime.now() + datetime.timedelta(seconds=100)

@sch.got("a") #此处暂时不用管,之后会讲。理解为等待发送者发送下一个消息的再交给响应器处理的字段就行，用以进行连续性对话。
async def _(key: str = ArgStr("a")):
    await sch.finish(key)
```

上述代码即设置了一个存续时间为30s的响应器，当30s过去后此响应器会自动销毁。

执行效果：输入`/定时结束`后，若在30s内再次发送消息，则bot会复读一次。若超过30s，由于响应器已被销毁，所以没有任何答复。

### To Be Continue <========>