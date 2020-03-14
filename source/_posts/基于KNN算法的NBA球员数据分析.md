---
title: 基于KNN算法的NBA球员数据分析
typora-copy-images-to: ../../source/assets
typora-root-url: ../../source
date: 2019-06-1 16:21:35
updated: 2019-06-1 16:21:35
tags:
 - Python
 - knn
categories:
 - 大数据

---

这篇文章是我在学校大数据课的一篇论文，有兴趣的可以详细了解，由于是最初最初学习，使用knn算法进行一个简单的分析。

<!-- more -->

# 一、研究意义

NBA的篮球哲学，随着金州勇士队连续闯入五年闯入总决赛，正在发生巨大的转变，三分成为了球队获胜的必要因素，联盟规则偏向进攻球员的转变，而本世纪初期铁血的防守理论已经无法适应新的环境。于此同时带来的就是球员位置的飘忽不定，每个球队都有小个阵容，球员的篮球习惯不同，在场上每套阵容安排时的司职位置也就不同。我使用球员的场均得分，篮板，助攻，抢断，盖帽来评估球员的场上位置。可以为教练和资深球迷提供了数据支持。

# 二、数据描述

Espn提供了NBA的篮球数据，供球迷和NBA分析师使用，这里我使用requests库来发送http请求，使用BeautifulSoup库解析html。数据来源网站即是[espn官网](http://www.espn.com)。

**表2-1 数据属性结构**

| 属性            | 作用       |
| --------------- | ---------- |
| 场均上场时间min | 预处理依据 |
| 场均得分pts     | 判断依据   |
| 场均篮板reb     | 判断依据   |
| 场均助攻ast     | 判断依据   |
| 场均抢断stl     | 判断依据   |
| 场均篮板blk     | 判断依据   |
| 场上位置        | 标签label  |

如表2-1所示，我爬取了2018-2019赛季常规赛30个球队每名球员的

- 场均上场时间min，
- 场均得分pts，
- 场均篮板reb，
- 场均助攻ast，
- 场均抢断stl，
- 场均篮板blk，
- 场上位置

共7个属性，536条数据。其中`min`作为预处理时排除噪声数据的依据，`场均得分pts，场均篮板reb，场均助攻ast，场均抢断stl，场均篮板blk`五项属性作为判断球员场上位置的数据依据，而球员`场上位置`作为**标签label**。通过numpy库的save方法将前六条数据和标签分别保存，如下：

```python
ori_data = transpose([min_data_list, pts_data_list, reb_data_list, ast_data_list, stl_data_list, blk_data_list])
save("ori_data.npy", ori_data)
save("label_data.npy", label_list)
```

**预处理：**由于每个球员场均上场时间过小时，其篮球数据可能不准确，因此将场均上场时间min大于8分钟的数据保留，其他数据删除，相关代码如下：

```python
# 预处理开始
    ori_dataSet = load("ori_data.npy")
    label = load("label_data.npy")
    dataSet = hstack((ori_dataSet, transpose([label])))
    filter_dataSet = zeros(shape=(1,7))
    for i in range(len(dataSet)):
        if float(dataSet[i,0]) > 8:
            filter_dataSet = vstack((filter_dataSet,dataSet[i]))
    labels = filter_dataSet[1:,-1]
    filter_dataSet  = filter_dataSet[1:,1:-1].astype(float)
# 预处理结束
```

**归一化：**由于每个属性的权值不同，这里使用归一化方式处理每个属性，相关代码如下：

```python
# 归一化操作开始
    maxVals = filter_dataSet.max(0)
    minVals = filter_dataSet.min(0)
    ranges = maxVals - minVals
    m = filter_dataSet.shape[0]
    normDataSet = (filter_dataSet - tile(minVals, (m,1))) / tile(ranges, (m,1))
# 归一化操作结束
```

# 三、模型描述

主要函数如下：

- `createDataSet()`：读取保存的数据，并进行预处理，归一化操作；	
  - 读取数据
  - 将data和label水平拼接成矩阵temp
  - 对temp行循环，将min满足要求的数据添加到mydata
  - mydata即为预处理后数据，按列取mydata最大最小值
  - 以最大值减最小值ranges为分母，将mydata归一化

- `classify(input, dataSet, label, k)`：使用KNN算法进行归类，input为期望归类的数据，dataSet是判断依据集合，label是对应的标签，k为每次判断时邻居个数，合理的k值根据下文提到的交叉判断得出。
  - 通过两个矩阵运算来求欧式距离
  - 对距离排序
  - 从排序后的K个数据中寻找label属性最多的数据，即为所求
  - main()：程序的主入口，输入input数据，并得出结果。
  - 创建数据
  - 执行核心算法classify（）

- `cross_validate.py` 设计好算法后用于交叉验证，计算出最优K值。
  - 对原数据2，8分类，其中20%做测试数据，80%做训练数据
  - 用20%的训练数据依次在训练数据中执行算法，并得到结果集合
  - 将结果集合和实际数据对比，求出错误个数，错误率
  - 使用不同的K值重复上两步操作，得到错误率最小的K

# 四、算法实现

因为算法较为简单，一个py跑到底

```python
##给出训练数据以及对应的类别
def createDataSet():
# 预处理开始
ori_dataSet = load("ori_data.npy")
label = load("label_data.npy")
dataSet = hstack((ori_dataSet, transpose([label])))
filter_dataSet = zeros(shape=(1,7))
for i in range(len(dataSet)):
if float(dataSet[i,0]) > 8:
filter_dataSet = vstack((filter_dataSet,dataSet[i]))
labels = filter_dataSet[1:,-1]
filter_dataSet  = filter_dataSet[1:,1:-1].astype(float)
# 预处理结束
# 归一化操作开始
maxVals = filter_dataSet.max(0)
minVals = filter_dataSet.min(0)
ranges = maxVals - minVals
m = filter_dataSet.shape[0]
normDataSet = (filter_dataSet - tile(minVals, (m,1))) / tile(ranges, (m,1))
# 归一化操作结束
return normDataSet, labels, ranges, minVals

###通过KNN进行分类
def classify(input, dataSet, label, k):
    dataSize = dataSet.shape[0]
    ####计算欧式距离
    diff = tile(input, (dataSize, 1)) - dataSet
    sqdiff = diff ** 2
    squareDist = sum(sqdiff, axis=1)  ###行向量分别相加，从而得到新的一个行向量
    dist = squareDist ** 0.5
    ##对距离进行排序
    sortedDistIndex = argsort(dist)  ##argsort()根据元素的值从大到小对元素进行排序，返回下标
    classCount = {}
    for i in range(k):
        voteLabel = label[sortedDistIndex[i]]
        ###对选取的K个样本所属的类别个数进行统计
        classCount[voteLabel] = classCount.get(voteLabel, 0) + 1
        # classCount.clear()
    ###选取出现的类别次数最多的类别
    maxCount = 0
    classes  = ''
    for key, value in classCount.items():
        if value > maxCount:
            maxCount = value
            classes = key
    return classes

def main():
    normDataSet, labels, ranges, minVals = createDataSet()
    input = array([18,8,3,0.3,2.1])
    normInput = (input - minVals) / ranges #对输入的菜品进行归一化处理
    K = 17
print("测试数据为:",input,"分类结果为：",classify(normInput, normDataSet, labels, K))

cross_validata.py:
from numpy import *
import main
import matplotlib.pyplot as plt

# 预处理开始
ori_dataSet = load("ori_data.npy")
label = load("label_data.npy")
my_label = label
# my_label 相当于取每个位置的最后一个位置来验证
# PG,SG => G
# SF,PF => F
# C     => C
# 因为现在篮球位置飘忽摇摆，所以能验证出是前场球员还是后场球员即是好的算法
for i in range(label.shape[0]):
    my_label[i] = label[i][-1]

dataSet = hstack((ori_dataSet, transpose([my_label])))
filter_dataSet = zeros(shape=(1,7))
for i in range(len(dataSet)):
    if float(dataSet[i,0]) > 8:
        filter_dataSet = vstack((filter_dataSet,dataSet[i]))
filter_dataSet = filter_dataSet[1:]
# 预处理结束
# 选取训练集和验证集
vaildate_range = 0.2 #选取多少数据进行验证
trian_set = filter_dataSet[0:int(filter_dataSet.shape[0] * vaildate_range), :]
vaildate_set = filter_dataSet[int(filter_dataSet.shape[0] * vaildate_range):, :]
trian_only_dataSet = trian_set[:, 1:-1].astype(float)
trian_labels = trian_set[:, -1]
vaildate_only_dataSet = vaildate_set[:, 1:-1].astype(float)
vaildate_labels = vaildate_set[:, -1]

# 进行验证
test_count = trian_set.shape[0]
k_err_di = {}
k_err = []
for k in range(1,30):
    err_time = 0
    for i in range(test_count):
        temp_test_label = main.classify(trian_only_dataSet[i], vaildate_only_dataSet, vaildate_labels, k)
        if temp_test_label != trian_labels[i]:
            err_time += 1
    k_err.append(err_time/test_count)
    k_err_di[k] = err_time/test_count
print(k_err_di)
xs = arange(len(k_err)) + 1
plt.plot(xs,k_err)
plt.xlabel('K')
plt.ylabel('Error')
plt.title('Error vs K')
plt.show()
```

# 五、运行结果及意义说明

![图5-1交叉验证结果](图5-1交叉验证结果.png)

交叉验证结果如图5-1, 因为现代篮球的位置区分率不高，所以按照5个传统位置来分，错误率较高；因此使用F前场，C中锋，G后场，三种球员位置来进行交叉验证。在`K=(1:30)`区间内进行验证，发现`K = 17`时错误率最低，因此选K = 17。

![图5-2核心代码验证](图5-2核心代码验证.png)

核心代码验证结果如图5-2，所示，输入分别为场均得分，篮板，助攻，抢断，盖帽，结果符合现实篮球规律。

# 六、总结

在实现这次基于KNN算法的应用过程中，自己亲身体验到了一种分类算法的实现过程，同时也深刻了解了该算法的经典和不足之处，在进行交叉验证时，发现错误率均较高，根据核心分析原因，是因为label不同的一些数据分布在一个较近的区域，因此不容易区分，同时这也是KNN算法的不足之处：对于区域内模糊的数据难以区分。这也激起了我学习其他分类算法的兴趣，对比各种算法的优劣，在合适的应用环境中选择合适的算法。

经过2个学时的学习，我对于大数据的概念已经有了一定的认识，也已经体验了一次数据分析的过程，相信在以后的学习和研究过程中，能够不断丰富相关知识，同时也不断应用所学知识解决问题。