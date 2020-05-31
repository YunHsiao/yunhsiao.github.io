---
layout:     post
title:      "It's not a bug, it's a feature!"
date:       2020-05-30 23:00:00
author:     "YunHsiao"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 骨骼动画，数据压缩
---

最近有 CP 反应骨骼动画播放效果错误的问题，在沟通和调试过程中发生了一些非常有趣的事情，几乎可以总结为是把一个低级错误的 bug 最终非常合理地整合成了引擎中的小亮点 feature。<br>
这里尝试梳理还原一下这次迭代过程中的几个核心问题。

## How it all begins
一切的起因是在 CP 项目中，为了节省包体，他们把游戏中所有怪物 prefab 中的骨骼节点树全都删掉了。<br>
![](/img/in-post/skeletal-animation-bug/skeleton-nodes.png)<br>
在怪物种类繁多的情况下，视骨骼数量不同，每个 prefab 里都可能有不少空间是在存骨骼树的节点信息，统一删掉它们的确能节省不少存储空间。正常情况下在 Creator 3D 中因为骨骼动画是默认开启预烘焙，所有的数据都会在加载时预先预采样并烘焙到贴图，运行时在 GPU 直接根据帧数采样即可，除更新当前帧数外没有任何计算和其他数据传递，挂点也有另外单独处理的接口，整个过程如果只考虑不带混合地播放各动画片段，的确不需要骨骼节点树，因此不会有问题，项目也一直在正常推进。

直到我们今天的主角，毒蘑菇被加入项目，开始浮现出一些奇怪的问题：<br>
![](/img/in-post/skeletal-animation-bug/mushroom.gif)<br>
保留骨骼节点树时，在蘑菇身体动画正常的情况下，蘑菇头上的刺莫名地没了动画。更诡异的是，删了骨骼节点树之后，竟然效果就对了。

## We are the professionals!
先忽略删了节点树就对了的情况，考虑蘑菇整体是一个 drawcall，因此基本确定是底层实现出了问题，拿来排查一下，很快有了结论：<br>
![](/img/in-post/skeletal-animation-bug/downstream-pose.png)<br>
（蓝色对勾代表测试动画数据中有对应曲线直接控制这个节点）<br>
因为实现时的疏忽，少计算了一部分数据：当模型使用的骨骼节点实际上并不受任何动画曲线的直接控制，但这个骨骼节点的父节点链上有节点受曲线控制时，这个子节点烘焙出的模型空间动画数据，应该是上游动画曲线数据，和下游的默认姿势两部分数据的级联。前面的实现中没有处理这种情况，计算时发现没有曲线控制就直接使用默认姿势了，没有考虑父节点链上的动画曲线影响。<br>
问题找到，对应修改也就很直接了，但不改不知道，无意间又发现一个可能已经藏了好几个版本的低级错误：
```js
const m4_temp = new Mat4(); // reused globaly

for (let f = 0; f < frames; f++) {
    for (let j = 0; j < joints; j++) {
        // ...
        // calculate and store the joint transform for current frame in `mat`
        // could be Mat4.IDENTITY as a fallback placeholder
        // if no valid animation curve is found

        // apply the bindpose if not directly uploading placeholder transform
        if (mat !== Mat4.IDENTITY) Mat4.multiply(m4_temp, mat, bindposes[j]);
        // record the data to be uploaded
        uploadJointData(textureBuffer, offset, m4_temp);
        // all good... but wait, just `m4_temp`? seriously?
    }
}
```
可以看到，当前面的计算给出的结果 `mat` 为单位矩阵时，我们理论上最终应该传的就是单位矩阵，但这里上传的数据根本就是错的！<br>
这怎么能行，趁没人发现赶紧顺手改掉……<br>
改完前面说的算法问题，修掉这个低级错误，跑一下试试看吧！<br>

## Wait, what?
![](/img/in-post/skeletal-animation-bug/missing-data-error.gif)<br>
![](/img/in-post/skeletal-animation-bug/be-cool.jpg)<br>
冷静一下，仔细想想发现情况其实变得完全符合预期了：<br>
骨骼树存在时数据是完整的，我们能够精确计算出每根骨骼的变换，无论它是否直接受动画曲线控制；<br>
骨骼树不存在时数据本身就缺失一部分，我们最多只能算出最近的受曲线控制的父骨骼的变换，但并没有下游默认姿势的信息，所以图中这样的误差也就是在所难免的了。

