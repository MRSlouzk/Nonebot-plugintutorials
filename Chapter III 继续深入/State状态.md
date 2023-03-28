# T_State 的使用

#### 前言

在有`got`以及`receive`等装饰器或者多个处理函数的事件响应器当中，有时候会需要另外一个处理函数当中的变量，这时候该如何在同一个响应器的多个处理函数之间传递参数呢？

这时候就需要使用 Nonebot 当中自带的**用于存储事件处理状态**的类`T_State`了，下面我们会详细讲解这个参数的使用

> 当然，要实现参数的上下文传递还有很多别的方法，不过这里只是为了讲解 state 的一个用途所举的例子

> 重要：Nonebot-2.0.0-rc1 对 State 进行了一些[更改](https://github.com/nonebot/nonebot2/pull/1160)，旧版的 State 不再适用

### 一个案例

```python
from nonebot import on_command
from nonebot.params import CommandArg, ArgStr
from nonebot.adapters.onebot.v11 import Message
import random

a = on_command("猜数字")
@a.handle()
async def f1(args: Message = CommandArg()):
    arg = args.extract_plain_text().split()
    rnd = random.randint(0, int(arg[0]))

@a.got("num", prompt=f"请输入猜的数字")
async def f2(arg: str = ArgStr("num")):
    if(arg==...): #该如何得知上面一个处理函数(f1)当中的rnd值呢?
        await a.finish("恭喜你猜对了")
    await a.finish("很遗憾猜错了~")
```

上述代码实现了一个非常简单的猜数字(_不具备参数效验等功能_)，`f1`当中生成了一个随机数`rnd`，之后接收用户回答并判断是否等于生成的`rnd`值，若相等就猜对了。

问题就来了：**如何将`f1`的 rnd 参数传递给`f2`**？

### 问题解决

首先想到的是能不能直接传参给 f2，然后判断是否相同，如下面这段代码

```python
a = on_command("猜数字")
@a.handle()
async def f1(args: Message = CommandArg()):
    arg = args.extract_plain_text().split()
    rnd = random.randint(0, int(arg[0]))
    print(rnd)
    await f2(rnd)

@a.got("num")
async def f2(rnd: int, arg: str = ArgStr()):
    print(rnd, arg)
    if(arg==rnd):
        await a.finish("恭喜你猜对了")
```

但是我们发现启动这段代码后会报错：

```shell
ValueError: Unknown parameter rnd for function <function f2 at 0x7f63ba10bf40> with type <class 'int'>
```

看来是不行的，程序无法解析`f2`的 rnd 参数

那我们需要怎么做？有没有办法**将这个变量存储在响应器(类)内部，并且可以被该响应器的处理函数随时调用或者更改？**答案是有的，这也是我们接下来要讲的`T_State`。

### T_State

由[官方文档](https://v2.nonebot.dev/docs/api/typing#T_State)的说明，`T_State`是一个字典类型的数据，描述是**<u>事件处理状态 State 类型</u>**。这个参数会伴随着一个响应器的诞生而诞生，消亡而消亡，并且每个响应器都有自己独立的 State 参数，那这个参数里面到底有些什么内容？我们可以通过下面这段代码输出查看

```python
from nonebot import on_command
from nonebot.typing import T_State

a = on_command("state测试")
@a.handle()
async def f1(state: T_State):
    print(state)
```

我的输出：

```python
{
    '_prefix': {
        'command': (
        'state测试',
        ),
        'raw_command': '#state测试',
        'command_arg': [],
        'command_start': '#'
    }
} #重新格式化过了
```

大家可以很明显的看到 State 当中用字典形式存储了响应器的一些基本信息，如触发指令，指令前缀，传入参数等等。

那我们有没有办法向这个字典中添加一些我们自己人为定义的东西？答案是**可以的**，接下来我们来看如何加入以及使用。

```python
from nonebot import on_command
from nonebot.typing import T_State

a = on_command("state测试")
@a.handle()
async def f1(state: T_State):
    print(state)
    state.update({"arg": 114514})
    print(state)
    print(state.get("arg"))
```

执行后我们发现输出为

```shell
{'_prefix': {'command': ('state测试',), 'raw_command': '#state测试', 'command_arg': [], 'command_start': '#'}}
{'_prefix': {'command': ('state测试',), 'raw_command': '#state测试', 'command_arg': [], 'command_start': '#'}, 'arg': 114514}
114514
```

对，就是这么简单，我们完全可以把传入的`state`变量当成一个字典来使用，开头的那段代码我们也可以重写一下了

```python
a = on_command("猜数字")
@a.handle()
async def f1(state: T_State, args: Message = CommandArg()):
    arg = args.extract_plain_text().split()
    rnd = random.randint(0, int(arg[0]))
    state.update({"num": rnd})

@a.got("num")
async def f2(state: T_State, arg: str = ArgStr()):
    num = state["num"]
    if(arg==num):
        await a.finish("恭喜你猜对了")
```

当然，State 的应用领域不止是这里，剩下的就留给大家自己去探索了。
