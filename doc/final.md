---
layout: page
mathjax: true
permalink: /2019/projects/p07/final/


---

## 基于用户行为和番剧特征的b站番剧推荐

#### 成员及分工

| 成员   | 分工                               |
| ------ | ---------------------------------- |
| 李嘉盛 | 系统设计，算法实现，文档编写       |
| 曹昊天 | 关联规则挖掘，数据分析，数据可视化 |
| 万凯斌 | 数据爬取，数据预处理，模型评估     |



### 1. 题目介绍

在b站、爱奇艺等视频网站中，用户观看视频的数量往往关乎着网站的收益。为了增大用户观看量，视频网站会使用推荐系统向用户推荐其可能感兴趣的视频，引诱用户观看。因此，一个高质量的推荐系统不仅能为网站带来更多的收益，也能为用户带来良好的体验。

![source_code](https://ws1.sinaimg.cn/large/005J7jqOly1g5nx5epfstj30qu0a1q3l.jpg)

在2019年4月的一次Bilibili源码泄漏事件中，B站的视频推荐算法被人解读了出来，其中对于视频的各类特征的权重占比仅针对视频的播放属性而言，例如播放量、点攒数、分享数等，如果直接套用到番剧推荐上，缺少对番剧本身特有的特征关联，如番剧的声优阵容，画师等。

在本项目中，我们设计并实现了一个b站番剧推荐系统，为了保证推荐的准确性，我们综合用户行为和番剧特征两个方面来进行推荐。其中，用户行为包括用户的关注、点赞、收藏记录等，番剧特征包括番剧的标签、评分、声优、画师阵容等其它属性。最终，我们将用b站的真实数据集来评估系统的推荐效果。



### 2. 模型描述

#### 关联规则挖掘

关联规则挖掘是一种基于规则的机器学习算法，该算法可以在大数据库中发现感兴趣的关系。“尿布与啤酒”是一个典型的关联规则挖掘的例子，沃尔玛为了能够准确了解顾客在其门店的购买习惯，对其顾客的购物行为进行购物篮分析，想知道顾客经常一起购买的商品有哪些。沃尔玛利用所有用户的历史购物信息来进行挖掘分析，一个意外的发现是："跟尿布一起购买最多的商品竟是啤酒！

关联规则挖掘算法不仅被应用于购物篮分析，还被广泛的应用于网页浏览偏好挖掘，入侵检测，连续生产和生物信息学领域，以及本项目所做的推荐系统。

我们所选用的关联规则挖掘算法时FP-Growth算法，算法的实现主要分为三个步骤：建立项头表、建立TP-Tree、基于FP-Tree进行关联规则挖掘。

##### 建立项头表

FP树的建立需要首先依赖项头表的建立。我们第一次扫描数据，得到所有频繁一项集的的计数。然后删除支持度低于阈值的项，将1项频繁集放入项头表，并按照支持度降序排列。接着第二次也是最后一次扫描数据，将读到的原始数据剔除非频繁1项集，并按照支持度降序排列。

![](https://ws1.sinaimg.cn/large/0061Qobaly1g5nxi2bipzj305a08fq30.jpg)

如上图所示为一个建立好的项头表，其中A、C、E、G、B、D、F为挖掘出的频繁项集。

##### 建立FP-Tree

有了项头表和排序后的数据集，我们就可以开始FP-Tree的建立了。开始时FP-Tree没有数据，建立FP-Tree时我们一条条的读入排序后的数据集，插入FP-Tree，插入时按照排序后的顺序，插入FP-Tree中，排序靠前的节点是祖先节点，而靠后的是子孙节点。如果有共用的祖先，则对应的公用祖先节点计数加1。插入后，如果有新节点出现，则项头表对应的节点会通过节点链表链接上新节点。直到所有的数据都插入到FP-Tree后，FP-Tree的建立完成。

![](https://ws1.sinaimg.cn/large/0061Qobaly1g5nxi2ot4fj30go0ddgo6.jpg)



如上图所示，是一个建好后的FP-Tree

##### 基于FP-Tree进行关联规则挖掘

得到了FP树和项头表以及节点链表，我们首先要从项头表的底部项依次向上挖掘。对于项头表对应于FP树的每一项，我们要找到它的条件模式基。所谓条件模式基是以我们要挖掘的节点作为叶子节点所对应的FP子树。得到这个FP子树，我们将子树中每个节点的的计数设置为叶子节点的计数，并删除计数低于支持度的节点。从这个条件模式基，就可以递归挖掘得到频繁项集了。

![](https://ws1.sinaimg.cn/large/0061Qobaly1g5nxi2lyj2j30n009x0uh.jpg)

上图右侧展示了E的条件模式基，递归挖掘到E的最大频繁项集为频繁3项集。

#### 推荐算法设计

##### 基于关联规则直接进行推荐

在通过FP-Growth算法得到了关联规则后，我们根据关联规则的置信度对所有规则进行了排序。在对某个用户进行推荐时，将用户的过往观看记录与关联规则的前项进行匹配，从而得到备选关联规则，再从备选关联规则中选择置信度最高规则的后项作为给该用户推荐的番剧。

##### 基于属性权重的改进 

为了能使推荐系统的准确率更高，我们不把置信度作为评价关联规则的唯一标准，而是将番剧评分、追番用户数、播放量及番剧的画师、声优阵容等番剧属性也纳入了考虑范围内。我们为置信度和上述属性设置了权重，并通过调参来不断调整权重分配，使得推荐系统能更加精确。

### 3. 数据处理

#### 缺失值处理

缺失值主要来源于爬虫爬取数据过程中的意外操作，如用户的隐私设置了非公开，番剧被下架，用户本身处于[小黑屋](https://www.bilibili.com/blackboard/blackroomrule_v16.html)状态，都有一定可能导致数据的缺失。此外，由于B站本身的反爬机制，所以在高速爬取时会被IP限制，所以使用IP池的方法进行绕过时，代理IP的不稳定性也可能导致一定量的数据缺失，但是比例较小，所以可以忽略不计。

对于这类缺失值的处理方法一般采用直接丢弃的方法，因为缺失的数据占比较小，且大多数缺失数据本身不具有特征倾向，所以直接丢弃即可。但是对于被下架番剧的信息缺失，可以考虑从百度快照等缓存网站获取最近一次的缓存信息进行补充。而由于代理IP出现的信息缺失情况，在爬虫的中间件部分设置了缺失重爬的逻辑，所以只需要在最后的爬取结果中进行一次去重操作即可。



#### 非法数据处理

B站本身的数据其实也存在一定的错误率，例如在声优及职员列表中，经常出现中英符号混用，非严格的职员-角色格式，以及已注销用户仍然可以得到有效数据，这部分数据其实不应该出现在最终的用户数据中。由于这类情况出现较少，且缺少可以过滤的规则，所以只能手动对这部分数据处理。

此外，由于对于用户数据的爬取是bfs的方式进行的，所以得到“低等级”用户有非常多，有很大一部分这样的用户是广告号，所以为了尽量减少这类账号对数据整体的影响，设置了一定的规则进行过滤，如2级以下的用户数据将不被归纳到最终的数据集中，此外，[节操值](https://www.bilibili.com/blackboard/help.html#%E8%8A%82%E6%93%8D%E7%9B%B8%E5%85%B3?id=b177a87e1e714e2ea734f8a64f4e1da2)低于阈值（50）的用户也将被规则过滤出去。



### 4. 结果分析

首先我们利用FP-Growth算法挖掘出了所有的关联规则，并根据置信度对所有关联规则进行排序，以此来作为推荐的标准。

![](https://ws1.sinaimg.cn/large/0061Qobaly1g5o1fc649jj31bj0l2wmj.jpg)

上图显示了置信度最高的20条关联规则。在以置信度作为唯一的评价标准对用户进行推荐时，推荐的准确率约为71.5%。

之后，我们将用户及番剧的某些属性也纳入了考虑范围内，包括番剧的得分、观看量、追番人数、画师声优整容等。规则的置信度和以上属性各占一半的权重，并根据改权重对每一条关联规则进行重新打分。



![](https://ws1.sinaimg.cn/large/0061Qobaly1g5o1fccqrqj31ae0kvgtj.jpg)

上图为对关联规则重新打分后，分数最高的20条关联规则。可以看出与完全依靠置信度相比，结果有了很大不同。我们还尝试不断迭代以调整权重，使得推荐准确率能够达到最高。经过调整，推荐的准确率有了小幅度的上升。

2019年4月22日，bilibili发生了代码泄露事件。我们阅读了bilibili部分推荐规则的算法，发现其自身的推荐系统分为两类，一类是针对视频的推荐系统，另一类是针对文章的推荐系统。在对视频的推荐中，bilibili考虑了视频的    硬币、收藏、弹幕、评论、播放量、点赞数、分享数、更新时间等属性；对文章的推荐中，bilibili考虑了专栏文章硬币、收藏、评论、阅读量、点赞数、分享数、更新时间等属性。bilibili本身的推荐系统所采用的属性和设置的权重具有一定的参考价值，然而，番剧的某些特性不同于一般的视频。我们考虑到，很多用户观看番剧的原因是喜欢该番剧的某些幕后工作者，比如画师、声优、导演等，因此，在我们这个番剧推荐系统中，我们对番剧工作人员也设置了一定的权重，以提升整体的推荐效果。

### 5. 小结

通过本次大作业，小组各成员在对数据挖掘的算法理解上又深了一步，相比于B站源码中所透露出的推荐算法权重来说，本课题中所提出的推荐算法涉及的特征更为丰富，在原有基础上结合动漫爱好者的具体喜好，总结并使用了番剧这类视频所具有的额外特征，即声优与画师阵容等，通过关联规则的挖掘，结合不同特征的权重调整，从而得出一个更为综合的番剧推荐系统。





###### 项目地址：[BilibiliCrawler](https://github.com/crazywkb/BilibiliCrawler)



###### References：

1. [读懂B站代码：防做号，重激励，推荐机制的秘密](https://mp.weixin.qq.com/s?__biz=MzUyNDQyNTI1OQ==&mid=2247484668&idx=1&sn=50de4c62baa059c294901c41349b7b90&chksm=fa2cc5b8cd5b4cae2e4ec68e2167611241acff1539c93db25d55f4faf196d48801bc9b0a2e8b&mpshare=1&scene=1&srcid=0804lARH0MTbiSCEJo1GTBOi&sharer_sharetime=1564896087595&sharer_shareid=d5b954a726f6c8fbfe215b73d3a594ca&key=05df2f32dfb7363608bde6d8cd4c0f3f77959d143b6eeacb8188aae90bd9402e9d2f9b2fe47ea43eac45cffe4329e5ce954b5b6fe4a05615fbba2c73373ee307d5e024edbbb3c151a6aefafae7bacef3&ascene=1&uin=Mjg4OTM3MDAyOA%3D%3D&devicetype=Windows+10&version=62060833&lang=zh_CN&pass_ticket=DxkIs8i3l7ws22V%2FS1imz7siLzxxRsVdhIlm6YCFtdC1vPx3bYFBUxk%2BLEAbuKpS)
2. [FP Tree算法原理总结](https://www.cnblogs.com/pinard/p/6307064.html)

