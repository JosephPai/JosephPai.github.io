---
layout:     post
title:      "机器学习初学者:手把手教你做数字图像识别"
subtitle:   "Kaggle——Digit Recognizer"
date:       2017-12-27 17:00:00
author:     "Bai"
header-img: "img/post-bg-alitrip.jpg"
header-mask: 0.3
catalog: true
comments: true
tags:
    - 杂谈
    - 台北
    - 随想
---

项目代码Github传送门：https://github.com/JosephPai/KaggleSolution/tree/master/DigitRec
 英文版本Notebook：https://www.kaggle.com/archaeocharlie/a-beginner-s-approach-to-classification
 
 数据集来源：https://www.kaggle.com/c/digit-recognizer/data
 该问题来源于Kaggle，没有接触过Kaggle的同学可以先下载数据集，然后跟随本教程进行实战演练，之后可以继续挖掘Kaggle的更具有挑战性的问题。
 本人Github和博客也会不定期同步更新机器学习相关问题的Solution

## 简介

**首先说明，这不是数字图像分类问题的最好方法！ 这个教程主要作用是为了指引那些从来没有过实战经验的机器学习新手。 我自己作为一个机器学习的初学者，我觉得这应该会很有帮助的。 任何改进建议也都通通接受！**
 
```
import pandas as pd
import matplotlib.pyplot as plt, matplotlib.image as mpimg
from sklearn.model_selection import train_test_split
from sklearn import svm
%matplotlib inline
```

## 载入数据

 - 我们使用 pandas 的 read_csv 函数将训练数据 train.csv 读入 这个包的 dataframe。
 - 然后我们将训练图像和相应的label分割开，为我们的监督学习算法做准备。
 - 我们使用train_test_split 方法将数据集分为两部分，一部分作为训练集，一部分作为测试样例用来验证我们的训练模型的表现效果。

```
labeled_images = pd.read_csv('../input/train.csv')
images = labeled_images.iloc[0:5000,1:]
labels = labeled_images.iloc[0:5000,:1]
train_images, test_images,train_labels, test_labels = train_test_split(images, labels, train_size=0.8, random_state=0)
```

## 查看图像
 - 由于图像数据现在是一维的，所以我们将其载入到一个numpy array，然后进行reshape操作，将其转换成二维 (28x28 pixels)
 - 然后我们可以用matplotlib 绘制图像来看一下了。
 
**你可以自行设定 i 的值来查看你想看的图像**

```
i=1
img=train_images.iloc[i].as_matrix()
img=img.reshape((28,28))
plt.imshow(img,cmap='gray')
plt.title(train_labels.iloc[i,0])
```

 ![这里写图片描述](https://www.kaggle.io/svf/470167/97ff4dd59348564952d32f070bf251be/__results___files/__results___5_1.png)
 
## 检查像素值

**请注意，这些图像实际上并不是黑白的（0,1）， 他们是灰度值（0-255）。**
 
 用直方图来看一下像素值范围

 

```
plt.hist(train_images.iloc[i])
```
![这里写图片描述](https://www.kaggle.io/svf/470167/97ff4dd59348564952d32f070bf251be/__results___files/__results___7_1.png)
 
## 训练模型
 - 首先，我们使用 sklearn 包提供的 svm 模型来建立一个分类器 classifier。
 - 然后，我们把我们刚刚准备好的训练集部分的训练数据和label都放到 classifier 的 fit 方法中，它会帮助我们进行训练拟合。
 - 最后，把我们刚刚分割出来的测试样例和label放到 classifier 的 score 方法中，它会帮我们评估出我们刚刚训练的模型在测试集的表现情况，输出结果是一个 0-1 范围的 float 型数字，代表精准度。

**尝试使用svm.SVC查看结果如何。**

```
clf = svm.SVC()
clf.fit(train_images, train_labels.values.ravel())
clf.score(test_images,test_labels)
```
 输出结果
 

> 0.10000000000000001

## 结果如何？

 你应该得到0.10左右，或10％的准确度。显然是个糟糕的结果，和我们随机胡乱猜测的准确度一样了。然而有很多方法可以改善这一点，比如换一个分类器，调整参数，但是这里是一个简单的开始。 让我们简化我们的图像，使它们真正的黑白图像再看看吧。

 - 为了简化数据，我们把每个数据点都改写成，要么为0要么为1的形式。
 - 然后我们可以再把图像绘制出来看一下！
 

```
test_images[test_images>0]=1
train_images[train_images>0]=1

img=train_images.iloc[i].as_matrix().reshape((28,28))
plt.imshow(img,cmap='binary')
plt.title(train_labels.iloc[i])
```
![这里写图片描述](https://www.kaggle.io/svf/470167/97ff4dd59348564952d32f070bf251be/__results___files/__results___11_2.png)

```
plt.hist(train_images.iloc[i])
```
![这里写图片描述](https://www.kaggle.io/svf/470167/97ff4dd59348564952d32f070bf251be/__results___files/__results___12_1.png)

## 再次训练分类模型

 **我们按照和之前相同的步骤，但现在我们的训练和测试集是黑白01，而不是0-255灰度值。 得到结果分数还不是很高，但这已经是一个不小的进步了！**

 
```
clf = svm.SVC()
clf.fit(train_images, train_labels.values.ravel())
clf.score(test_images,test_labels)
```
 结果 

> 0.88700000000000001

## 预测测试集

 刚刚我们是把原始数据集分割出来20%作为测试样例，得到了上面的精确度，接下来我们要用我们训练好的模型对真正的测试集进行预测了，预测结果将写入到csv文件当中。
 

```
test_data=pd.read_csv('../input/test.csv')
test_data[test_data>0]=1
results=clf.predict(test_data[0:5000])

df = pd.DataFrame(results)
df.index.name='ImageId'
df.index+=1
df.columns=['Label']
df.to_csv('results.csv', header=True)
```

## 提升优化
 
 不知道你有没有注意到，上面的代码中，我们的训练集和测试集都是取的前5000个，事实上，train.csv中提供的数据远不止5000条，这样写只是为了让模型跑的快一点方便学习，在真正的训练中，更大的数据集可以获得更好的训练效果，你可以自己尝试，在自己的电脑运算承受能力和数据集大小对应的精确度之间取得一个平衡。
 初次之外，尝试其他的分类器也可能对模型精确度有提升，比如笔者尝试使用sklearn中提供的kNN分类器进行训练，我们知道kNN算法是很简单粗暴的算法，需要很大的运算量，但是它仍然取得了很好的训练效果。
 讲这些是为了说明，我们仍然有很多渠道可以提升模型精确度，比如在Kaggle上有很多人用CNN模型取得了100%的好成绩，这些都是我们努力的目标。