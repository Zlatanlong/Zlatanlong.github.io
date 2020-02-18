---
title: 基于BP神经网络的NBA球员薪资评估
typora-copy-images-to: ../../source/assets
typora-root-url: ../../source
date: 2019-07-1 16:21:35
updated: 2019-07-1 16:21:35
tags:
 - python
 - bp
categories:
 - 大数据
---

这篇文章是我在学校大数据课的一篇论文，有兴趣的可以详细了解。

# 一、研究意义

在当今NBA的时代背景下，左右球队能在一个赛季走多远的不止是教练的精妙布置和球员在场上的努力发挥，球队在休赛期间的交易十分重要，比如2017年杜兰特加盟勇士队，和联盟第一球星詹姆斯的几次转回，还有猛龙与马刺的交易等，这些重磅交易不仅引起了巨大的关注，同时也为近几年来NBA总冠军的归属和联盟的格局埋下了伏笔。不止是超级巨星，包括明星球员和重要角色球员的转回，都可能为球队的比赛以及运营带来巨大的影响；而交易最重要得一个环节就是薪水的评估，球队和球员双方可能需要很多人员对球员的场上数据，身体条件进行评估，同时基于市场考虑谈判出一个令双方都满意的结果。因此我针对这一现象，基于老师提供的BP神经网络算法和相关实现代码，提出一套评估球员薪水的解决方案，这套方案还有很大不足和需要进步之处，希望能够在日后的学习中不断弥补。

<!-- more -->

# 二、数据描述