见鬼，那修改之前的版本为什么看起来更对呢！<br>
![](/img/in-post/skeletal-animation-bug/why-is-it-working.png)<br>
回来再仔细审视上面的旧代码，我们现在考虑的情况是没有骨骼树，又不直接受曲线控制的骨骼的情况，那么在上面的旧代码中，正好对应 `mat` 是单位矩阵，我们之前修掉的低级错误的情况，意味着最终上传的数据是 `m4_temp`，而这个全局复用的临时变量里此时存着的……是同一帧内上一根骨骼的变换数据！但等一下，蘑菇模型的上一根骨骼就是它的父骨骼，而它也直接受动画曲线的影响：
```js
// The skeleton resource that the mushroom skinning model uses
{
    "_joints": [
        "RootNode/Bone_Hip/Bone_upbody/Bone_spine/Bone_head01/Bone_head02/Bone_head03",
        // The previous joint, under direct control of a animation curve
        "RootNode/Bone_Hip/Bone_upbody/Bone_spine/Bone_head01/Bone_head02",
        // The problematic joint
        "RootNode/Bone_Hip/Bone_upbody/Bone_spine/Bone_head01/Bone_head02/Bone_ci06", 
        // ...
    ],
    // bindposes, etc. ...
}
```
也就是说我们的新旧算法两面上传的数据非常接近，设当前骨骼的序号为 $m$，它的父骨骼链的序号为 $[0, m-1]$ 区间，$\textbf{M}$ 代表骨骼的本地变换，$\textbf{B}$ 代表骨骼的绑定姿势逆变换(Inverse Bindpose Matrix)，假设直属父骨骼 $m-1$ 就受动画曲线控制，那么：

旧算法最终上传的数据是上一根骨骼的变换：
{% raw %}
$$\textbf{B}_{m-1} * \prod_{j=0}^{m-1}\textbf{M}_j$$
{% endraw %}

新算法最终上传的是缺失下游默认姿势的当前骨骼变换：
{% raw %}
$$\textbf{B}_{m} * \prod_{j=0}^{m-1}\textbf{M}_j$$
{% endraw %}

可以看到两边唯一的区别是，旧算法用的是上一根骨骼的 bindpose 而新算法用的还是当前骨骼！

## Ooooouuu... Right!
![](/img/in-post/skeletal-animation-bug/strange-knowledge-increased.jpg)<br>
好像有被启发到，这似乎是一个近似抵消误差的好方法！于是受这个低级错误 bug 的启发，我在新算法框架内引入了专门针对数据缺失情况下，通过修改 bindpose 索引来近似抵消误差的处理：
```js
// the default behavior, just use the bindpose for current joint directly
let bindposeIdx = j;
/**
 * It is regularly observed that developers may choose to delete the whole
 * skeleton node tree for skinning models that only use baked animations,
 * to reduce prefab file size.
 *
 * This becomes troublesome in some cases during baking though, e.g. when a
 * skeleton joint node is not directly controlled by any animation curve,
 * but its parent nodes are. Due to lack of proper downstream default pose,
 * the joint transform can not be calculated accurately.
 *
 * We address this issue by employing some pragmatic approximation.
 * Specifically, by multiplying the bindpose of the joint corresponding to
 * the nearest curve, instead of the actual target joint. This effectively
 * merges the skinning influence of the 'incomplete' joint into its nearest
 * parent with accurate transform data.
 * It gives more visually-plausible results compared to the naive approach
 * for most cases we've covered.
 */
// this is recorded as the nearest curve path if no downstream pose is present
if (correctionPath !== undefined) {
    // find the matching joint
    bindposeIdx = skeleton.joints.findIndex((path) => path === correctionPath);
    // just use the previous joint if the exact path is not found
    if (bindposeIdx < 0) bindposeIdx = j - 1;
}
```
![](/img/in-post/skeletal-animation-bug/mission-accomplished.gif)<br>
如在注释中提到的，稍进一步想一下就可以发现，这么做本质上是把因为缺失数据而无法精确计算变换的骨骼的蒙皮权重全都合并进离它最近的数据正确的父骨骼中，虽然一定会引入偏差，但对我们的情况来说（预烘焙 + 极限包体优化）已经是一个合理的近似。<br>
到此成功将 bug 内化成为一个小亮点 feature，可以交差了（逃
