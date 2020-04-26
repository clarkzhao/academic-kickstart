---
title: "读《百面机器学习》"
date: 2020-04-24T00:13:00+08:00
toc: true

# View.
#   1 = List
#   2 = Compact
#   3 = Card
view: 2

# Optional header image (relative to `static/img/` folder).
header:
  caption: ""
  image: ""
categories: [AI]
tags: [machine learning]
---

这本书很适合查漏补缺，我将其中精华的部分摘录并加入一些补充和代码实践。


<!--more-->
{{% toc %}}


# 特征工程

## 有哪些文本表示模型？它们各自有什么的优缺点？

### 词袋模型 Bags of Words，TF-IDF Term Frequency-Inverse Document Frequency


$$
TF-IDF(t,d) = TF(t,d)\times IDF(t)
$$

其中 $TF(t,d)$ 为单词 $t$ 在文档 $d$ 中出现的频率, $IDF(t)$ 是逆文档频率，用来衡量单词 $t$ 对表达语义所起的重要性，如果一个单词在多个文章中出现，那么该词就是一个通用的词所有对其权重做出一定的惩罚，


$$
IDF(t) = \log \frac{文章总数}{包含单词 t 的文章总数}
$$


将文章进行单词级别的划分有时候并不是一种好的做法，比如英文中的 natural language processing（自然语言处理）一词，如果将 natural，language，processing 这 3个词拆分开来，所表达的含义与三个词连续出现时大相径庭。通常，可以将连续出现的n个词（n≤N）组成的词组（N-gram）也作为一个单独的特征放到向量表示中去，构成 **N-gram** 模型。





### 主题模型 Topic Model

基于词袋模型或 N-gram 模型的文本表示模型有一个明显的缺陷，就是无法识别出两个不同的词或词组具有相同的主题。因此，需要一种技术能够将具有相同主题的词或词组映射到同一维度上去，于是产生了主题模型。主题模型是一种特殊的概图模型。想象一下我们如何判定两个不同的词具有相同的主题呢？这两个词可能有更高的概率同时出现在同一篇文档中；换句话说，给定某一主题，这两个词的产生概率都是比较高的，而另一些不太相关的词汇产生的概率则是较低的。假设有K个主题，我们就把任意文章表示成一个K维的主题向量，其中向量的每一维代表一个主题，权重代表这篇文章属于这个特定主题的概率。主题模型所解决的事情，就是从文本库中发现有代表性的主题（得到每个主题上面词的分布），并且计算出每篇文章对应着哪些主题。

### 词嵌入模型 Word Embedding

词嵌入是一类将词向量化的模型的统称，核心思想是将每个词都映射成低维空间（通常K=50～300维）上的一个稠密向量（Dense Vector）。K维空间的每一维也可以看作一个隐含的主题，只不过不像主题模型中的主题那样直观。由于词嵌入将每个词映射成一个K维的向量，如果一篇文档有N个词，就可以用一个N×K维的矩阵来表示这篇文档。




## Word2Vec 是如何工作的？它和 LDA 有什么区别与联系？

![word-embedding](/img/word-embedding.png)

CBOW 的目标是根据上下文出现的词语来预测当前词的生成概率，如图所示；而 Skip-gram 是根据当前词来预测上下文中各词的生成概率。其中 w(t) 是当前所关注的词，w(t−2)、w(t−1)、w(t+1)、w(t+2) 是上下文中出现的词。这里前后滑动窗口大小均设为2。CBOW 和 Skip-gram 都可以表示成由输入层（Input）、映射层（Projection）和输出层（Output）组成的神经网络。输入层中的每个词由独热编码方式表示，即所有词均表示成一个N维向量，其中N为词汇表中单词的总数。在向量中，每个词都将与之对应的维度置为1，其余维度的值均设为0。在映射层（又称隐含层）中，K个隐含单元（Hidden Units）的取值可以由N维输入向量以及连接输入和隐含单元之间的N×K维权重矩阵计算得到。在 CBOW 中，还需要将各个输入词所计算出的隐含单元求和。同理，输出层向量的值可以通过隐含层向量（K维），以及连接隐含层和输出层之间的 K×N 维权重矩阵计算得到。输出层也是一个N维向量，每维与词汇表中的一个单词相对应。最后，对输出层向量应用 Softmax 激活函数，可以计算出每个单词的生成概率。

