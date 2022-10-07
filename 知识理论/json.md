# JSON文件的读写

*其实这应该算Python基础知识版块的内容，不过因为这个东西在Nonebot插件编写过程中也经常使用，而且不是每个人都学过，所以特地开了篇教程**简单**<u>(详细教程请到别的地方去看!)</u>讲一讲JSON读写。*

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

下面我们就以上面这个Json文本为样板讲解一下Json文件的解析。

在Python当中，如果我们要解析Json文件，就必须先导入Python安装时自带的`json`库，即`import json`，之后才可以进行json操作。

------

下面的代码告诉了我们如何读入一个json文件

```python
with open("./new.json", "r", encoding="utf-8") as f:
    content = json.load(f)
```

> json.load是直接从文件读取json文本，而json.loads是读取进行了json编码的文本而非本地文件。

上述代码即从当前目录下的"new.json"文件当中读入json数据，随后`json.load`方法会将读入的json文本**转换成Python当中的[字典](https://www.w3school.com.cn/python/python_dictionaries.asp)**存入content变量当中，所以之后讲解的内容本质其实都是Python字典的读写操作。

例如我们现在想要在上面的json文本当中提取最下面的"d"的值**(代码中1号位置)**，可以这么操作`val=content["c"]["d"]`，这样就可以取到`d`的值，也就是`333`了。除此之外上例我们还可以看到[]包裹的列表，如果我们想取到其中的"a"**(代码中2号位置)**的值，应该这么写：`val=content["a"][1]["a"]`，此时val的值为`2`。



### To Be Continue(剩下部分正在制作中)