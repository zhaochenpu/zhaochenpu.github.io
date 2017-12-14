title: 来谈谈APP提示弹窗交互设计
date: 2017-06-14
tags: 产品
---

移动端常见的提示弹窗可分为3类：提示框、泛Toast 和消息推送。<!--more-->

###1.提示框
提示框是一种打断用户操作行为的弹窗，用户必须做出确认、取消等操作才能进行下一步。

常见的用法有功能引导（但别指望傲娇的用户会认真看）、弹出广告信息或者重要通知（虽然用户未必觉得有卵用还感觉有点烦）、提醒用户当前操作会发生什么事情（告诉用户别手贱，想好了再点）、耗时操作提示（安慰客官别着急，菜正在做着）、进行文本输入或功能设置（少跳转一个页面）。

####1.1 功能引导
功能引导一般是APP第一次安装、UI交互改版或者更新重要功能时对用户进行使用引导，增加用户对某功能的认知度，减轻用户对APP陌生感，避免了用户面对复（nǎo）杂(cán)交互时的一脸懵逼。
![天猫用户引导](http://upload-images.jianshu.io/upload_images/828721-346a03a31609514c.gif?imageMogr2/auto-orient/strip)

![](http://upload-images.jianshu.io/upload_images/828721-849b5204c1b41e02.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
####1.2 信息弹窗

广告的弹窗最好是能抓人眼球。用户不是烦广告，而是烦对自己没用并且没趣的广告。
![](http://upload-images.jianshu.io/upload_images/828721-3b71a214e3152c5d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
####1.3 对话框

有一些操作是无法挽回的，所以需要增加一个确认提示，防止用户误触。
![“客官，三思啊”](http://upload-images.jianshu.io/upload_images/828721-84ef63ce4bc84348.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

应用正常运行的时候缺少权限会使某些功能无法正常使用，但是乐视你一个薯片……哦不对，视频软件要我定位、通话这种敏感权限，不给还强制关闭APP，这是想干嘛？还是卸载了吧。
![负面典型，没说明申请权限原因](http://upload-images.jianshu.io/upload_images/828721-bc903d9c4bcb5d2b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

申请权限或者应用更新的提示框最好能把事情写详细，让用户明白授权权限或者升级APP对自己有什么好处。UC浏览器耍了个心眼，把取消按钮故意设计的不显眼，视觉上引导用户去点击“立即体验”按钮。

![UC浏览器的升级提示](http://upload-images.jianshu.io/upload_images/828721-5ede30034365d9e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####1.4 操作框

如果用户需要进行简单的设置或输入，可以用弹出操作框的形式减少界面跳转。但是操作框大小有限，如果是复杂设置的话最好还是跳转新的页面。
![](http://upload-images.jianshu.io/upload_images/828721-61ff81e31214c85a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
####1.5 loding弹窗

用户的耐心比金鱼的记忆时间还短。如果应用3秒没反应用户就会疑惑，5秒没反应用户就可能会觉得是出问题了而去尝试关闭。这时候就体现出了一个有趣的loding界面的重要性了。
![](http://upload-images.jianshu.io/upload_images/828721-7cb60f9a94b8eb8f.jpg?imageMogr2/auto-orient/strip)

###2.泛Toast
为什么是泛Toast 呢，因为现在Toast 的玩法越来越多了。
####2.1 设计规范中的Toast和Snackbar
Android官方设计规范里，Toast 是一种主要用于提示系统消息的轻量级控件，显示一段时间后自动消失，不包含操作也不能从屏幕上手动关闭，不会打断用户的操作，多个Toast 可以叠加出现。

之后Android针对轻量级反馈操作在Material Design中新增加了一种叫Snackbar 的控件。Snackbar 以一个小的弹出框的形式，出现在手机屏幕最底部，左侧为提示文本，右侧是操作按钮。Snackbar 不仅会超时自动消失，用户也可以滑动将其关闭，屏幕上同时最多只能显示一个 Snackbar。

![微信中对Snackbar的使用很符合设计规范](http://upload-images.jianshu.io/upload_images/828721-d19bb6d723e1073f.gif?imageMogr2/auto-orient/strip)

Android官方设计规范里Toast 和Snackbar都应该保持简约，不对用户造成过多的打扰，所以不建议提示文本过长（显示时间有限，太长用户看不完），更不建议在其中增加图片和过多按钮。

####2.2 反面典型的Toast和Snackbar
但是有句名言说得好，规则就是用来打破的。
![规则，就是用来打破的 —— Jinx ](http://upload-images.jianshu.io/upload_images/828721-eea074e6bcc898c1.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![小米应用市场APP安装完成提示](http://upload-images.jianshu.io/upload_images/828721-b3c58c5c5351ae5a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

小米应用市场安装完APP后出现的Toast ，功能上是方便了用户，但是设计上不太好看。


![豌豆荚升级提醒](http://upload-images.jianshu.io/upload_images/828721-0e4f006aa69b8bf8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

豌豆荚的新版本升级提示用Snackbar 的形式出现，左侧有两个操作按钮。因为豌豆荚的这个Snackbar 是不自动消失的，除非用户点击操作按钮或者滑动才会关闭，所以不会出现用户正在纠结两个按钮该点哪个而Snackbar 超时自动消失的情况。但是一句“新版本已准备好了”就想让用户去升级，看来这版本也没多重要。



####2.3 泛Toast

![B站客户端投币](http://upload-images.jianshu.io/upload_images/828721-d2eae4e1df356d8e.gif?imageMogr2/auto-orient/strip)

上面bilibili这个投币的交互设计挺有趣的，如果投币成功后弹出的Snackbar增加“再次投币”的操作按钮，一定能增加用户投币的数量。


很多APP为了更少的遮挡内容，将Toast移到了顶部。

![微博刷新提示](http://upload-images.jianshu.io/upload_images/828721-cabb0fdc533c5735.gif?imageMogr2/auto-orient/strip)
![QQ音乐收藏交互](http://upload-images.jianshu.io/upload_images/828721-e0cea9ce92e3744c.gif?imageMogr2/auto-orient/strip)

![QQ中的断网提示](http://upload-images.jianshu.io/upload_images/828721-9ad95a97f2733eaa.gif?imageMogr2/auto-orient/strip)

传统的Toast样式低调，很容易被用户忽略，我们可以按照提示内容设计不同的颜色，也使得APP变得更加生动有趣。

![不同类型提醒设置成不同颜色](http://upload-images.jianshu.io/upload_images/828721-e5c2ad3ee6363edc.gif?imageMogr2/auto-orient/strip)


###3. 消息推送

消息推送的使用场景一般是应用处于非活跃状态（未启动或在后台里），为了避免用户错过重要信息而通过系统发出提示，点击消息可跳转到应用内相关页面。

消息推送也经常被用来促进用户活跃度，消息推送的频率和质量反映了一个团队对用户心理的掌握程度。优秀的运营应该对用户群进行详细分类，投其所好才能吸引用户点击，否则用户就会烦的禁止APP推送。


![彩云天气的降雨提醒](http://upload-images.jianshu.io/upload_images/828721-37fc0780b03ef1ed.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###4. 提示弹窗交互原则总结

有用、好用、有趣，我觉得这是交互设计中最重要的三点。

1、有用：没有反馈的APP毫无生气，而各种乱弹窗又会使用户心烦到恨不得立刻卸载。所以反馈的信息一定是有价值的，让用户知道刚才发生了什么，接下来需要进行什么操作，一些显而易见的情景可以省略掉提示。

2、好用：提示控件的形式一定要把握好，不同的提示类型最好要区别展示。对话框会打断用户的操作，一般是处理重要事件，做好用户引导操作；toast是轻量级提示，不会打断用户的操作流程，提示的信息一定要言简意赅，显示时间不宜过长；消息推送最容易成为垃圾信息的重灾区，如果不能决定到底推送给用户哪些信息时，可以在设置里增加一些推送开关。

3、有趣：有趣的事情，大家都会喜欢。我相信即使是很讨厌广告的用户，在看到一个有趣的提示时也会十分感动然后把它关闭；即使是很讨厌等待的用户，在看到一个好玩的等待动画时也会会心一笑，然后时间就不知不觉的过去了。