谈到 Word2Vec 与 LDA 的区别和联系，首先，LDA 是利用文档中单词的共现关系来对单词按主题聚类，也可以理解为对“文档-单词”矩阵进行分解，得到“文档-主题”和“主题-单词”两个概率分布。而 Word2Vec 其实是对“上下文-单词”矩阵进行学习，其中上下文由周围的几个单词组成，由此得到的词向量表示更多地融入了上下文共现的特征。也就是说，如果两个单词所对应的 Word2Vec 向量相似度较高，那么它们很可能经常在同样的上下文中出现。

需要说明的是，上述分析的是 LDA 与 Word2Vec 的不同，不应该作为主题模型和词嵌入两类方法的主要差异。主题模型通过一定的结构调整可以基于“上下文-单词”矩阵进行主题推理。同样地，词嵌入方法也可以根据“文档-单词”矩阵学习出词的隐含向量表示。主题模型和词嵌入两类方法最大的不同其实在于模型本身，**主题模型是一种基于概率图模型的生成式模型，其似然函数可以写成若干条件概率连乘的形式，其中包括需要推测的隐含变量（即主题）；而词嵌入模型一般表达为神经网络的形式，似然函数定义在网络的输出之上，需要通过学习网络的权重以得到单词的稠密向量表示。**



## 在图像分类任务中，训练数据不足会带来什么问题？如何缓解数据量不足带来的问题？

一个模型所能提供的信息一般来源于两个方面，一是训练数据中蕴含的信息；二是在模型的形成过程中（包括构造、学习、推理等），人们提供的先验信息。当训练数据不足时，说明模型从原始数据中获取的信息比较少，这种情况下要想保证模型的效果，就需要更多先验信息。先验信息可以作用在模型上，例如让模型采用特定的内在结构、条件假设或添加其他一些约束条件；先验信息也可以直接施加在数据集上，即根据特定的先验假设去调整、变换或扩展训练数据，让其展现出更多的、更有用的信息，以利于后续模型的训练和学习。

具体到图像分类任务上，**训练数据不足带来的问题主要表现在过拟合方面，即模型在训练样本上的效果可能不错，但在测试集上的泛化效果不佳。**根据上述讨论，对应的处理方法大致也可以分两类：

- **一是基于模型的方法，主要是采用降低过拟合风险的措施**，包括：

  简化模型（如将非线性模型简化为线性模型）、添加约束项以缩小假设空间（如L1/L2正则项）、集成学习、Dropout超参数等；

- **二是基于数据的方法，主要通过数据扩充（Data Augmentation），即根据一些先验知**
  **识，在保持特定信息的前提下，对原始数据进行适当变换以达到扩充数据集的效果。**

  具体到图像分类任务中，在保持图像类别不变的前提下，可以对训练集中的每幅图像进行以下变换。

  1. 一定程度内的随机旋转、平移、缩放、裁剪、填充、左右翻转等，这些变换对应着同一个目标在不同角度的观察结果。
  2. 对图像中的像素添加噪声扰动，比如椒盐噪声、高斯白噪声等。
  3. 颜色变换。例如，在图像的 RGB 颜色空间上进行主成分分析，得到 3 个主成分的特征向量 p1,p2,p3 及其对应的特征值 λ1,λ2,λ3，然后在每个像素的RGB 值上添加增量 $[p1,p2,p3]\cdot[α1λ1,α2λ2,α3λ3]^T$，其中 α1,α2,α3 是均值为 0、方差较小的高斯分布随机数。
  4. 改变图像的亮度、清晰度、对比度、锐度等

# 模型评估

## 准确率的局限性？

准确率是指分类正确的样本占总样本个数的比例，即
$$
accuracy = \frac{n_{correct}}{n_{total}}
$$
其中 $n_{correct}$ 为被正确分类的样本个数，$n_{total}$ 为总样本的个数。
准确率是分类问题中最简单也是最直观的评价指标，但存在明显的缺陷。比如，当负样本占99%时，分类器把所有样本都预测为负样本也可以获得99%的准确率。所以，当不同类别的样本比例非常不均衡时，占比大的类别往往成为影响准确率的最主要因素。为了解决这个问题，可以使用更为有效的平均准确率（每个类别下的样本准确率的算术平均）作为模型评估的指标。

**TOP-N Accuracy, accuracy@N** 正样本标签在模型预测可能性最高的前 N 个中的占比。

## 精确率与召回率的权衡？

