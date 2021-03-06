# NoneBot2插件从零开始的教程  

### 学习要求  

- 熟练使用Python。内容包括但不仅限于安装，环境配置，代码编写，调试等。本教程**不会详细的去讲解这些Python基础的语法点**，所以想在这里边学Nonebot边学Python的可以放弃了。想学Python基础的请自行去看课程，B站一搜一大堆。
- 熟悉常用编辑器。写Python的话一般主流编辑器都是[VScode](https://code.visualstudio.com/)或者[Pycharm](https://www.jetbrains.com/pycharm/download/#section=windows)，Nonebot插件项目的话一般都用的VScode(当然本项目为了方便管理用了Pycharm)。
- 熟悉Nonebot2的部署与搭建。注意！注意！注意！**<u>本教程不是Nonebot2基础教程，而是插件编写教程</u>**，所以不要询问关于Bot的部署方面的问题，请自行解决。这里也推荐一个视频教程，很适合入门：[【零基础搭建QQ机器人】开源、免费、纯新手向的nonebot2.0.0beta版讲解](https://www.bilibili.com/video/BV1aZ4y1f7e2)，还有配套的GitHub仓库：[使用NoneBot2搭建QQ机器人](https://github.com/Well2333/NoneBot2_NoobGuide)。
- 遇到问题请先自己解决。NoneBot2的官方文档等其实讲的都挺齐全了，虽然说对纯新手有点不友好。文档共有两个：[Nonebot2官方文档](https://v2.nonebot.dev/docs/tutorial/create-project)，和本教程所用的适配器[OneBot V11官方文档](https://github.com/botuniverse/onebot-11)。如果实在搞不懂再来问，当然问问题的时候不要求精通《提问的艺术》，但也请遵循标准提问规范，不要让人血压升高。
- 因为我其实Nonebot2到现在才玩了3个月左右，有些地方肯定讲的不是非常好，如果有讲的不对或者不好的地方欢迎指出或者补充，如果想一起交流技术的话那就更好了，~~看看到时候能不能建个群白嫖大佬们的技术（笑）~~。
- 最重要的一点：不要做伸手党，不要看到个厉害的插件就说能不能把源码发给我用？如果当事人不在意还好，严重了的话叫剽窃他人劳动成果。请尽量自己独立思考来解决问题！

### 本教程配置

- Nonebot框架版本：2.0.0-Beta2

- Go-cqhttp协议版本：1.0.0-rc1

- OneBot版本：V11 2.0.0

- Pycharm版本： 2022.1.3 Community Edition

- Python版本：3.10(Windows端＆Ubuntu端)

- Ubuntu版本：20.04 LTS

  ------

  #### 一些注意事项

  bot.py文件代码**(不用完全照抄的)**

```python
import nonebot
from nonebot.adapters.onebot.v11 import Adapter as ONEBOT_V11Adapter

onebot.init()
app = nonebot.get_asgi()

driver = nonebot.get_driver()
driver.register_adapter(ONEBOT_V11Adapter)

nonebot.load_all_plugins([""],["src/plugins"])

nonebot.run(access_log=True)

if __name__ == "__main__":
    nonebot.logger.warning("Always use `nb run` to start the bot instead of manually running!")
    nonebot.run(app="__mp_main__:app")

```

也可以直接使用 `nb-cli` 生成的 `bot.py` 启动。

启动的时候尽量使用虚拟环境启动，不然不保证不会报错。

编写的插件都放在 `src/plugins` 文件夹下。类型有两种，一种是单文件型，还有个包(Package)型。前者适用于简单的插件，后者则适用于编写功能较为繁杂的插件，两者区别是单文件型直接在plugins文件夹下放置.py文件，而包类型则是在plugins文件夹下新建以插件名命名的文件夹后，再在里面放置.py文件（注：每个包文件夹内必须包含**\__init__.py**文件！即使里面为空也要有！）。

本教程配套视频教程请看我的B站主页：[MRSlou1](https://space.bilibili.com/634651362)
