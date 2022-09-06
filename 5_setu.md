# 实例3 来点涩涩

难度：★★★★

作者：ubby

### 功能

发送`/随机涩图`后bot会从涩图api拿取一张图并且发到群里，但是每个人都有一定的CD时间。

本教程使用的色图API为`lolicon`。

### 要点

- 获取图片(爬虫)
- CD时间设定

### 实现

#### 获取图片部分

使用requests库获取请求后的数据，并用json库解析出返回的json中的链接

> **注意：学习此部分需要一定的爬虫知识基础！**

```python
async def get_url():
    url = "https://api.lolicon.app/setu/v2"
    content = requests.post(url).text
    return content
```
上述代码用于向API发送`post`请求，并得到相应的JSON数据，此处使用到了`requests`库(不过开发中的话个人建议是使用`httpx`库，因为可以异步执行)。具体使用规范请参见[随机色图 (lolicon.app)](https://api.lolicon.app/#/setu)。

上述API网站返回的JSON数据如下

```json
{
  "error": "",
  "data": [
    {
      "pid": 98978803,
      "p": 0,
      "uid": 3465319,
      "title": "アリス",
      "author": "じょーん",
      "r18": false,
      "width": 1581,
      "height": 2236,
      "tags": [
        "ブルーアーカイブ",
        "碧蓝档案",
        "天童アリス",
        "Alice Tendou",
        "マイクロビキニ",
        "极小比基尼",
        "ロリ",
        "萝莉",
        "お尻",
        "臀部",
        "ちっぱい",
        "贫乳"
      ],
      "ext": "png",
      "uploadDate": 1654940982000,
      "urls": {
        "original": "https://i.pixiv.re/img-original/img/2022/06/11/18/49/42/98978803_p0.png"
      }
    }
  ]
}
```

json数据在python中的处理过程是利用json库把`json`文本转成字典，然后对其进行字典操作。上述响应数据中有很多字段，大家可以选择性的去提取，作为图片发送时的附带消息，在这里只讲如何提取到图片链接。

> 顺便提一下图片消息(MessageSegment)怎么和普通消息(Message)怎么合并在一个消息中发送：有两个方法。一是使用send_forward_msg方法发送合并消息。二是直接使用"+"拼接两段消息，任何消息段、消息都是可以直接使用"+"进行拼接，如MessageSegment.at(id)+MessageSegment.image(file=)+Message("图片来了!")就是拼接消息，**消息段可以看为消息类型的子类**。

可以看到图片链接位于"data"->**列表第一位**->"urls"->"original"当中，由此我们可以知道提取方式为：\["data"]\[0]\["urls"]["original"]，下面的代码是具体实现

```python
img_url = content["data"][0]["urls"]["original"]
```

获取到图片连接后就可以开始下载了，但是此处有两种方案：

> 1.下载图片到本地后缓存，再把本地缓存的图片发送到QQ当中。优点是降低因为网络环境问题导致图片无法发送的概率(可以请求多次发送)，还可以对图片进行一些处理。缺点是比较耗时间。
>
> 2.直接用`image`方法中的在线下载。优点是速度相对来说快一点，缺点是在网络短暂异常的情况下会出错，且不能进行处理。

第二种方法相信大家看过前面的教程后都应该知道是怎么做的了，在此就不赘述。

第一种方法仍使用request库当中的方法来接收图片数据，代码如下

```python
response = requests.get(img_url)
with open("tmp/img.jpg", "wb") as f:
    f.write(req.content)
```

上述代码将图片下载到`bot.py`目录下的`tmp/img.jpg`文件当中。

##### *压缩图片(选学)*

有时候下载的图片会比较大，这也会导致发送图片时速度过慢，这时候就有需要对图片进行压缩。

```python
import glob
import PIL

DIR = r"你自己的绝对路径！"

class Compress_Picture(object):
    def __init__(self):
        # 图片格式，可以换成.bpm等
            self.file = '.jpg'

    # 图片压缩批处理
    async def compressImage(self):
        for filename in glob.glob('%s%s%s' % (DIR, '*', self.file)):
            # 打开原图片压缩
            sImg = Image.open(filename)
            w, h = sImg.size
            #变成1280*1080的图片
            dImg = sImg.resize((1280, 1080), Image.ANTIALIAS)  # 设置压缩尺寸和选项，注意尺寸要用括号
            # 如果不存在目的目录则创建一个
            comdic = "%scompress/" % DIR
            if not os.path.exists(comdic):
                os.makedirs(comdic)

            # 压缩图片路径名称
            f1 = filename.split('/')
            f1 = f1[-1].split('\\')
            f2 = f1[-1].split('.')
            f2 = '%s%s1%s' % (comdic, f2[0], self.file)
            dImg.save(f2)  # save这个函数后面可以加压缩编码选项JPEG之类的
            
```

#### CD时间判定

CD时间的计算有很多的方案，在这里我选择的是通过json文件存储上一次请求成功的时间，然后被请求时将当前时间减去存储的时间，即可获取到已经经过的时间，进而实现CD功能。

```python
import datetime

data_dir = "./data/"
def write(): #写入CD时间
    with open(data_dir + "cd.json", "w", encoding="utf-8") as f:
        CD = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        dict = {
            "CD": str(CD)
        }
        json.dump(dict, f, indent=4)

with open(data_dir + "cd.json", "r", encoding="utf-8") as f:
    content = json.load(f)
if (not content):
    write()
else:
    CD = content["CD"]
    old_CD = datetime.datetime.strptime(CD, "%Y-%m-%d %H:%M:%S")
    sec = (datetime.datetime.now() - old_CD).seconds #剩余CD时间计算
    if (sec >= 5): #CD时长
        write()
        #......
    else:
        #......
        pass
```

图片的发送使用的是`image`方法中的`file`参数，值为一个绝对路径，即你下载到本地的图片的路径，具体写法参看[onebot-11文档](https://github.com/botuniverse/onebot-11/blob/master/message/segment.md#图片)。

这里就不直接放成品代码了，大家可以自己思考一下然后写出来。