*Hulu提供视频的模糊搜索功能，搜索排序模型返回的Top 5的精确率非常高，但在实际使用过程中，用户还是经常找不到想要的视频，特别是一些比较冷门的剧集，这可能是哪个环节出了问题呢？*

**精确率 Precision**是指分类正确的正样本个数占分类器判定为正样本的样本个数的比例。

**召回率 Recall**是指分类正确的正样本个数占真正的正样本个数的比例。

**Precision 值和 Recall 值是既矛盾又统一的两个指标，为了提高 Precision 值，分类器需要尽量在“更有把握”时才把样本预测为正样本，但此时往往会因为过于保守而漏掉很多“没有把握”的正样本，导致 Recall 值降低。**

在排序问题中，通常没有一个确定的阈值把得到的结果直接判定为正样本或负样本，而是采用 Top N 返回结果的 Precision 值和 Recall 值来衡量排序模型的性能，即认为模型返回的 Top N 的结果就是模型判定的正样本，然后计算前 N 个位置上的准确率 Precision@N 和前 N 个位置上的召回率 Recall@N。

回到问题中来，模型返回的 Precision@5 的结果非常好，也就是说排序模型 Top5 的返回值的质量是很高的。但在实际应用过程中，用户为了找一些冷门的视频，往往会寻找排在较靠后位置的结果，甚至翻页去查找目标视频。但根据题目描述，用户经常找不到想要的视频，这说明模型没有把相关的视频都找出来呈现给用户。显然，问题出在召回率上。如果相关结果有100个，即使 Precision@5 达到了 100%，Recall@5 也仅仅是 5%。在模型评估时，我们是否应该同时关注 Precision 值和 Recall 值？进一步而言，是否应该选取不同的 Top N 的结果进行观察呢？是否应该选取更高阶的评估指标来更全面地反映模型在 Precision 值和 Recall 值两方面的表现？

答案都是肯定的，为了综合评估一个排序模型的好坏，不仅要看模型在不同 Top N 下的 Precision@N 和 Recall@N，而且最绘制出模型的 P-R（Precision-Recall）曲线。这里简单介绍一下 P-R 曲线的绘制方法。P-R 曲线的横轴是召回率，纵轴是精确率。对于一个排序模型来说，其 P-R 曲线上的一个点代表着，在某一阈值下，模型将大于该阈值的结果判定为正样本，小于该阈值的结果判定为负样本，此时返回结果对应的召回率和精确率。整条 P-R 曲线是通过将阈值从高到低移动而生成的。

除此之外，**F1 score** 和 **ROC** 曲线也能综合地反映一个排序模型的性能。**F1 score 是精准率和召回率的调和平均值**，它定义为：
$$
F1 = \frac{2\times precision \times recall}{precision + recall}
$$

## 平方根误差的“意外”

*Hulu 作为一家流媒体公司，拥有众多的美剧资源，预测每部美剧的流量趋势对于广告投放、用户增长都非常重要。我们希望构建一个回归模型来预测某部美剧的流量趋势，但无论采用哪种回归模型，得到的 RMSE 指标都非常高。然而事实是，模型在 95% 的时间区间内的预测误差都小于 1%，取得了相当不错的预测结果。那么，造成 RMSE 指标居高不下的最可能的原因是什么？*
$$
RMSE = \frac{\sum_{i=1}^{n}({y_i-\hat{y}_i})^2}{n}
$$
一般情况下，RMSE 能够很好地反映回归模型预测值与真实值的偏离程度。但在实际问题中，如果存在个别偏离程度非常大的离群点（Outlier）时，即使离群点数量非常少，也会让 RMSE 指标变得很差。回到问题中来，模型在 95% 的时间区间内的预测误差都小于 1%，这说明，在大部分时间区间内，模型的预测效果都是非常优秀的。然而，RMSE 却一直很差，这很可能是由于在其他的 5% 时间区间内存在非常严重的离群点。事实上，在流量预估这个问题中，噪声点确实是很容易产生的，比如流量特别小的美剧、刚上映的美剧或者刚获奖的美剧，甚至一些相关社交媒体突发事件带来的流量，都可能会造成离群点。

