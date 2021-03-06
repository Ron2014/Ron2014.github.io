---
layout: post
title: Hello World
date: 2020-05-15 02:57
description: 如果赛博不再朋克……
toc: true
tags:
 - blog
 - cyberpunk
---

欢迎来到 cybeureka's pensieve. 中文名：赛博尤里卡的冥想盆。

# 网站的由来


很早的时候痴迷时间管理的理论，自己搭了个 [redmine](https://redmine.cybeureka.com/) 用来记录业余时间想做的事情。在面试某工作室的时候，被路人朋友附了个域名，之前都是 `IP地址:端口号` 的方式访问。刚好最近在复习计算机网络理论，看到 `DNS域名解析` 的章节，总觉得该是时候拿点东西来练练手了。

我的域名 cybeureka 是在 [namesilo](https://www.namesilo.com/) 上购买的，年费9刀，而且续费不加价，这是我综合考虑了 [阿里万网](https://wanwang.aliyun.com/) 和 [godady](https://sg.godaddy.com/) 的选择。godady 的续费价格是首年的几十倍，真的是个爹。

用 [redmine](https://redmine.cybeureka.com/) 来练手 DNS解析，apache2 服务器跑 web，[nginx 做反向代理](https://www.cnblogs.com/fanzhidongyzby/p/5194895.html) 实现域名访问可以跳到 redmine 默认的 3000端口号。再后面就是参考[这篇](https://linuxize.com/post/secure-nginx-with-let-s-encrypt-on-ubuntu-18-04/) 的流程用免费的证书做HTTPS支持。（PS. digitalocean 的教程并没有work）

之前看到大牛的博客叫 [The Brain Dump](https://http://floooh.github.io/)，不禁想起邓布利多的冥想盆，这让我有了开博客的打算——一个集中记录自己成长的地方。后来发现 github.io 就是所谓的 github page，免费开。域名都有了，跟着[教程](https://sspai.com/post/54608)直接做 CNAME 的映射即可。博客的框架 Jekyll 和 Hexo 两个最火，前者又是 github page 原生支持，且主题众多。

- [http://jekyllthemes.org/](http://jekyllthemes.org/)
- [https://jekyllthemes.io/](https://jekyllthemes.io/)
- [http://themes.jekyllrc.org/](http://themes.jekyllrc.org/)

作为小白也不想花太多经历在选择上了。陆陆续续，这个网站的影子就有了。

看了[主题](http://jekyllthemes.org/themes/jekyll-clean-dark/)的[说明文件](http://pavelmakhov.com/jekyll-clean-dark/2016/08/analytics-tags-comments/)，也知道还欠缺很多东西。

- 评论 [Disqus](http://disqus.com)
- 分析工具 [Google Analytics](http://www.google.com/analytics/)
- 另一个分析工具 [Yandex Metrica](http://metrica.yandex.com)

以后可能还会研究下 python 的 Django 怎么做 web，不过这些都是后话了。

相信 pensieve + github + PicGo（图床工具）将会让我更加善于表达自己吧。

![img]({{ '/assets/images/pensieve_of_dumbledore.jpg' | relative_url }}){: .center-image }*pensieve of dumbledore*


# 代替方案？


你可能会问：

- 微信公众号 不香么？
- CSDN 不行？
- github 上 repo 的 wiki 不行？
- github 版本的 twitter —— gist （其实就是平时用来发代码片段记录灵感，或者发发牢骚的短文本平台）不行？

让我抒发一下使用感受。

## 页面版本的文本编辑普遍难用

微信公众号操作有两种方式，一是原平台的编辑页面，而是135Editor。两个都用过之后，虽然135Editor差强人意，但是还是太难用。

因为这个世界上，最理解编辑器应该怎么设计的，其实是程序员（文字工作者中的高级物种）。

一个优秀的编辑器应该支持：

- 语法解析+高亮+配色
- 关键字导航（跳转到函数、变量定义处）
- 浏览历史（在导航中方便返回上一次光标的位置）
- 自动补全
- 语法纠错
- 搜索和替换
- 插件支持（没错，搜索、使用甚至DIY插件对程序员来说才是真正的“淘宝”）

这些对任何一个（代码）编辑器（Vim / Sublime Text / PyCharm / Visual Studio Code）来讲都是家常便饭，哪里还轮的上微信、135编辑器。

所以哪怕是公众号的文章，我其实都是在网易云笔记或者 vs code 中写一遍 markdown 格式，然后再贴到公众号页面修改格式的，所以非常麻烦。

makedown 用顺手之后，很难再回到以前那种低效的编辑方式了。

![img]({{ '/assets/images/code_editor.jpg' | relative_url }}){: .center-image }*code editor is the REAL colorful world!*

## 公家的东西必有强硬的审核机制

目前 CSDN 支持 markdown，但是作为主流平台，它和微信、B站等有个审核系统，发布或者修改，提交后动不动要几个小时链接才会生效。

说其强硬，从感受上理解就是：当你点击提交按钮之后，你并不会立即看到它的展示效果。更有甚者，你提交之后就没有下文了（没过审）。

抛开审核机制的效率不谈，这种 **默认所有人都为坏人的做法** ，会强烈挫伤文字工作者的热情。如果提交的文章很长，准备了很长时间，这种暴击感会更强烈。

作为程序员，本来就不太喜欢非代码类的东西（没有关键词的语法高亮，感觉自己像瞎子）。在公家平台上写非代码，更有一种 *我本来不太擅长啪啪啪，现在我在啪啪啪了，但我超级害怕啪地TA不满意* 的感受。

这种精神枷锁让我无法享受表达自己的快乐。不要审核，那就自立门户。

而且，只要 github page 更新页面的效率超高（应该是没有审核）。

## 自家的东西想改就改，说话可以不算话

程序员做久了，diff 自己的现象会越来越多。代码提交到 SVN 上根本就不代表工作完成，一个追求真理的程序员脑海中还会萦绕这些问题：

- 我的代码，可读性怎样？别人能不能看懂。
- 是不是正确的，边界考虑全不全，要不要再多写几个case？
- 只有这一种方法么？我要不要再查一查论文，试试别的思路？

代码如此，文章亦如此。所以，程序员有很强的编辑已发布文章的诉求啊，可是微信公众号不care。

每次发布仅能一次修改，而且不超过10个字符。

这让我大量文章都宁愿躺在后台仓库里，不见天日。

## 碎片化+自定义

这一点其实 github 已经满足了，但是 repo - wiki 的形式感觉太重度。

如果我只是想讨论一个函数或者一个算法呢？我觉得这些随便化的东西应该有个集中处理的地方，所以自建了 [issue-test](https://github.com/Ron2014/issue-test)。

这样哪怕是我写了非常碎片的代码，验证了一个非常细节的猜想，也有地方保存。所有的灵感以 testXXX/README.md + source 的结构保存。

可我还是觉得，**展示不够** ：

github 上白底黑字的方式展示 markdown 不太接受，对眼睛不够友好。它为什么不在个人设置里面增加一个修改主题的方式呢？就像 stackoverflow 一样。

github 改不了主题，但 github page 博客是自由定制的。

最后提一下，gist 既然这么像程序员版的 twitter，毫无疑问，它被墙了（似乎更适合做为吐槽工具了）。

![img]({{ '/assets/images/alita0.gif' | relative_url }}){: .center-image }*cyberpunk*

# 关于 cybeureka 的由来


cybeureka 是个合成词：cyber（控制论）+ eureka（我发现了）。

毕竟 cyberpunk（赛博朋克） 文化占据了主流媒体，但我觉得 cybeureka 才是这个世界真正的样子，也是 programmer 的信仰。

cyberpunk 想要表达的是科技垄断、两极分化，世界濒临崩塌的设定。

最早可以追溯到《1984》中那个被描绘成人人都被严密监控，没有任何言论、行动自由的世界。

《黑客帝国》中的 Matrix 统治下的 “缸中之脑”；《银翼杀手》中讨论的仿生人的生存的自由；《西部世界》中机器人的迷宫和操控人类命运的罗波安；《黑镜》中各种科技外衣下包裹的悲剧。

我甚至想到，最早的科幻小说、也是恐怖小说《弗兰肯斯坦》描绘的造人的科学实验，其实在现实中从未停歇。它早就不是中世纪时代背景下，尸体拼接+电力驱动这种神话般的方式来创造生命，而是另一种形式————计算机发明之后很快提出的“人工智能” ———— cyberpunk 版的弗兰肯斯坦。

我无时无刻不在恐惧，恐惧某天机器人作为更优秀的物种在商业活动中崭露头角，恐惧那个真的可以预测未来的罗波安被研制成功，恐惧我们都未必了解自身的情况下却早被大数据看穿……这种恐惧是让我学习计算机的 cornerstone，我也是在阅读、实践的过程中不断去证明，科技并非冰冷的，它创造的幸福要比悲剧多得多。就拿真实历史上1984年发生的事情，和小说中的景象大相径庭。

![img]({{ '/assets/images/block_chain_sex.jpg' | relative_url }}){: .center-image }*first head tech improve sex first*

有人不满被商业垄断的操作系统，自己照着开放的API定义，逆向推理出新的操作系统的实现。这就是UNIX和Linux的历史，以至于后面越搞越大的GNU，倡导开源的自由软件基金会FSF，到后来的 github 知乎 区块链。为什么 google 什么都能找到？正是因为互联网上的活动大多都和商业、政治无关，很多人都在无偿地做着喜爱的事情：分享一篇教程，剪辑一个视频，开发一个软件……这是个无比开放的世界，和过去的人们描绘的未来完全不一样。

你会庆幸，互联网不像基督教、不像儒家思想沦为政治工具。但真要去回溯互联网的发展历史，看到它从军用、实验室走出来，各种组织、国家为了实现硬件、软件兼容不得不制定统一的协议，协议和其草案甚至拥有“请求评论”（RFC）这么开放的名字，它也必然不会沦为政治工具。当然好的东西被坏人利用，并不能作为诋毁这个东西的缘由。菜刀杀人，菜刀本没错，这是社会工程学的问题，而不能归咎于科技的错误。大家都说，世上善恶的总量不变。如果真的害怕坏人掌握技术，那就多让一个好人去融会贯通它，比如，你。

没错，那些恐惧，会伴随着我的睡眠、购物，以及无意义的追求（比如房子、婚姻）越发加重，只有在阅读、写作、刷题、游戏时才能减轻。如果这些无济于事，那就打开 google，打开B站视频，打开 Visual Studio Code，执行 g++ devenv make cmake fips 直到 warning error 都为0。

最终你会发现，一切的工作都是文字编辑。而这些文字中，必有闪光的词语，Aloha，Magi，Present，Cyberpunk，Eureka，Pensieve，如同一道道咒语，闪过历史的苍穹，就像每个人心中必然存在的光芒一般。

![img]({{ '/assets/images/expecto_patronum1.jpg' | relative_url }}){: .center-image }*Expecto Patronum: Eureka VS Terror*

我很感激这些科幻小说、电影不断去挖掘科学的人文内涵，但有些不合逻辑的招黑剧情确实显得太过故意。举个简单的例子，《西部世界》中的机器人觉醒，更大的可能是，最早出现在某个角色扮演类游戏（RPG）中的数字人类，或者某个实验性质的虚拟乐园中可以进行时间压缩式的上千亿次训练，这样根本就不会发生到现实世界杀人的惨剧。如果将来很多工作被机器人代替，那个时候，AI的调试和维护将需要很多的人力。

毫无疑问， cyberpunk 的基调过于悲观了。很多时候面对历史，我们本应满怀自信和感恩：

当第一个电话铃声响起。

当第一封报文从一台机器发送到另一台。

当无人机缓缓升起。

面对科技的发展，最该做的事情，也最自然的事情，还是惊叹得说出那一句：

Eureka!

而那些未曾翻开的篇章，作者们可能各个都有着 Roy 那深邃而高傲的眼神：

我领略的东西你可能从未见过，甚至不敢相信……

![img]({{ '/assets/images/roy.gif' | relative_url }}){: .center-image }*Roy in Blade Runner*