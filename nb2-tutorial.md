# NoneBot2插件从零开始的教程  

### 学习要求  

- 熟练使用Python。内容包括但不仅限于安装，代码编写，调试等。本教程**不会详细的去讲解这些Python基础的语法点**，所以想在这里边学Nonebot边学Python的可以放弃了。想学Python基础的请自行去看课程，B站一搜一大堆。
- 熟悉常用编辑器。写Python的话一般主流编辑器都是VScode或者Pycharm，Nonebot插件项目的话一般都用的VScode(当然本项目为了方便管理用了Pycharm)，这里附上下载链接：[vscode - Visual Studio Code - Code Editing. Redefined](https://code.visualstudio.com/)
- 熟悉Nonebot2的部署与搭建。注意！注意！注意！本教程不是Nonebot2基础教程，而是插件编写教程，所以不要询问关于Bot的部署方面的问题，请自行解决。这里也推荐一个视频教程，感觉很适合入门：[【零基础搭建QQ机器人】开源、免费、纯新手向的nonebot2.0.0beta版讲解](https://www.bilibili.com/video/BV1aZ4y1f7e2)，还有配套的GitHub仓库：[使用NoneBot2搭建QQ机器人](https://github.com/Well2333/NoneBot2_NoobGuide)。
- 遇到问题请先自己解决。NoneBot2的官方文档等其实讲的都挺齐全了，虽然说对纯新手有点不友好。文档共有两个：[Nonebot2官方文档](https://v2.nonebot.dev/docs/tutorial/create-project)，和本教程所用的适配器[OneBot V11官方文档](https://github.com/botuniverse/onebot-11)。如果实在搞不懂再来问，当然问问题的时候不要求精通《提问的艺术》，但也请遵循标准提问规范，不要让人血压升高。
- 因为我其实Nonebot2到现在才玩了3个月左右，有些地方肯定讲的不是非常好，如果有讲的不对或者不好的地方欢迎指出或者补充，如果想一起交流技术的话那就更好了（看看到时候能不能建个群白嫖大佬们的技术（笑））。

### 项目配置

- Nonebot框架版本：2.0.0-Beta2
- Go-cqhttp协议版本：1.0.0-rc1
- Pycharm版本： 2021.3.3 Community Edition
- Python版本：3.8.1(Windows端)＆3.10(Ubuntu端)