针对这个问题，有什么解决方案呢？可以从三个角度来思考。第一，如果我们认定这些离群点是“噪声点”的话，就需要在数据预处理的阶段把这些噪声点过滤掉。第二，如果不认为这些离群点是“噪声点”的话，就需要进一步提高模型的预测能力，将离群点产生的机制建模进去（这是一个宏大的话题，这里就不展开讨论了）。第三，可以找一个更合适的指标来评估该模型。关于评估指标，其实是存在比RMSE的鲁棒性更好的指标，比如平均绝对百分比误差（Mean Absolute Percent Error MAPE），它定义为:
$$
MAPE = \sum_{i}^{N}|\frac{y_i-\hat{y_i}}{y_i}|\times\frac{100}{n}
$$
相比 RMSE，MAPE 相当于把每个点的误差进行了归一化，降低了个别离群点带来的绝对误差的影响。



## 什么是 ROC 曲线？

**ROC** 曲线是 Receiver Operating Characteristic Curve 的简称，中文名为“受试者工作特征曲线”。ROC 曲线源于军事领域，而后在医学领域应用甚广，“受试者工作特征曲线”这一名称也正是来自于医学领域。ROC 曲线的**横坐标为假阳性率**（False Positive Rate，FPR）；**纵坐标为真阳性率**（True Positive Rate，TPR）。FPR 和 TPR 的计算方法分别为：
$$
FPR = \frac{FP}{N} 
$$

$$
TPR = \frac{TP}{P}
$$

其中，P 是真实正样本数量，N 是真实负样本数量，TP 是 P 个正样本中被分类器预测为正样本的个数，FP 是 N 个负样本中被分类器预测为正样本的个数。

## 如何绘制 ROC 曲线？

事实上，ROC 曲线是通过不断移动分类器的“截断点”来生成曲线上的一组关键点的，通过下面的例子进一步来解释“截断点”的概念。
在二值分类问题中，模型的输出一般都是预测样本为正例的概率。样本按照预测概率从高到低排序。在输出最终的正例、负例之前，我们需要指定一个阈值，预测概率大于该阈值的样本会被判为正例，小于该阈值的样本则会被判为负例。

通过动态地调整截断点，从最高的得分开始（实际上是从正无穷开始，对应着 ROC 曲线的零点），逐渐调整到最低得分，每一个截断点都会对应一个 FPR和 TPR，在 ROC 图上绘制出每个截断点对应的位置，再连接所有点就得到最终的 ROC 曲线。

下图为一个完美的 ROC 曲线，不管预测阈值如何变化，模型完美的预测出所有正负样本，此时在该曲线上只有一个点（FPR=0，TPR=1），此时 AUC 为 1。

![perfect_roc](/img/perfect_roc.png)

## 如何计算 AUC ？

顾名思义，AUC （Area Under Curve）指的是 ROC 曲线下的面积大小，该值能够量化地反映基于 ROC 曲线衡量出的模型性能。计算 AUC 值只需要沿着 ROC 横轴做积分就可以了。由于 ROC 曲线一般都处于 y=x 这条直线的上方（如果不是的话，只要把模型预测的概率反转成 1−p 就可以得到一个更好的分类器），所以 AUC 的取值一般在 0.5～1 之间。AUC 越大，说明分类器越可能把真正的正样本排在前面，分类性能越好。

## ROC 曲线相比 P-R 曲线有什么特点？

相比 P-R 曲线，ROC 曲线有一个特点，当正负样本的分布发生变化时，ROC 曲线的形状能够基本保持不变，而 P-R 曲线的形状一般会发生较剧烈的变化。

在很多实际问题中，正负样本数量往往很不均衡。比如，计算广告领域经常涉及转化率模型，正样本的数量往往是负样本数量的 1/1000 甚至 1/10000。若选择不同的测试集，P-R 曲线的变化就会非常大，而 ROC 曲线则能够更加稳定地反映模型本身的好坏。所以，ROC 曲线的适用场景更多，被广泛用于排序、推荐、广告等领域。但需要注意的是，选择 P-R 曲线还是 ROC 曲线是因实际问题而异的，如果研究者希望更多地看到模型在特定数据集上的表现，P-R 曲线则能够更直观地反映其性能。

## 为什么在一些场景中要使用余弦相似度而不是欧氏距离？



$$
\cos(\textbf{A},\textbf{B})  = \frac{\textbf{A}\cdot \textbf{B}}{||\textbf{A}||_2 ||\textbf{B}||_2}
=\frac{\sum_{1}^n a_i b_i}{\sqrt{\sum_{1}^n a_i^2}\sqrt{\sum_{1}^n b_i^2}}
$$


