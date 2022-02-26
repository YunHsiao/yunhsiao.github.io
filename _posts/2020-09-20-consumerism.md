---
layout:     post
title:      "2020 GameJam: Consumerism"
date:       2020-09-20 00:18:00
author:     "YunHsiao"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - GameJam
---

![](/img/GGJ-20.jpg)

## GameJam
GameJam 比赛一直都是公司里我们几个圈里人每年都热衷于搞那么一两次的一个活动，短平快的节奏非常适合我们在工作之余腾一个周末出来玩一把。<br>
这次组了引擎组五位程序同学，最后时刻 panda 又拉来了一位音乐外援妹子，我们的观音山老年活动中心就这么准备开张了。

## 观音山老年活动中心
题目发布时我们还没下班，但看着直播放出题图就开始明目张胆地划水，一头雾水有一句没一句地开始解读，大概是有人从满地的快递箱先看出了图里讽刺消费主义的味道，还在加班的我才忽然觉得找到了图的灵魂，有了可以做文章的地方。这个观察非常深刻，当我从这个视角再去看这幅图时，其实基本所有的细节都有了明确的主题指向（快递是消费品，地面的裂缝是内心的欲望沟壑，墙外的蚂蚁是让人心痒的诱惑等等），我很喜欢这个视角，所以心里就希望往这个方向去探索，看能做点什么，去和作者这幅画要表达的东西（至少是我们看到的东西）共鸣。

## 共鸣
首先冒出来的想法肯定是最简单直觉性的，结合之前一个有趣的 shower thought，如果去具象生活中那些看不见却无处不在扮演重要角色的元素，比如 wifi 信号，那我们就是在一个个球型安全区之间穿梭，在安全区时回复快乐值，一旦离开就要消耗，由此去做一个平台跳跃之类的东西。但继续想下去，好像除了这个具象化的 idea 不错之外，很难落到我们 5 个程序 48 小时能做好的游戏性上去，并且或多或少有种格局太小的感觉，和主题联系也不强。<br>
接下来是一段欢乐的不靠谱点子乱丢大会过程，我们否定了诸如蚂蚁上树，坐着椅子过河，拿快递补墙等一系列天才想法后，忘记是怎么说到这样一句，“如果我们不做任人宰割的韭菜，而从万恶的资本家视角去展示这个系统会怎么样？”

## 万恶的资本家
这听起来更抽象了，但我看到了背后的潜力：这完美解决了之前 wifi 球那个想法欠缺的格局！与其展示一个个体在整个环境下的挣扎让玩家去勉强共情，不如直接把屠刀放到玩家手中，真正体会一把收割的角色，从收割过程中去理解资本机器的运作，进而在生活中不再被同样的手段迷惑。<br>
另一方面，结合我们几个的实际情况，我们的长项是逻辑（同时短板是美术），设计一个相对复杂精巧的系统更接近我们的主场，这正符合资本机器冰冷严谨的运作风格；引擎这回应该没什么理由不用自家的狗粮，所以基础设施的长项是骨骼动画（instancing），加上相对合理数量的粒子和物理，所以这回要做的东西，似乎越来越像……一个实时策略类游戏了噢。

## 实时策略
实时策略游戏的核心正是系统构建，虽然我们都没有真正专业地做过，但这听起来是一个我们可以尝试搞的东西！明确了思想路线和作品形态，接下来最重要的就只剩下一个很简单的目标：我们只要将市场和资本的运作剥离到它最纯粹的本质，建立合理的模型，在 48 小时里完成对它的展示和交互。Good! Easy, right?<br>
Well...<br>
事实上这部分是整个创作过程中最艰难的部分。这需要我们给出一个相当具有洞察力的答案，一个能够让人连脑子都还没转就能马上感受到扑面而来的布尔乔亚腐朽气息，看到这一切欢愉背后的造梦机器的答案，来回答这个早就在每个人脑海里却没人愿意碰的问题：所以到底什么 TM 是消费主义？

