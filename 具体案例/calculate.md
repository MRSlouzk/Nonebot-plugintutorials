# 实例4 计算器

难度：★★★

作者：[XieXiLin](https://github.com/XieXiLin3)

### 功能

发送`/加法 数值 数值`来进行求和计算，减法同理。

### 要点

- 参数的分离
- 子函数

### 实现

##### 参数的分离

```python
from nonebot.adapters.onebot.v11.event import GroupMessageEvent
from nonebot.adapters.onebot.v11.message import Message
from nonebot.params import CommandArg
from nonebot import on_command

add=on_command("add",priority=99,block=True)

@add.handle()
async def add_handle(event: GroupMessageEvent, args: Message = CommandArg()):
    arg = args.extract_plain_text().split()
    pass
```

在此处，我们用 [`on_command`](https://v2.nonebot.dev/docs/api/plugin/on#on_command) 检测命令，然后进行解析。其中指令参数使用 `Message = CommandArg()` 进行解析，最后使用 `args.extract_plain_text().split( )` 将参数转换为字符串后再进行分割 (其中分隔符是 ` `) ，成为我们最后拿的指令参数。

例如，我们发送的指令是 `/add 1 2` ，则进行 `args.extract_plain_text()` 后变成 `1 2` ，最后经过 `split()` 后就变成 `["1", "2"]` 。

> 为了简便，可以直接在函数定义里直接转换，如
> ```python
> async def add_handle(event: GroupMessageEvent, args: Message = CommandArg(), arg = args.extract_plain_text().split()):
>     pass
> ```

##### 子函数

```python
def cal_1(arg):
    count=len(arg)
    sum=0
    for num in range(0,count):
        sum+=float(arg[num])
    return sum
```

在此处，我们新定义了一个子函数进行计算处理。首先客户端使用 `cal_1(arg)` 进行调用，并在 `arg` 处传入参数；子函数开始处理。

首先通过 `len()` 进行取参数数量，然后定义 `sum=0` 储存加法结果；进入 `for` 循环，使 `num` 的值从 `0` 一直到 `count - 1` 为止； `for` 循环中， `sum` 的值为原本的值加上 `num` 对应变量经过浮点转换后的值；最后通过 `return sum` 返回最终结果。

> 关于 `for` 循环的知识具体参见[菜鸟教程🔗](https://www.runoob.com/python/python-for-loop.html)

> 关于更高级的运算，需要导入 `math` 库进行 (`import math`) 。

#### 完整代码

```python
from audioop import mul
from math import factorial
from nonebot.adapters.onebot.v11 import GroupMessageEvent
from nonebot.params import CommandArg
from nonebot.adapters.onebot.v11.message import Message,MessageSegment
from nonebot import on_command
import math

add=on_command("加法",priority=99,block=True)
minus=on_command("减法",priority=99,block=True)
multi=on_command("乘法",priority=99,block=True)
fact=on_command("阶乘",priority=99,block=True)
equ_1=on_command("二元一次",priority=99,block=True)
ten_to_bin=on_command("十转二",priority=99,block=True)
bin_to_ten=on_command("二转十",priority=99,block=True)

char_list=["1","2","3","4","5","6","7","8","9","0"]

def can_cal(args):
    for num in range(0,len(args)):
        try:
            if(float(args[num])<-100000 or float(args[num])>100000):
                return False
        except ValueError:
            return False
    return True

def cal_1(arg):
    count=len(arg)
    sum=0
    for num in range(0,count):
        sum+=float(arg[num])
    return sum

def cal_2(arg):
    return float(arg[0])-float(arg[1])

def cal_3(arg):
    count=len(arg)
    sum=1
    for num in range(0,count):
        try:
            sum*=float(arg[num])
        except OverflowError:
            return "溢出了!!!"
    return sum

def cal_4(arg):
    if int(arg[0])>=100:
        return "太大了!!!"
    try:
        result=factorial(int(arg[0]))
        return result
    except OverflowError:
        return "溢出了!!!"
    except ValueError:
        return "必须要是整数!!!"

def cal_5(arg):
    a = float(arg[0])
    b = float(arg[1])
    c = float(arg[2])

    if(a == 0 and b == 0 ):
        return "无解"
    elif(a == 0 and b != 0):
        x = float(-(c / b))
        return "此方程有一个实根"+str(x)
    elif(b * b - 4 * a * c == 0):
        x1 = float(-(b / (2 * a)))
        x2 = float(-(b / (2 * a)))
        return "有两个相等的实根:"+str(x1)
    elif(b * b - 4 * a * c > 0):
        x1 = float(-(b / (2 * a))) + (math.sqrt(math.pow(b, 2) - 4 * a * c)) / 2 * a
        x2 = float(-(b / (2 * a))) + (math.sqrt(math.pow(b, 2) - 4 * a * c)) / 2 * a
        return str.format("有两个不等实根{0:.2f}和{1:.2f}",x1 , x2)
    elif(b * b - 4 * a * c < 0):
        a = float(-(b / (2 * a)))
        b = (math.sqrt(math.pow(b, 2) - 4 * a * c)) / 2 * a
        return str.format("有两个共轭{0}+{1}i和{0}-{1}i", a, b)

@add.handle()
async def _(event:GroupMessageEvent, args: Message = CommandArg()):
    arg=args.extract_plain_text().split()
    if len(arg)==0:
        await add.finish("没有数字也不会算啊...")
    elif len(arg)==1:
        await add.finish("只有一个数字...")
    elif (len(arg)>=2) and (len(arg)<=10):
        if can_cal(arg):
            await add.finish(f"算出来了!答案是:{cal_1(arg)}")
        else:
            await add.finish("算不过来了!")
    else:
        await add.finish("算不过来了!")

@minus.handle()
async def _(event:GroupMessageEvent, args: Message = CommandArg()):
    arg=args.extract_plain_text().split()
    if len(arg)==0:
        await add.finish("没有数字也不会算啊...")
    elif len(arg)==1:
        await add.finish("只有一个数字...")
    elif (len(arg)==2):
        if can_cal(arg):
            await add.finish(f"算出来了!答案是:{cal_2(arg)}")
        else:
            await add.finish("算不过来了!")
    else:
        await add.finish("只会减两个数字!")

@multi.handle()
async def _(event:GroupMessageEvent, args: Message = CommandArg()):
    arg=args.extract_plain_text().split()
    if len(arg)==0:
        await add.finish("没有数字也不会算啊...")
    elif len(arg)==1:
        await add.finish("只有一个数字...")
    elif (len(arg)>=2) and (len(arg)<=8):
        if can_cal(arg):
            await add.finish(f"算出来了!答案是:{cal_3(arg)}")
        else:
            await add.finish("算不过来了!")
    else:
        await add.finish("算不过来了!")

@fact.handle()
async def _(event:GroupMessageEvent, args: Message = CommandArg()):
    arg=args.extract_plain_text().split()
    if len(arg)==0:
        await add.finish("没有数字也不会算啊...")
    elif len(arg)==1:
        if can_cal(arg):
            await add.finish(f"算出来了!答案是:{cal_4(arg)}")
        else:
            await add.finish("算不过来了!")
    else:
        await add.finish("算不过来了!")

@equ_1.handle()
async def _(event:GroupMessageEvent, args: Message = CommandArg()):
    arg=args.extract_plain_text().split()
    if len(arg)<=2:
        await add.finish("数字不够也不会算啊...")
    elif len(arg)==3:
        if can_cal(arg):
            await add.finish(f"算出来了!答案是:{cal_5(arg)}")
        else:
            await add.finish("算不了!")
    else:
        await add.finish("算不了!")

@ten_to_bin.handle()
async def _(event: GroupMessageEvent, args: Message = CommandArg()):
    arg = args.extract_plain_text().split()
    if(len(arg)==0):
        await ten_to_bin.finish(Message("没参数算不了啊!"))
    elif(len(arg)==1):
        if(arg[0].isascii()):
            num=int(arg[0])
            if(num>0 and num <=10000):
                binary_list = []
                binary = num % 2
                divide = num
                while True:
                    binary_list.append(str(binary))  # 需要将里面的数字转成字符串，用于join方法解析
                    divide = int(divide / 2)
                    binary = divide % 2
                    if binary == 0 and divide == 0:
                        break
                arr = list(reversed(binary_list))  # 使用 list(reversed(array)) 对列表进行反向排序
                str_1 = ''.join(arr)  # 使用 .join将数组直接转成字符串
                await ten_to_bin.finish(Message(f"算出来了!答案是{str_1}!"))
            await ten_to_bin.finish(Message("数字太大了或者太小了!"))
        await ten_to_bin.finish(Message("不会算这个!"))
    await ten_to_bin.finish(Message("参数不对哦~"))

@bin_to_ten.handle()
async def _(event: GroupMessageEvent, args: Message = CommandArg()):
    arg = args.extract_plain_text().split()
    if (len(arg) == 0):
        await ten_to_bin.finish(Message("没参数算不了啊!"))
    elif (len(arg) == 1):
        if (arg[0].isascii()):
            num=arg[0]
            try:
                result=int(str(num),2)
                await ten_to_bin.finish(Message(f"算出来了!答案是{result}!"))
            except ValueError:
                await ten_to_bin.finish(Message("输入参数格式不对!"))
        await ten_to_bin.finish(Message("不会算这个!"))
    await ten_to_bin.finish(Message("参数不对哦~"))
```