```python
import numpy as np
from math import sqrt

def norm(x):
    res = 0
    for i in x:
        res += i*i
    return sqrt(res)


def cos_sim(A, B):
    res = 0
    for a, b in zip(A, B):
        res += a*b
    return res/(norm(A)*norm(B))


def cos_sim_np(A, B):
    return np.dot(A, B)/(np.linalg.norm(A)*np.linalg.norm(B))
```

对于两个向量A和B，其余弦相似度定义为，即两个向量夹角的余弦，关注的是向量之间的角度关系，并不关心它们的绝对大小，其取值范围是 $[−1,1]$。当一对文本相似度的长度差距很大、但内容相近时，如果使用词频或词向量作为特征，它们在特征空间中的的欧氏距离通常很大；而如果使用余弦相似度的话，它们之间的夹角可能很小，因而相似度高。此外，在文本、图像、视频等领域，研究的对象的特征维度往往很高，余弦相似度在高维情况下依然保持“相同时为1，正交时为0，相反时为−1”的性质，而欧氏距离的数值则受维度的影响，范围不固定，并且含义也比较模糊。

在一些场景，例如Word2Vec中，其向量的模长是经过归一化的，此时欧氏距离与余弦距离有着单调的关系，即
$$
||\mathbf{A}-\mathbf{B}||_2=\sqrt{2(1-\cos(\mathbf{A},\mathbf{B}))}
$$
其中$||\mathbf{A}-\mathbf{B}||_2$表示欧氏距离，$\cos(\mathbf{A},\mathbf{B})$表示余弦相似度，$1-\cos(\mathbf{A},\mathbf{B})$表示余弦距离。在此场景下，如果选择距离最小（相似度最大）的近邻，那么使用余弦相似度和欧氏距离的结果是相同的。

总体来说，欧氏距离体现数值上的绝对差异，而余弦距离体现方向上的相对差异。例如，统计两部剧的用户观看行为，用户A的观看向量为 (0,1)，用户B为 (1,0)；此时二者的余弦距离很大，而欧氏距离很小；我们分析两个用户对于不同视频的偏好，更关注相对差异，显然应当使用余弦距离。而当我们分析用户活跃度，以登陆次数(单位：次)和平均观看时长(单位：分钟)作为特征时，余弦距离会认为 (1,10)、(10,100) 两个用户距离很近；但显然这两个用户活跃度是有着极大差异的，此时我们更关注数值绝对差异，应当使用欧氏距离。

特定的度量方法适用于什么样的问题，需要在学习和研究中多总结和思考，这样不仅仅对面试有帮助，在遇到新的问题时也可以活学活用。

## 余弦距离是否是一个严格定义的距离？

不是，距离有三个性质：

1. 正定性
2. 对称性
3. 三角不等式

## 在对模型进行充分的离线评估后，为什么还要进行在线的 A/B 测试？

1. 离线评估无法完全消除过拟合的影响。
2. 离线评估无法完全还原线上的工程环境。比如线上环境的延迟，数据丢失，标签数据确是等情况。因此，离线评估的结果是理想工程环境下的结果。
3. 线上系统的某些商业指标在离线评估中无法计算。离线评估一般是针对模型本身进行评估，而与模型相关的其他指标，特别是商业指标，往往无法直接获得。比如，上线了新的推荐算法，离线评估往往关注的是ROC曲线、P-R曲线等的改进，而线上评估可以全面了解该推荐算法带来的用户点击率、留存时长、PV访问量等的变化。这些都要由A/B测试来进行全面的评估。

## 进行线上 A/B 测试？

进行 A/B 测试的主要手段是进行用户分桶，即将用户分成实验组和对照组，对实验组的用户施以新模型，对对照组的用户施以旧模型。在分桶的过程中，要注意样本的独立性和采样方式的无偏性，确保同一个用户每次只能分到同一个桶中，在分桶过程中所选取的 user_id 需要是一个随机数，这样才能保证桶中的样本是无偏的。

## 如何划分实验组和对照组？

H公司的算法工程师们最近针对系统中的“美国用户”研发了一套全新的视频推荐模型A，而目前正在使用的针对全体用户的推荐模型是B。在正式上线之前，工程师们希望通过A/B测试来验证新推荐模型的效果。下面有三种实验组和对照组的划分方法，请指出哪种划分方法是正确的？
（1）根据user_id（user_id完全随机生成）个位数的奇偶性将用户划分为实验组和对照组，对实验组施以推荐模型A，对照组施以推荐模型B；
（2）将user_id个位数为奇数且为美国用户的作为实验组，其余用户为对照组；
（3）将user_id个位数为奇数且为美国用户的作为实验组，user_id个位数为偶数的用户作为对照组。