国内的虎扑社区提供了NBA的篮球数据，供球迷使用，其中包括了球员的合同和当季薪资，这里我使用requests库来发送http请求，使用BeautifulSoup库解析html。数据来源网站即是[虎扑网站](https://nba.hupu.com/players)。

**表2-1 数据属性结构**

| **属性**       | **作用**   | **属性**       | **作用**   |
| -------------- | ---------- | -------------- | ---------- |
| 场均得分       | 神经元输入 | 场均三分命中率 | 神经元输入 |
| 场均篮板       | 神经元输入 | 场均罚球命中率 | 神经元输入 |
| 场均助攻       | 神经元输入 | 场均抢断       | 神经元输入 |
| 场均投篮命中率 | 神经元输入 | 场均盖帽       | 神经元输入 |
| 薪资水平       | 神经元输出 |                |            |

 

我爬取了2018-2019赛季常规赛30个球队每名球员的场均得分，场均篮板，场均助攻，场均抢断，场均盖帽，场均投篮命中率，场均三分命中率，场均罚球命中率，当季共9个属性，497条数据。在预处理过程中将当季薪资为0或者没有信息的数据删除，因为这些球员可能别没有签约，其他8项属性作为输入神经元，并将球员按照薪资水平，每500W美元为一个档次，构建出可以与输出矩阵做算术运算格式匹配的响应的Y矩阵。

如表2-1所示，通过numpy库的save方法将数据保存，如下：

```python
save("date_players.npy", date_players)
```

## 预处理：

如果在爬取数据过程中球员薪水没有获取到，说明该球员可能没有签约，因此将该数据删除，代码如下：

```python
# 预处理

    data = pd.DataFrame(date_players)

    data_salary = data.iloc[:,8]

    data_salary[data_salary == 0] = np.NAN

    data_new = np.hstack((date_players[:,:8], np.array([data_salary]).T))

    data2 = pd.DataFrame(data_new).dropna(thresh=9)

    date_players_new = np.array(data2)

    xdata_ori = date_players_new[:,:8]

    salary_ori = date_players_new[:,8]//500 #以500W为间隔
    
# 预处理结束
```

## 归一化：

在我分析完老师提供的bp神经网络代码实现后，发现其中使用的S函数等函数，值域在[0,1]之间，因此神经元的输入必须进行归一化处理，否则将可能报错；同时归一化处理也将神经元输入的各种数据进行平权处理。

```python
# 归一化

    maxVals = xdata_ori.max(0)

    minVals = xdata_ori.min(0)

    ranges = maxVals - minVals

    m = xdata_ori.shape[0]

    normDataSet = (xdata_ori - np.tile(minVals, (m, 1))) / np.tile(ranges, (m, 1))

    trainX = normDataSet.T
# 归一化结束
```

## 处理Y矩阵：

因为爬取到的只是薪资数据，为了跑通BP神经网络算法，必须将数据进行处理，处理方法为：根据输出长度生成一列0矩阵，根据薪资水平的档次将该矩阵的某一个数据设置为1，在bp神经网络进行迭代的过程中，每一次向前传递都会得出一组输出`tempY`，其中`tempY`中每个数据及为是这一类的概率有多少，使用处理过的Y矩阵与`tempY`进行数学运算即可向后传递，不断迭代即可训练出合适的参数。

# 三、模型描述

## 典型的神经元模型：

![image-20200217163219161](/assets/image-20200217163219161.png)

## 神经网络结构（单一输出）：

![image-20200217163239394](/assets/image-20200217163239394.png)

## BP神经网络基本原理：

分为两个过程：

工作信号正向传递子过程，输入信息从输入层经隐层逐层、正向传递，直至得到各计算单元的输出。

误差信号反向传递子过程，输出层误差从输出层开始，逐层、反向传播，可间接计算隐层各单元的误差，并用此误差修正前层的权值。

Bp算法主要函数如下：

- `iniPara()`：根据给定的神经网络结构根据正态分布随机初始化一套参数；

- `forwardModel(X, parameters)`：一个完整的向前传播过程，根据输入X和神经网络中间参数parameters，返回输出和中间每一层的参数。

- ` backwardModel(AL, Y, caches)`：完整的反向传播过程，根据输出和真实的Y矩阵进行计算，同时计算每一隐含层梯度，返回梯度矩阵diffs

- `updateParameters(parameters, diffs, learningRate)`：根据之前的参数，梯度矩阵diffs和学习率，进行参数更新，返回新的参数。

- `predict(X, y, parameters, ifPrint)`：预测函数，根据训练好的参数和新的输入，预测输出，并且可以和实际情况Y做对比，根据ifPrint是否打印输出算法准确率。

# 四、算法实现:

经过精读老师提供的bp神经网络实现代码，发现其中核心函数中可以修改地方只有激活函数的选取可以更改，而根据bp神经网络原理可知，在神经网络的中间层更加建议使用`relu函数`。

如果`bp神经网络`的输出不是单输出，`predict函数`需要进行修改(位于代码最后)，我对该函数进行修改后使其能够应对多种输出情况，代码如下：

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
Created on Sat Oct 14 09:54:44 2017

@author: zhangshichaochina@gmail.com
"""

import numpy as np
import matplotlib.pyplot as plt
import h5py
import scipy
import time
from PIL import Image
from scipy import ndimage


def sigmoid(z):
    """
    使用numpy实现sigmoid函数
    
    参数：
    Z numpy array
    输出：
    A 激活值（维数和Z完全相同）
    """
    return 1/(1 + np.exp(-z))

def relu(z):
    """
    线性修正函数relu

    参数：
    z numpy array
    输出：
    A 激活值（维数和Z完全相同）

    """
    return np.array(z>0)*z

def sigmoidBackward(dA, cacheA):
    """
    sigmoid的反向传播
    
    参数：
    dA 同层激活值
    cacheA 同层线性输出
    输出：
    dZ 梯度
    
    """
    s = sigmoid(cacheA)
    diff = s*(1 - s)
    dZ = dA * diff
    return dZ

def reluBackward(dA, cacheA):
    """
    relu的反向传播
    
    参数：
    dA 同层激活值
    cacheA 同层线性输出
    输出：
    dZ 梯度
    
    """
    Z = cacheA
    dZ = np.array(dA, copy=True) 
    dZ[Z <= 0] = 0
    return dZ

def iniPara(laydims):
    """
    随机初始化网络参数
    
    参数：
    laydims 一个python list
    输出：
    parameters 随机初始化的参数字典（”W1“，”b1“，”W2“，”b2“, ...）
    """
    np.random.seed(1)
    parameters = {}
    for i in range(1, len(laydims)):
        parameters['W'+str(i)] = np.random.randn(laydims[i], laydims[i-1])/ np.sqrt(laydims[i-1])
        parameters['b'+str(i)] = np.zeros((laydims[i], 1))
    return parameters


def loadData(dataDir):
    """
    导入数据
    
    参数：
    dataDir 数据集路径
    输出：
    训练集，测试集以及标签
    """
    train_dataset = h5py.File(dataDir+'/train.h5', "r")
    train_set_x_orig = np.array(train_dataset["train_set_x"][:]) # your train set features
    train_set_y_orig = np.array(train_dataset["train_set_y"][:]) # your train set labels

    test_dataset = h5py.File(dataDir+'/test.h5', "r")
    test_set_x_orig = np.array(test_dataset["test_set_x"][:]) # your test set features
    test_set_y_orig = np.array(test_dataset["test_set_y"][:]) # your test set labels

    classes = np.array(test_dataset["list_classes"][:]) # the list of classes
    
    train_set_y_orig = train_set_y_orig.reshape((1, train_set_y_orig.shape[0]))
    test_set_y_orig = test_set_y_orig.reshape((1, test_set_y_orig.shape[0]))
    
    return train_set_x_orig, train_set_y_orig, test_set_x_orig, test_set_y_orig, classes


def forwardLinear(W, b, A_prev):
    """
    前向传播
    特征乘权值+b
    Z 就是乘激活函数前的东西
    """
    Z = np.dot(W, A_prev) + b
    cache = (W, A_prev, b)
    return Z, cache

def forwardLinearActivation(W, b, A_prev, activation):
    """
    带激活函数的前向传播
    A: 加上激活函数的效果
    cacheL: 最原始的参数
    cacheA: 加权后的参数
    """
    Z, cacheL = forwardLinear(W, b, A_prev)
    cacheA = Z
    if activation == 'sigmoid':
        A = sigmoid(Z)
    if activation == 'relu':
        A = relu(Z)
    cache = (cacheL, cacheA)
    return A, cache

def forwardModel(X, parameters):
    """
    完整的前向传播过程
    X: 训练集特征
    AL: 完整前项传播得到的输出
    """
    layerdim = len(parameters)//2 #有几层对应隐含层
    caches = []
    A_prev = X
    for i in range(1, layerdim):
        A_prev, cache = forwardLinearActivation(parameters['W'+str(i)], parameters['b'+str(i)], A_prev, 'relu')
        caches.append(cache)
        
    AL, cache = forwardLinearActivation(parameters['W'+str(layerdim)], parameters['b'+str(layerdim)], A_prev, 'sigmoid')
    caches.append(cache)
    
    return AL, caches

def computeCost(AL, Y):
    """
    代价函数的计算
    
    参数：
    AL 输出层的激活输出
    Y 标签值
    输出：
    cost 代价函数值
    """
    m = Y.shape[1]
    cost = (1./m) * (-np.dot(Y,np.log(AL).T) - np.dot(1-Y, np.log(1-AL).T))
    return cost

def linearBackward(dZ, cache):
    """
    线性部分的反向传播
    
    参数：
    dZ 当前层误差
    cache （W, A_prev, b）元组
    输出：
    dA_prev 上一层激活的梯度
    dW 当前层W的梯度
    db 当前层b的梯度
    """
    W, A_prev, b = cache
    m = A_prev.shape[1]
    
    dW = 1/m*np.dot(dZ, A_prev.T)
    db = 1/m*np.sum(dZ, axis = 1, keepdims=True)
    dA_prev = np.dot(W.T, dZ)
    
    return dA_prev, dW, db

def linearActivationBackward(dA, cache, activation):
    """
    非线性部分的反向传播
    
    参数：
    dA 当前层激活输出的梯度
    cache （W, A_prev, b）元组
    activation 激活函数类型
    输出：
    dA_prev 上一层激活的梯度
    dW 当前层W的梯度
    db 当前层b的梯度
    """
    cacheL, cacheA = cache
    
    if activation == 'relu':
        dZ = reluBackward(dA, cacheA)
        dA_prev, dW, db = linearBackward(dZ, cacheL)
    elif activation == 'sigmoid':
        dZ = sigmoidBackward(dA, cacheA)
        dA_prev, dW, db = linearBackward(dZ, cacheL)
    
    return dA_prev, dW, db

def backwardModel(AL, Y, caches):
    """
    完整的反向传播过程
    
    参数：
    AL 输出层结果
    Y 标签值
    caches [cacheL, cacheA]
    输出：
    diffs 梯度字典
    """
    layerdim = len(caches)
    Y = Y.reshape(AL.shape)
    L = layerdim
    
    diffs = {}
    
    dAL = - (np.divide(Y, AL) - np.divide(1 - Y, 1 - AL))
    
    currentCache = caches[L-1]
    dA_prev, dW, db =  linearActivationBackward(dAL, currentCache, 'sigmoid')
    diffs['dA' + str(L)], diffs['dW'+str(L)], diffs['db'+str(L)] = dA_prev, dW, db
    
    for l in reversed(range(L-1)):
        currentCache = caches[l]
        dA_prev, dW, db =  linearActivationBackward(dA_prev, currentCache, 'relu')
        diffs['dA' + str(l+1)], diffs['dW'+str(l+1)], diffs['db'+str(l+1)] = dA_prev, dW, db
        
    return diffs


def updateParameters(parameters, diffs, learningRate):
    """
    更新参数

    参数：
    parameters 待更新网络参数
    diffs 梯度字典
    learningRate 学习率
    输出：
    parameters 网络参数字典
    """
    layerdim = len(parameters)//2
    for i in range(1, layerdim+1):
        parameters['W'+str(i)] -= learningRate*diffs['dW'+str(i)]
        parameters['b'+str(i)] -= learningRate*diffs['db'+str(i)]
    return parameters

def finalModel(X, Y, layerdims, learningRate=0.01, numIters=5000,pringCost=False):
    """
    最终的BP神经网络模型
    
    参数：
    X 训练集特征
    Y 训练集标签
    layerdims 一个明确网络结构的python list
    learningRate 学习率
    numIters 迭代次数
    pringCost 打印标志
    输出：
    parameters 参数字典
    """
    np.random.seed(1)
    costs = []
    
    parameters = iniPara(layerdims)
    
    for i in range(0, numIters):
        AL, caches = forwardModel(X, parameters)
        cost = computeCost(AL, Y)
        
        diffs = backwardModel(AL,Y, caches)
        parameters = updateParameters(parameters,diffs, learningRate)
    
        if pringCost and i%100 == 0:
            print(cost)
            costs.append(np.sum(cost))
    return parameters

def predict(X, y, parameters, ifPrint):
    """
    在测试集上预测
    
    参数：
    X 输入特征值
    y 测试集标签
    parameters 参数字典
    输出：
    p 预测得到的标签
    """
    
    m = X.shape[1]
    n = len(parameters) // 2 
    p = np.zeros((1,m))
    probas, caches = forwardModel(X, parameters)
    truth = np.argmax(y,0)
    pre = np.argmax(probas,0)
    if ifPrint:
        print("Accuracy: "  + str(np.sum((pre == truth)/m)))

    return pre, np.sum((pre == truth)/m)
```

其他算法实现均为原bp神经网络代码实现。

# 五、运行结果及意义说明:

![图1 验证训练集准确率](/assets/图片1.png)

对训练集的准确率验证如图1，发现在学习率为0.8，迭代次数为20600次时，准确率最高。因此考虑用该参数。

![图2-1 勒布朗·詹姆斯薪水预测](/assets/图片2-1.png)

![图2-2 小乔丹薪水预测](/assets/图片2-2.png)

![图2-3 路易斯·威廉姆斯薪水预测](/assets/图片2-3.png)

图2-1，2-2，2-3分别对三名不同类型球员进行了薪水预测，完全符合市场情况，其中小乔丹刚刚以1000万美元的年薪加盟网队，可以说是名副其实的降薪了，而快船队的路易斯·威廉姆斯实际合同只有700万美元，也是物美价廉了，勒布朗虽然上赛季受伤，但是依然可以打出自己的价值，预测薪水也和实际相符。

# 六、总结

在实现这次基于bp神经网络算法的应用过程中，自己第一次亲身体验到了神经网络算法的实现过程，在对老师的实现代码进行精读时，结合课上所学知识，还算比较流畅地熟悉了代码，在自己进行测试时，也遇到了一些“坑”，其中印象较为深刻的有两个地方：**第一个地方是输入数据没有进行归一化处理，导致在第一次向前传播后经过S函数变化，数值均接近S函数的最大值1；第二个地方是输出的数据和Y矩阵的关系，在我再次理解bp神经网络原理时，了解到每个输出的值域[0,1]表示输出为该值得概率，理解之后问题更改Y矩阵格式问题便解决了。**

在这一过程中，也深刻了解了该算法的经典和不足之处，在进行训练数据验证时，发现准确率并不如人意，根据核心算法分析原因，可能是因为输入数据维度不足，输入数据和输出数据的比例不够大，因此不容易区分，因此考虑可以对此方案进行改进：增加球员年龄，身体数据和场上更多详细数据进行输入，还可以寻找球员历年的薪水（因为每年薪水受市场影响，需进行归一化）和数据以获取更多输入。但是由于球员的往年薪水数据并不容易获取，因此这是我下一步需要进行的改进。

同时在这次大作业的完成之中，我发现我所进行的工作量主要在于读懂算法，想出合理的应用环境，获取可用数据上；我的工作量与算法核心的数学内容基本无关，因此这是我未来需要进步的地方，需要不断强化数学基础，能够完成在读懂代码后根据实际项目进行算法改进的工作。

最后，我要感谢老师的在课上细心的讲解，深入到算法的每个细节，让我能对人工神经网络有了清晰地认识！