我们是参加 GameJam，不是在学院里探讨理论，所以，如果说这个问题的“标准答案”……谁在乎呢。<br>
但我们需要一个模型，一个最纯粹又在直觉上能让人亲近的情景，来让玩家“感受”。<br>
我们很快开始在新的视角下扩展之前 wifi 球的思路，把基本元素改为消费者和产品：消费者在这世界中大量随机分布着，每个消费者都有随机数量的“快乐值”，“欲望值”和“剩余价值”等几个基本属性。玩家作为生产者，拥有固定的初始资本，然后要随时决定要在哪里投放什么产品，来尽可能榨取最多的“剩余价值”。从这开始，越来越多的规则加入，每款产品都有不同的属性定位和特征，比如投放生活必需品就增加快乐和剩余价值但降低欲望，投放奢侈品就增加欲望收割剩余价值，投放抖音可以增加快乐和欲望等，消费者欲望越高购买越积极，但快乐越高欲望又会降低得更快…… 等等等等，随着规则的逐步扩大，很快就没人能理得清了，更不用说动手实现了。（“欲望降低得更快”？听起来我们是在讨论二阶导数……）<br>
显然我们陷入了受限的思路，被这个世界无限丰富的细节迷了双眼，还没有找到真正的灵魂。<br>
这番有趣但令人头疼的尝试后慢慢冷静下来，我们考虑很多这里提出的基础设定是没什么大问题的，关键在于没有找到足够纯粹的那个核心，那个不可再分的，消费主义的最小子集。看着我们画了满白板的系统结构，我盯着“欲望”两个字看了很久。

### 欲望
欲望，是人性中不可分割的一部分。从碰碰车场外满眼星星的三岁小孩，到看着老伴照片回想这一生陪伴，幻想还能再见一面的迟暮老人，它都是最真实又锋利的一份驱动力。<br>
欲望没错。但欲望是瞬息万变，起落无常的。这意味着它非常容易受环境影响，或……控制。<br>
我们的时代是自由的。自由意味着每个人都可以追求他的欲望。这意味着一个巨大的市场，掌控了更多人的欲望，就掌控了更多……资本。<br>
没有什么特别的欲望？“没关系，我帮你找，你看这个怎么样？”<br>
喜欢什么很简单的东西？“为什么不试试这个更复杂精妙的升级版？”<br>
偶然发现喜欢做一件什么事情？“为什么不再多做几次呢？”<br>
我们对这样的话术太过熟悉，以至于我们几乎要忘记它们最初的最初，可能并不是你自己想要的。<br>
但谁在乎呢，我想要，就这里就现在就想要，我就……买了。

像是一道闪电照亮了整个房间，我似乎一下看到了我想找的东西：<br>
广告。

### 广告
广告是一个再完美不过的消费主义具象化的落脚点，它是一切纷繁热闹背后的推手，一曲宏大的交响乐最基础的定调。<br>
如果我们的玩家是资本家，要迎战一个目标市场，他手边永远无法放下的武器里，一定有广告。<br>
广告创造欲望，广告引导欲望，广告升级欲望。它是我们会一个接一个下单取快递的原因，它像被蚂蚁一般无时无刻不在挠得我们心头犯痒想要更多，它是我们明明已经有了那么多盒子可心中的空洞和裂缝还是越来越大的原因。<br>
消费主义，不过如此。

找到了灵魂，我们的实时策略游戏构建起来就明确和轻松多了，所有的游戏性全都围绕广告开展，消费者也只需要一个“购买欲望”的属性，初始状态下每个消费者可能有不同的起始购买欲望，欲望值达标就会向资本家走来，走到位置就会购买成为韭菜，但欲望会持续衰减，还没走到欲望减到阈值以下就会冷静下来，回到正常状态。玩家的任务就是在有限的成本下，要平衡出货和广告投放，只有货物没有广告是没有人买的，只有广告没有货物也会为产品带来负面口碑，影响更多人的购买欲望。

周五晚上明确了这个方向，接下来的两天不断细化打磨试玩迭代，调整到所有我们认为不切实际收割方案都无法顺利得逞，经过大家在各自主场齐心协力的合作（每个人都有很重要的贡献，非常了不起，我为所有人骄傲），最后的结果就是另一个大家通过介绍视频和比赛结果听到的故事啦。

---

[项目介绍](https://www.youxibd.com/gamejam/cgjcyber2020/detail/439)<br>
[项目 Github 地址](https://github.com/YunHsiao/GGJ-20)<br>
[Cocos 公众号采访](https://mp.weixin.qq.com/s?__biz=MjM5ODAxNTM2NA==&mid=2659653459&idx=1&sn=ac4f17fb0adc2e5cb47158eac893f7d3)<br>
[2020 WePlay 文化展](https://show.bilibili.com/platform/detail.html?id=30475)