上述3种 A/B 测试的划分方法都不正确。我们用包含关系图来说明三种划分方法，如图2.4所示。方法1 没有区分是否为美国用户，实验组和对照组的实验结果均有稀释；方法2的实验组选取无误，并将其余所有用户划分为对照组，导致对照组的结果被稀释；方法3的对照组存在偏差。正确的做法是将所有美国用户根据user_id个位数划分为试验组合对照组，分别施以模型A和B，才能够验证模型A的效果。

![AB-test](/img/AB-test.png)

## 在模型评估过程中，有哪些主要的验证方法，它们的优缺点是什么？

1. Holdout检验

   Holdout 检验是最简单也是最直接的验证方法，它将原始的样本集合随机划分成训练集和验证集两部分。比方说，对于一个点击率预测模型，我们把样本按照 70%～30% 的比例分成两部分，70% 的样本用于模型训练；30% 的样本用于模型验证，包括绘制 ROC 曲线、计算精确率和召回率等指标来评估模型性能。Holdout 检验的缺点很明显，即在验证集上计算出来的最后评估指标与原始分组有很大关系。为了消除随机性，研究者们引入了“交叉检验”的思想。

   

2. 交叉检验

   k-fold 交叉验证：首先将全部样本划分成 k 个大小相等的样本子集；依次遍历这 k 个子集，每次把当前子集作为验证集，其余所有子集作为训练集，进行模型的训练和评估；最后把 k 次评估指标的平均值作为最终的评估指标。在实际实验中，k 经常取 10。

3. 自助法 Bootstrap

   不管是Holdout 检验还是交叉检验，都是基于划分训练集和测试集的方法进行模型评估的。然而，当样本规模比较小时，将样本集进行划分会让训练集进一步减小，这可能会影响模型训练效果。有没有能维持训练集样本规模的验证方法呢？自助法可以比较好地解决这个问题。

   自助法是基于自助采样法的检验方法。对于总数为 n 的样本集合，**进行 n 次有放回的随机抽样，得到大小为 n 的训练集**。n 次采样过程中，有的样本会被重复采样，有的样本没有被抽出过，将这些没有被抽出的样本作为验证集，进行模型验证，这就是自助法的验证过程。

## 在自助法的采样过程中，对n个样本进行n次自助抽样，当n趋于无穷大时，最终有多少数据从未被选择过？

一个样本在一次抽样中未被抽中的概率为 $(1-\frac{1}{n})$

n 次抽样均未被抽中的概率为 $(1-\frac{1}{n})^n$

所以

$$
\begin{aligned}
\lim _{n\rightarrow  \infty} (1-\frac{1}{n})^n &= \lim _{n\rightarrow  \infty}(\frac{n-1}{n})^n \\\\
&=\lim _{n\rightarrow  \infty}(\frac{1}{1+\frac{1}{n-1}})^n \\\\
&= \frac{1}{\lim _{n\rightarrow  \infty}(1+\frac{1}{n-1})^{n-1}} *\frac{1}{\lim _{n\rightarrow  \infty}(1+\frac{1}{n-1}) } \\\\
&=\frac{1}{e} *1 (重要极限) \\\\
&\approx 0.368
\end{aligned}
$$

因此，当样本数很大时，大约有36.8%的样本从未被选择过，可作为验证集。


## 超参数有哪些调优方法？

为了进行超参数调优，我们一般会采用网格搜索、随机搜索、贝叶斯优化等算法。在具体介绍算法之前，需要明确超参数搜索算法一般包括哪几个要素。一是目标函数，即算法需要最大化/最小化的目标；二是搜索范围，一般通过上限和下限来确定；三是算法的其他参数，如搜索步长。

一、网格搜索

网格搜索可能是最简单、应用最广泛的超参数搜索算法，它通过查找搜索范围内的所有的点来确定最优值。如果采用较大的搜索范围以及较小的步长，网格搜索有很大概率找到全局最优值。然而，这种搜索方案十分消耗计算资源和时间，特别是需要调优的超参数比较多的时候。因此，在实际应用中，网格搜索法一般会先使用较广的搜索范围和较大的步长，来寻找全局最优值可能的位置；然后会逐渐缩小搜索范围和步长，来寻找更精确的最优值。这种操作方案可以降低所需的时间和计算量，但由于目标函数一般是非凸的，所以很可能会错过全局最优值。

