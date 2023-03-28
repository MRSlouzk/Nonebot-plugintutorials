# 实例 6 每日 CP(暂弃)

难度：★★★★

作者：[MRSlouzk](https://github.com/MRSlouzk)

### 功能

每日每人可以随机抽取到另一位群友作为自己的今日 couple(对象)，并且每日有且只能抽到一个人。

~~根据群等级判断是否应该被抽，低于某个等级的人不考虑抽取~~

**因为 gocqhttps 协议的问题，此功能无法实现**

临时方案：每日零点整合群员权重，判定依据为发言时间离零点的长度，离零点越近被抽到的概率越大。

### 要点

- 使用日期以及 QQ 作为随机数种子生成随机数
- 获取群员列表
- 判断该群员今日是否抽取到了人(本地化存储，使用 JSON 文本)
- ~~群等级权重抽取算法(☆☆☆)~~(因为 gocqhttps 协议的问题，此功能无法使用)

### 实现

因为 gocq 无法正确获取群等级，所以方案作废，这里稍微说一下解决思路和我的算法实现

#### 一、加和权重法

将所有群员等级相加，形成一个有序列表，如有以下几个群员

| 名称 | 等级 |
| ---- | ---- |
| a    | 10   |
| b    | 5    |
| c    | 44   |
| d    | 54   |

那么就生成列表`[10, 15, 59, 113]`，之后使用 random 在 0~113 之间取随机数，根据随机数所处空间判定是哪个人。

这里有一个难点：如何知道我抽取到的是第几个人？比如我抽到了 100，那我怎样计算出是第几个人？方法有很多，这里大家可以自己想一想，我这里就用原始暴力的枚举法了，最后贴一下我的代码实现

```python
data_dir = "./data/couple/"
class ExtractAlgorithm(): #抽取算法
    def __init__(self, gid: int): #初始化类
        self.gid = gid
        try:
            if(os.path.exists(data_dir+str(gid)+".json")):
                with open(data_dir + str(gid) + ".json", "r") as f:
                    self.content = json.load(f)
            else:
                new_content = {}
                with open(data_dir+str(gid)+".json","w") as f:
                    json.dump(new_content, f, ensure_ascii=False, indent=4)
                self.content = new_content
        except IOError as e:
            print(e)

    def __exit(self): #保存并退出
        with open(data_dir + str(self.gid) + ".json", "w") as f:
            json.dump(self.content, f, ensure_ascii=False, indent=4)

    def test(self):
        if(self.content == {}):
            print("True")
        else:
            print("False")

    def addNewMember(self, uid: str, level: int):
        """
        添加新成员
        uid(int): 群员ID
        level: 当前群等级
        """
        if(self.content == {}):
            now_queue = 1
        else:
            max_queue = 1
            for v in self.content.values():
                if(v["queue_pos"]>=max_queue):
                    max_queue = v["queue_pos"] #取出最大数
                    now_queue = max_queue + v["level"]
        new_dict={
            uid:{
                "level": level,
                "queue_pos": now_queue, #TODO 权重排位
                "isExist": 1, #TODO 群员退群时设置为0
                "isExtracted": 0 #TODO 当日抽取过时设置为1
            }
        }
        self.content.update(new_dict)
        self.__exit()

    def memberStatus(self, uid: str, status: bool, *level: int): #成员状态修改
        """
        成员状态改动
        uid(int): 群员ID
        status(boolean): 是否在群内活跃且满足条件
        *level(Optional int): 当前群等级, 当status为True时填入有效
        """
        if(status):
            self.content[uid]["isExist"] = 1
            self.content[uid]["level"] = level
        else:
            self.content[uid]["isExist"] = 0
        self.__exit()

    def randomChoice(self, uid: str):
        if (len(self.content.values()) <= 2):
            return -1
        rnd = random.Random()
        seed = int(datetime.date.today().strftime("%y%m%d")) * int(uid)
        rnd.seed(seed)
        sum = 0
        for i in self.content.values():
            # if(i["isExist"]==0):
            #     continue
            # else:
            sum += i["level"]
        while(True):
            result = rnd.randint(1,sum)
            result_qq = None
            i = 0

            for k,v in self.content.items():
                if(v["queue_pos"]<=result):
                    result_qq = k
            if(result_qq == None):
                raise Exception("抽取时发生错误!")
            elif(result_qq == uid or self.content[result_qq]["isExist"]==0):
                i += 1
                rnd.seed(seed+i)
            else:
                return result_qq
```

#### 二、分组权重法

这个方法是一个群友提供的，感觉也挺不错，所以稍微讲讲，不过没有做代码实现。

首先读取一次群员等级，将等级根据特定区间分为 N 个组。对每个组确定一个权重，然后进行抽取，抽取出来组号后再进行组内抽取，从而达到抽取的目的。

此方法简单易用，代码实现也不算很难，可以作为一个参考

> PS：怎么这么像最近学的 FFT(快速傅里叶变换)啊草

如果读者有其他方法欢迎加群讨论或者提交 pr~
