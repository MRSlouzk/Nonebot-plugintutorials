# JSON文件的读写

*其实这应该算Python基础知识版块的内容，不过因为这个东西在Nonebot插件编写过程中也经常使用(数据库?数据库哪有Json香(笑))，而且也确实不是每个人都学过，所以特地开了篇教程**简单**<u>(详细教程请到别的地方去看!)</u>讲一讲JSON读写。*

Json是一种文本格式，常用于数据存储。基本结构为`键: 值`，键仅可以为**字符串**，而值可以是**字符串**，**列表**(使用[]包裹)，**数字**，或者另一些**键值对**，下面这段代码很好的展示了不同形式的Json结构

```json
{
    "max": 1,
    "1": {
        "user": 3237231778,
        "type": "image"
    },
    "a": [
        1,
        {
            "a": 2 //2
        },
        "b"
    ],
    "c": {
        "d": 333 //1
    }
}
```

> 所有Json文本最外层都必须有{}！

接下来我们就以上面这个Json文本为样板讲解一下Json文件的解析。

在Python当中，如果我们要解析Json文件，就必须先导入Python安装时自带的`json`库，即`import json`，之后才可以进行json操作。

------

下面的代码告诉了我们如何读入一个json文件

```python
with open("./new.json", "r", encoding="utf-8") as f:
    content = json.load(f)
```

> json.load是直接从文件读取json文本，而json.loads是读取进行了json编码的文本而非本地文件。

> 若Json文件不存在则会报错FileNotFound

上述代码即从当前目录下的"new.json"文件当中读入json数据，随后`json.load`方法会将读入的json文本**转换成Python当中的[字典](https://www.w3school.com.cn/python/python_dictionaries.asp)**存入content变量当中，所以之后讲解的内容本质其实都是Python字典的操作。

例如我们现在想要在上面的json文本当中提取最下面的"d"的值**(代码中1号位置)**，可以这么操作`val=content["c"]["d"]`，这样就可以取到`d`的值，也就是`333`了。除此之外上例我们还可以看到[]包裹的列表，如果我们想取到其中的"a"**(代码中2号位置)**的值，应该这么写：`val=content["a"][1]["a"]`，此时val的值为`2`。

------

```python
with open("./new.json", "w", encoding="utf-8") as f:
    json.dump(data, f, indent=4, ensure_ascii=False)
```

上述代码用于向json文件当中写入`data`数据，其中`dump`方法拥有很多参数，这里只讲上面代码中的三个，其他的请自行查阅。

`obj`:即上述第一个参数，用于传入需要写入Json文件当中的数据。

`fp`:即上述第二个参数，用于传入需要写入Json数据的文本指针

`indent`:传入Json文件换行缩进量，一般为2或者4。

`ensure_ascii`:是否允许Ascii码。若为`True`(默认)，则输入的中文全会转化为\uXXXX存储。

### 可供参考的一个JsonUtils案例

*(小声:代码写的可能比较烂，轻喷~)*

```python
import json, os

class JsonUtils():
    relative_url = "./data/"
    abstract_url = "file:///home/ubuntu/qqbot/data/rpg/"

    @staticmethod
    def __preBuildFile(file: str, *default: Union[str, dict]) -> bool:
        """
        判断文件及路径是否存在,若不存在则创建并生成对应文件内容

        :param file: 文件路径
        :param default: 默认文件内容
        :return:
        """
        if (not os.path.exists(file)):
            path = os.path.split(file)
            if (not os.path.exists(path[0])):
                os.mkdir(path[0])
            with open(file, "w", encoding=utf-8") as f:
                if (default):
                    arg = default[0]
                    if (isinstance(arg, str)):
                        f.write(arg)
                    elif (isinstance(arg, dict)):
                        try:
                            js = json.dumps(arg, indent=4, ensure_ascii=False)
                            f.write(js)
                        except json.JSONEncoder:
                            return False
                    else:
                        f.write(str(arg))
                else:
                    f.write("")
        return True
                      
    @classmethod
    async def read(cls, filename: str, *default) -> Tuple[dict, bool]:
        """
        读取指定json文件

        :param filename: 文件路径
        :param default: 若无此文件,创建该文件时写入的内容
        :return: [读取到的内容, 是否为新创建的文件]
        """
        file_url = cls.relative_url + filename
        if (default):
            res = JsonUtils.__preBuildFile(file_url, default[0])
        else:
            res = JsonUtils.__preBuildFile(file_url)
        with open(file_url, "r", encoding="utf-8") as f:
            content = json.load(f)
        return (content, res)

    @classmethod
    async def write(cls, filename: str, data: dict) -> None:
        """
        写入Json数据
        
        :param filename: 文件路径
        :param data: 写入的数据
        :return: 
        """
        file_url = cls.relative_url + filename
        JsonUtils.__preBuildFile(file_url)
        with open(file_url, "w", encoding="utf-8") as f:
            content = json.dumps(data, indent=4, ensure_ascii=False)
            f.write(content)
```

