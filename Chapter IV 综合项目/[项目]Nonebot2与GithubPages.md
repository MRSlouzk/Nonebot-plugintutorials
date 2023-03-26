# 教程:群Meme图网站的搭建与使用
## 前言
灵感是看到[NoneMeme](https://github.com/NoneMeme/NoneMeme)仓库, 这是个收集Nonebot官群当中梗图的网站, 于是就想能不能利用这个模板给所有群聊使用以及对接Nonebot, 实现在QQ里随时上传梗图, 于是就诞生了这个视频

## 一、Github Page搭建
### 1.要点
- fork MemeBox仓库
- 修改参数
- 搭建Github Pages  
### 2.注意事项
- 除了Github Page外, Gitee Page与Coding都可以部署网页, 如果有自己的服务器也可以扔在服务器上(需要备案)。
- Github Page有一定限制, 如只能部署静态网页, 一段时间内的部署次数限制以及仓库大小限制
- 如果不想买域名可以用github.io解析, 具体方法见"步骤"
- 因为网页是托管到了Github的服务器, 所以国内有些区域访问速度会很慢(甚至直接TIMED_OUT), 解决方法见"步骤"
### 3.步骤

> 在正式开始之前, 我们先简单了解一下Github(Git)的一些操作, 以便后续讲解和网站维护  
Github是一个使用Git的云端代码托管仓库, 现在我们把它看成一个人人都可以分享自己内容的网盘。`Commit`(提交)操作相当于给你上传文件的这件事的内容(在网盘上传了哪些文件,删了哪些文件,改了哪些文件,时间)记在一个表格当中, `Push`(推送)相当于上传文件到网盘, `Pull`(拉取)是从网盘中下载文件。  

首先打开[MemeBox](https://github.com/NoneMeme/memebox)仓库, 点击右上角的`Use this template`(使用此模板), 选择`Create a new repository`(创建新仓库), 接下来会需要填写创建仓库的一些信息。  
`Repository name`即你的仓库名称, 这个可以随便取, 下面的`Description`是仓库的简介, 用于对你的仓库做一些大致的介绍。`Public`和`Private`分别是公共和私有仓库, 决定你的仓库能不能被别人看到, 一般选择Public即可, 其它选项则不用管。  

> 注意: 如果你不想购买域名, 请将仓库名称(Repository name)改为`你的github用户名.github.io`, 如我的用户名为MRSlouzk, 就改成`mrslouzk.github.io`。这是github官方提供的一个免费域名, 一个账号只有一个。  

填写完信息后点击最下方绿色的按钮即可创建你的仓库。  
> 创建完成后要开始编辑仓库的内容, 这里你有两种选择: 1.Clone仓库到本地进行文件修改, 之后进行`Commit`与`Push` 2.直接在Github页面进行修改, 具体操作是点击某个文件, 然后找到并点击文字框右上角一个铅笔形状的按钮即可。两种方法都可以

接下来开始填写网站的一些基本信息。打开仓库根目录(后文均用`./`表示)当中的`Makefile`文件, 可以看到有很多的参数。首先我们找到`PAGELANG`, 将后面的en改成zh, 这步是将英文界面模式换成中文, 之后的`TITLE`即网页标题(显示在标签栏的文字), `DESC`即网页的概述, `FOOTER`是网页的脚注, 剩下的参数则可以不用管, 修改完成后点击绿色按钮的`Commit`即可。  

下一步是部署Github Actions来自动`make`和`deploy`网页, 其中`deploy`已经有了, 我们需要对网站进行makr操作才能正常使用  
首先打开仓库的"Settings->Actions->General", 找到"Workflow permissions", 选择`Read and write permissions`, 之后点击下面的"Save"。
> 注:下面的步骤适用于不方便将代码`clone`到本地并安装make环境的人, 有能力的完全可以克隆下来`make`后再提交上去(Memebox仓库也是这么写的)  

打开你的Profile页面, 选择"Developer Settings", 点击选取"Personal access tokens (classic) -> fine-grained personal access token", 填入对应token名, 过期日期。"Repository access"中选取"Only select repositories", 然后选择对应仓库。点开"Repository permissions", 将`Actions`,`Contents`,`Deployments`,`Pages`,`Pull requests`,`Webhooks 
`,`Workflows`, 权限改为"Read and write", `Metadata`与`Secrets`为"Read-only", 然后"Generate token", 复制"github_pat_..."。再打开仓库的"Settings->Secrets->Actions", 点右上角新建一个secrets, 将上面的token复制进去保存即可, 命名为"TOKEN"。
<!-- 然后打开仓库的"Settings->Pages"页面, 将"Build and deployment"标签下面的"Source"改为`Github Actions`, 之后选择`Static HTML`并点击下面的`Configure`按钮, 之后会跳转到修改Github Actions文件的页面, 请将内容改成下面的代码 -->
之后跳转到"Actions"界面, 点击`New workflow` -> `set up a workflow yourself`, 之后把下面这段代码复制进去并提交即可。  
```yaml
name: Make project

on:
  workflow_dispatch:

permissions:
  contents: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout
      uses: actions/checkout@v3
      with:
        token: ${{ secrets.TOKEN }}
    - name: Install dependencies
      run: make
    - name: Commit
      uses: stefanzweifel/git-auto-commit-action@v4
      with:
        commit_message: 'Update File'
      env:
        GITHUB_TOKEN: ${{ secrets.TOKEN }}
```  
提交后跳转到"Actions"界面, 点击刚刚创建的workflow, 会出现一行"This workflow has a workflow_dispatch event trigger.", 点击右侧的`Run workflow`, 等待运行完成即可  
> 到此网站的 make工作就结束了, 接下来就是上传图片与部署了。

打开"meme"文件夹, 点击右上角Upload files, 上传图片并进行提交, 即可成功将图片加入网站当中(WorkFlow会自动生成配置文件并提交代码)。"text"文件夹则可以上传Markdown文件来发布文章。  
如果你使用的是github.io域名, workflow运行结束后你就可以看到有图的网站了~(PS:可能要等几分钟才能生效,  如果没生效请尝试清除浏览器缓存后重试)。如果不是github.io请参见"域名的购买与解析"部分  

> 如何解决Github访问慢的问题？  
使用者方面:  
1.科学上网  
2.使用[fast-Github](https://github.com/dotnetcore/FastGithub)进行加速  
网站方面:  
1.挂载CDN进行加速, 但是成本很昂贵  
2.反向代理  
3.压缩图片体积, 尽量都使用webp格式(Nonebot2对接的插件部分已添加格式转换)  
4.在Coding或者Gitee挂载镜像网站
## 二、域名购买与解析(可选)
### 1.要点
- 域名的购买  
- DNS解析  
### 2.注意事项
- 国内购买域名需要备案才能使用
- Cloudflare加速服务虽然免费, 但它的服务器在国外, 对于国内的访问速度没有明显提升, 仅作为防火墙使用, 加不加无所谓
### 3.步骤
域名的购买有多个渠道, 可以买国内的也可以国外的, 在这里就不推荐了, 大家可以自己去找。  
购买到域名后, 进入解析页面, 记录类型选择`A`, 主机记录根据自己需求选择, 线路类型为默认, TTL随意。记录值设置为`185.199.108.153`或者`185.199.109.153`, 然后保存, 等待解析生效即可。  
之后进入仓库, 依次点击"Settings->Pages", 找到"Custom domain", 将域名粘贴进去保存, 在`./`生成一个CNAME文件后即可。

## 三、对接Nonebot2插件
### 1.要点
- Github Rest API
- Nonebot实现  
### 2.注意事项
- 请勿向他人泄露你的Github Token(如公共仓库/论坛/截图等当中, 发送代码片段时请注意打上马赛克)
- 第三方Github库均是Github官方提供的, 讲解时不使用主要是可以稍微弄清一下里面的原理, 如想使用可以自行阅读文档并修改代码
- Nonebot的安装与使用这里省略, 具体的过程请在B站搜"Well404"看他的视频
### 3.步骤
Github提供了两种API进行各种操作, 一种是`Rest API`, 还有一个`GraphQL API`, 这里我们选用第一个来进行交互。为了方便理解, 下面的源代码当中直接使用`requests`向接口`PUT`参数来实现操作, 当然其实有很多现成的第三方库都封装了这个API, 具体有哪些可通过[Github Docs](https://docs.github.com/en/rest/overview/libraries)查询, 使用API进行`Commit`的具体操作也可以看[Github Docs](https://docs.github.com/en/rest/commits/commits)了解。  
下面是现成的代码, 请放置在`src/plugins`的一个文件夹当中运行
```python
# __init__.py
from nonebot.adapters.onebot.v11 import GroupMessageEvent, Message
from nonebot.params import ArgStr, CommandArg, T_State
from nonebot import on_command

import re
import requests as rq

from .github_opera import GithubOperation

def user_check(event: GroupMessageEvent):
    return event.user_id == 3237231778

add_pic = on_command("上传meme图", rule=user_check)
@add_pic.handle()
async def _(state: T_State, arg: Message = CommandArg()):
    args = arg.extract_plain_text().split()
    if(len(args) == 0):
        await add_pic.finish("请输入图片名字")
    elif(len(args) != 1):
        await add_pic.finish("图片有且仅能有一个名字")
    else:
        state["pic_name"] = args[0]

@add_pic.got("ans", prompt="下一个请发送你想添加的图片, 输入文字为取消发送")
async def _(state: T_State, event: GroupMessageEvent, arg: str = ArgStr("ans")):
    name = state["pic_name"]

    pattern = re.compile(r"CQ:image(.+?)url=(?P<url>.*?)]")
    match = re.search(pattern, arg)
    if(match is None):
        await add_pic.finish("已取消上传")
    img_url = match.group("url")
    try:
        resp = rq.get(url=img_url, timeout=10)
        img_cot = resp.content
    except Exception:
        await add_pic.finish("无法从tx服务器下载图片, 请重试")

    with open(f"temp.png", "wb") as f:
        f.write(img_cot)
    await add_pic.send(f"读取图片成功, 正在转换格式以及推送至仓库")

    import os, platform
    from nonebot.log import logger
    out = os.system(f"ffmpeg -i temp.png temp.webp")
    logger.info(out)
    img_cot = open(f"temp.webp", "rb").read()
    if(platform.system() == "Windows"):
        os.system("rm temp.webp")
    else:
        os.system("rm -rf temp.webp")

    g = GithubOperation()
    res = await g.pic_commit(name, "Auto Commit", img_cot, ".webp")
    await add_pic.finish(res)
```
下面这个文件请与上面放在一个文件夹当中
```python
# github_opera.py
import base64 as b64
import json

import requests as rq

class GithubOperation(object):
    token: str = ""
    # 请在这里填入你的Github Token

    def __init__(self):
        pass

    async def pic_commit(self, pic_name: str, commit_msg: str, content: bytes, pic_type: str = ".png"):
        data = {
            "message": f"{commit_msg}",
            "content": b64.b64encode(content).decode()
        }
        headers = {
            "Accept": "application/vnd.github+json",
            "Authorization": f"token {self.token}"
        }
        url = f"https://api.github.com/repos/你的github名字/你的仓库名字/contents/meme/{pic_name}{pic_type}"
        # 请将这里的中文部分根据自己的参数修改为对应值
        data = json.dumps(data)
        try:
            with rq.session() as s:
                res = s.put(url=url, headers=headers, data=data, timeout=10)
                if(res.status_code == 201):
                    return "上传成功!内容将在1分钟后生效~"
                else:
                    return "上传失败, 请检查网络或者源代码是否出错"
        except rq.exceptions.Timeout:
            return "上传失败, 连接超时"
        except Exception as e:
            return str(e)
```
- 运行此代码前请先学习Nonebot2的使用方式
- 上述`GithubOperation`中的token可以填入之前生成的(token必须要有respo的全部权限)
- 确保电脑环境变量当中添加了ffmpeg, 测试方法: 在命令行中输入`ffmpeg`没有提示找不到命令即可, ffmpeg的下载与添加环境变量的方法请百度, 不同系统均有差异。
- 运行此代码的python环境请先安装: `nb_cli`,`requests`,`nonebot-plugin-onebot`

### 插件使用说明
输入`/上传meme图 ${图片名}`即可触发, 之后发送图片即可录入网站(发送文字即取消录入)