二、随机搜索

随机搜索的思想与网格搜索比较相似，只是不再测试上界和下界之间的所有值，而是在搜索范围中随机选取样本点。它的理论依据是，如果样本点集足够大，那么通过随机采样也能大概率地找到全局最优值，或其近似值。随机搜索般会比网格搜索要快一些，但是和网格搜索的快速版一样，它的结果也是没法保证的。

三、贝叶斯搜索

贝叶斯优化算法在寻找最优最值参数时，采用了与网格搜索、随机搜索完全不同的方法。网格搜索和随机搜索在测试一个新点时，会忽略前一个点的信息；而贝叶斯优化算法则充分利用了之前的信息。

贝叶斯优化算法通过对目标函数形状进行学习，找到使目标函数向全局最优值提升的参数。具体来说，它学习目标函数形状的方法是，首先根据先验分布，假设一个搜集函数；然后，每一次使用新的采样点来测试目标函数时，利用这个信息来更新目标函数的先验分布；最后，算法测试由后验分布给出的全局最值最可能出现的位置的点。对于贝叶斯优化算法，有一个需要注意的地方，一旦找到了一个局部最优值，它会在该区域不断采样，所以很容易陷入局部最优值。为了弥补这个缺陷，贝叶斯优化算法会在探索和利用之间找到一个平衡点，“探索”就是在还未取样的区域获取采样点；而“利用”则是根据后验分布在最可能出现全局最值的区域进行采样。

## 在模型评估过程中，过拟合和欠拟合具体是指什么现象？

过拟合是指模型对于训练数据拟合呈过当的情况，反映到评估指标上，就是模型在训练集上的表现很好，但在测试集和新数据上的表现较差。欠拟合指的是模型在训练和预测时表现都不好的情况。欠拟合时，模型没有很好地捕捉到数据的特征，不能够很好地拟合数据。过拟合时，模型过于复杂，把噪声数据的特征也学习到模型中，导致模型泛化能力下降，在后期应用过程中很容易输出错误的预测结果。

## 能否说出几种降低过拟合和欠拟合风险的方法？

- 降低“过拟合”风险的方法

  1. 从数据入手，获得更多的训练数据。使用更多的训练数据是解决过拟合问题最有效的手段，因为更多的样本能够让模型学习到更多更有效的特征，减噪声的影响。当然，直接增加实验数据一般是很困难的，但是可以通过一定的规则来扩充训练数据。比如，在图像分类的问题上，可以通过图像的平移、旋转、缩放等方式扩充数据；更进一步地，可以使用生成式对抗网络来合成大量的新训练数据。

  2. 降低模型复杂度。在数据较少时，模型过于复杂是产生过拟合的主要因素，适当降低模型复杂度可以避免模型拟合过多的采样噪声。例如，在神经网络模型中减少网络层数、神经元个数等；在决策树模型中降低树的深度、进行剪枝等。

  3. 正则化方法。给模型的参数加上一定的正则约束，比如将权值的大小加入到损失函数中。以L2正则化为例：
     $$
     C = C_0 + \frac{\lambda}{2n}\cdot \sum_iw_i^2
     $$
     

     这样，在优化原来的目标函数 $c_0$ 的同时，也能避免权值过大带来的过拟合风险。

  4. 集成学习方法。集成学习是把多个模型集成在一起，来降低单一模型的过拟合风险，如Bagging方法。

- 降低“欠拟合”风险的方法

  1. 添加新特征。当特征不足或者现有特征与样本标签的相关性不强时，模型容易出现欠拟合。通过挖掘“上下文特征”“ID类特征”“组合特征”等新的特征，往往能够取得更好的效果。在深度学习潮流中，有很多模型可以帮助完成特征工程，如因子分解机、梯度提升决策树、Deep-crossing等都可以成为丰富特征的方法。
  2. 增加模型复杂度。简单模型的学习能力较差，通过增加模型的复杂度可以使模型拥有更强的拟合能力。例如，在线性模型中添加高次项，在神经网络模型中增加网络层数或神经元个数等。
  3. 减小正则化系数。正则化是用来防止过拟合的，但当模型出现欠拟合现象时，则需要有针对性地减小正则化系数。

# 经典算法