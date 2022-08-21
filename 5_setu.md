# 实例3 来点涩涩

难度：★★★★

### 功能

发送`/随机涩图`后bot会从涩图api拿取一张图并且发到群里，但是每个人都有一定的CD时间。

### 要点

- 获取图片(爬虫)
- CD时间设定
- 使用黑名单

### 实现

- 获取图片部分，使用requests库和json库获取请求后的数据，并用json库获取返回的json中的链接

```async def save_img():
    href = "https://api.lolicon.app/setu/v2"
    req1 = requests.post(href)
    str_json = req1.text
    get_json = json.loads(str_json)
    return get_json
```
- 压缩图片部分，使用PIL库和glob库

```DIR = r"你自己的绝对路径！"

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
- 使用datetime库进行CD的判断
- 这里使用nonebot的on_command
- 用re获取这个命令的QQ用户ID

```CD = {}

setu = on_command("随机涩图", priority=1)


@setu.handle()
async def out(bot: Bot, event: Event):
    re_compi = re.compile(r"user_id=.*")

    id = str(re.findall(re_compi,str(event.get_user_id))[0]).split(",")[0].split("=")[1]



    if id in CD.keys():
        if datetime.datetime.now() >= CD[id][1]:
            get_json = await save_img()
            img_href = get_json["data"][0]["urls"]["original"]
            req = requests.get(img_href)
            with open("tmp/img.jpg", "wb") as f:
                f.write(req.content)
            yasuo = Compress_Picture()
            await yasuo.compressImage()
            await setu.finish(
                MessageSegment.at(id) + MessageSegment.text("你的涩图来咯，接下来会有30秒CD哦，爱你哟~") + MessageSegment.image(r"file:///自己的绝对路径"))
        else:
            await setu.finish(MessageSegment.at(id) + MessageSegment.text("还在冷却哦，稍等，爱你哟~"))
    else:
        CD[id] = [datetime.datetime.now() , datetime.datetime.now() + datetime.timedelta(seconds=30)]
        get_json = await save_img()
        img_href = get_json["data"][0]["urls"]["original"]
        req = requests.get(img_href)
        with open("tmp/img.jpg", "wb") as f:
            f.write(req.content)
        yasuo = Compress_Picture()
        await yasuo.compressImage()
        await setu.finish(MessageSegment.at(id)+MessageSegment.text("你的涩图来咯，接下来会有30秒CD哦，爱你哟~")+MessageSegment.image(r"file:///自己的绝对路径"))
```
