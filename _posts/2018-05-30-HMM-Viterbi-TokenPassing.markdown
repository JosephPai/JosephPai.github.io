---
layout:     post
title:      "从语音识别的HMM模型的解码到Viterbi算法的Token Passing实现"
subtitle:   "A Simple Conceptual Model"
date:       2018-05-30 12:20:00
author:     "Bai"
header-img: "img/post-bg-alitrip.jpg"
header-mask: 0.3
catalog:    true
comments: true
tags:
    - 语音识别
    - 算法
---

## 1 从语音识别说起
语音识别是什么，通俗来说，就是输入音频，输出识别文字结果。基本方程如下
![这里写图片描述](https://img-blog.csdn.net/20180531120643920?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0pvc2VwaFBhaQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![这里写图片描述](https://img-blog.csdn.net/20180531120707285?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0pvc2VwaFBhaQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)： 识别结果
 	W：任一单词（以孤立词举例说明）
 	O：输入的语音序列（Observation Sequence）
 	
上述方程的变换应用了Bayes Rule.
等式右边是两项乘积，P(W)来自语言模型（Language Model， LM）， 常用的模型有 N-gram。 P(O | W)来自于声学模型（Acoustic Model， AM），传统的语音识别系统普遍采用的是基于GMM-HMM的声学模型，其中GMM用于对语音声学特征的分布进行建模，HMM则用于对语音信号的时序性进行建模。具体介绍网上有很多通俗易懂的文章，这里暂不赘述。本文重点讲解声学模型中的解码问题，暂不涉及语言模型。


## 2 计算P(O | W) —— 解码
P(O | W)可以继续分解如下
 	P(O | W)  =  P(A, O | W) ![这里写图片描述](https://img-blog.csdn.net/20180531121459223?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0pvc2VwaFBhaQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/10)
 	其中A表示状态序列，O表示观测序列，a表示转移概率，b表示观测概率
 	![这里写图片描述](https://img-blog.csdn.net/20180531121531209?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0pvc2VwaFBhaQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

如图，上方是状态序列（state sequence）（HMM），下方是观测序列（observation sequence）。我们看到，states之间存在不同的转移方式，每个state针对observation也有不同generate方式。可以想象，通过组合，我们可以得到一个巨大的状态网络。而我们语音识别的任务，通俗来说就是从这个巨大的状态网络中搜索到最佳路径（partial path），也就是最大概率的路径，对应上面公式的argmax。这个搜索匹配的过程在语音识别中叫做解码（decode）。在众多路径中找到最佳路径，一种暴力方法是穷举，但是计算量大到不现实。目前应用的主流方法是Viterbi算法。


## 3 Viterbi算法

Viterbi算法的基础概括成下面三点：
      

 1. 如果概率最大的路径P（或者说是最短路径）经过某点a，那么这条路径上从起始点s到a的这一段子路径一定是s到a之间的最短路径。否则用s到a的最短路径来替换上述路径，便构成了一条比P更短的路径，矛盾。
 2. 从S到E的路径必定经过第i时刻的某个状态，假定第i时刻有k个状态，那么如果记录了从S到第i个状态的所有k个节点的最短路径，最终的最短路径必经过其中的一条。这样，在任何时刻，只需要考虑非常有限条最短路径即可。
 3. 结合上述两点，假定当我们从状态i进入状态i+1时，从S到状态i上各个节点的最短路径已经找到，并且记录在这些节点上，那么在计算从起点S到前一个状态i所有的k个结点的最短路径，以及从这k个节点到Xi+1，j的距离即可。
 
 ![这里写图片描述](https://img-blog.csdn.net/20180531121832296?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0pvc2VwaFBhaQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

上图是一个简单的四个状态HMM应用Viterbi算法的例子。
Viterbi算法的思想是典型的动态规划思想。动态规划相对穷举已经大大减少了计算量，然而面对巨大的网络，我们意识到还是有很多不必要的计算。有些路径的计算过程中概率已经很小，完全偏离了我们要的最佳路径，继续计算这样的路径显然是没有必要的。这个时候就需要剪枝（pruning），剪枝的Viterbi算法其中一个实现叫做Token Passing，著名的语音识别工具箱HTK的解码部分应用的就是这一概念模型。

## 4 Token Passing Approach
### 4.1 概念模型
假设每一个HMM的state可以保存一个或多个Token。Token是一个概念上的对象object，它可以在state之间进行传递，一般都是按照箭头指向的方向，所以也叫前传（propagate）。每一个Token携带着它所经过路径的打分score，这个分值一般是log量级的概率和（因为我们要找的是最大概率路径嘛，也就是最高分的路径。）Token的传递是以观测序列的generate为节拍进行。
你可以想象每一条路径都是一条贪吃蛇，token就是蛇的头部的那一节，身体部分就是他所经过的state路径。算法过程大致如下：

```
初始化（t=0）：
 	初始state（入口处）的Token的s=0
 	其他state的Token的s=-inf
执行过程（t>0）：
	复制若干数目Token，并将其传递至所有与该state连接的其他state中，并且对其值做如下操作：	
	在每个state中，比较所有token，留下分值最高的token，抛弃其他所有token（Viterbi剪枝过程）
终态（t=T）：
 	比较所有终态（final state）的Token，保留其中分数最高的token
```

该token对应的就是最佳路径的概率。

![这里写图片描述](https://img-blog.csdn.net/20180531122053230?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0pvc2VwaFBhaQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

上图是一个Token Passing示意图，过段时间我计划做一个flash或者录一个小视频来更直观的演示这个过程。



### 4.2 孤立词识别（Isolated Word Recognition）
将Token Passing的思想应用到语音识别的解码过程中，我们首先从孤立词识别引入。一个英文单词音频一般分为三个音素，而一个音素又可以分为若干个状态，这些层级展开一个网络。他们之间的跳转符合隐马尔假设，所以可以应用Viterbi算法进行搜索解码。
单词级的HMM和音素级的HMM都可以直接套用上文的Token Passing模型，实现如下。
![这里写图片描述](https://img-blog.csdn.net/20180531122213113?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0pvc2VwaFBhaQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![这里写图片描述](https://img-blog.csdn.net/20180531122221899?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0pvc2VwaFBhaQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

通过这一过程，我们对每一个单词都建立了一个HMM model，为后面的连续词识别奠定了基础。

### 4.3 连续识别（Connected Word Recognition 或 Continuous Speech Recognition）
连续识别的主要问题在于：

> ① 音频中词与词之间的分界线（boundary）不明确
 								② 音频序列中总的单词数目不能确定

应用Token Passing模型，我们可以以上文的孤立词识别为基础，将不同的孤立词模型组合成一个网络，构成复合的语句级别的HMM模型（sentence model），并进行下图所示的抽象。在interface右边是上文提到的孤立词级别的模式匹配模型，在interface右边是语句级别的连续识别匹配，这时候我们就可以不用关注单词级别的具体实现。

![这里写图片描述](https://img-blog.csdn.net/20180531122411995?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0pvc2VwaFBhaQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

具体识别过程，即在上文提到的语句级复合HMM中继续应用Token Passing模型（因为我们的目标和之前一样，还是寻找最佳路径）。

![这里写图片描述](https://img-blog.csdn.net/20180531122440576?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0pvc2VwaFBhaQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

此外，在连续词识别中，我们还需要对基础的Token Passing进行一些扩展。为了能够准确的记录我们识别出的句子所包含的单词，我们引入一个新的概念Word Link Record(WLR). 顾名思义，这是一条记录，里面存放着记录word link的指针。Token除了携带score信息，还要携带一个path identifier来记录上一个WLR。
在C/C++中，token的数据结构大概会是：

```
struct Token{
 	double score;
 	struct WRL *path_id;	  
 	}
```

在上面的Token Passing基础算法中，我们新增如下过程：

```
在时间 t ，对每一个完成单词级识别的token（从小网络传递到大网络） do:
 	创建一个新的WLR
 	WLR包含 <token内容（最大概率）， 时间t， 记录路径的path id, 刚刚完成识别的单词model id>
 	将token中的path id指向刚刚创建的这条新的WLR
end
```
![这里写图片描述](https://img-blog.csdn.net/20180531122613932?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0pvc2VwaFBhaQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

如图所示，WLR构成的链表结构，记录了所有潜在的boundary信息。例如，t-3时刻很可能是单词two的结尾。
 	当到达T时刻整个句子识别结束后，我们再通过比较，得到得分最高的token，然后通过该token的path id以及WLR的链表结构可以轻松的回溯得出识别结果。

### 5 总结
 以上就是应用Token Passing模型实现Viterbi算法的具体过程，该模型通俗易懂，有效的解决了连续词识别的问题。泛化能力强，实际上One Pass、Level Building等经典算法都可以看做Token Passing的特殊情况。且易于扩展，譬如我们可以通过保留多个token来评比多条路径（N-best）以提升识别效果。
 	本文为了做到尽量通俗，有诸多不严谨之处，欢迎批评指正。



Reference：
[1] Young S J, Russell N H, Thornton J H S. Token passing: a simple conceptual model for connected speech recognition systems[M]. Cambridge, UK: Cambridge University Engineering Department, 1989.
[2] Young S, Evermann G, Gales M, et al. The HTK book[J]. Cambridge university engineering department, 2002, 3: 175.
[3] University of Cambridge Engineering Part IIB & EIST Part II Module 4F11: Speech Processing Lecture 11: Continuous Speech Recognition
[4] SGN-24006 Analysis of Audio, Speech and Music Signals lec09
[5] 吴军. 《数学之美》人民邮电出版社2012-5 ISBN-9787115282828