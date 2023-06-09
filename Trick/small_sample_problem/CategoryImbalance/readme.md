# 【关于 类别不均衡问题】 那些你不知道的事

## 1. 什么是类别不均衡？

机器学习中常常会遇到数据的类别不平衡（class imbalance），也叫数据偏斜（class skew）。

> 以常见的二分类问题为例，我们希望预测病人是否得了某种罕见疾病。但在历史数据中，阳性的比例可能很低（如百分之0.1）。在这种情况下，学习出好的分类器是很难的，而且在这种情况下得到结论往往也是很具迷惑性的。

以上面提到的场景来说，如果我们的分类器总是预测一个人未患病，即预测为反例，那么我们依然有高达99.9%的预测准确率。然而这种结果是没有意义的，这提出了今天的第一个问题，如何有效在类别不平衡的情况下评估分类器？

## 2. 什么是类别不均衡？

- 数据均衡场景：
  - 方法：准确率（accuracy）
  - 前提：**“数据是平衡的，正例与反例的重要性一样，二分类器的阈值是0.5。”**

- 数据不均衡场景：
  - 动机：准确率（accuracy）容易出现迷惑性（eg：正例有99条，负例1条，那么acc = 99%，然而这种结果是没有意义的）
  - 主流的评估方法:
    - ROC是一种常见的替代方法，全名receiver operating curve，计算ROC曲线下的面积是一种主流方法
    - Precision-recall curve和ROC有相似的地方，但定义不同，计算此曲线下的面积也是一种方法
    - Precision@n是另一种方法，特制将分类阈值设定得到恰好n个正例时分类器的precision
    - Average precision也叫做平均精度，主要描述了precision的一般表现，在异常检测中有时候会用
    - 直接使用Precision也是一种想法，但此时的假设是分类器的阈值是0.5，因此意义不大
  - 如何选择：至于哪种方法更好，一般来看我们在极端类别不平衡中更在意“少数的类别”，因此ROC不像precision-recall curve那样更具有吸引力。在这种情况下，Precision-recall curve不失为一种好的评估标准，相关的分析可以参考[2]。还有一种做法是，仅分析ROC曲线左边的一小部分，从这个角度看和precision-recall curve有很高的相似性。

> 注：precision 使用 注意点：<br/>
> 没有特殊情况，不要用准确率（accuracy），一般都没什么帮助<br/>
> 如果使用precision，请注意调整分类阈值，precision@n更有意义<br/>

## 3. 解决类别不平衡中的“奇淫巧技”有什么？

1. **SMOTE算法**：对数据进行采用的过程中通过相似性同时生成并插样“少数类别数据”，叫做SMOTE算法
2. **聚类算法做欠采样**：对数据先进行聚类，再将大的簇进行随机欠采样或者小的簇进行数据生成
3. **阈值调整（threshold moving）**，将原本默认为0.5的阈值调整到 较少类别/（较少类别+较多类别）即可
   1. 介绍：简单的调整阈值，不对数据进行任何处理。此处特指将分类阈值从0.5调整到正例比例
4. **无监督学习问题**：把监督学习变为无监督学习，舍弃掉标签把问题转化为一个无监督问题，如异常检测
5. **集成学习**: 先对多数类别进行随机的欠采样，并结合boosting算法进行集成学习
6. 重构分类器角度
   1. 将大类压缩为小类；
   2. 使用 One Class 分类器（将小类作为异常点）；
   3. 使用集成方式，训练多个分类器，联合这些分类器进行分类；
   4. 将二分类改为多分类问题；
7. 使用其他 Loss 函数
   1. Focal Loss: 对 CE Loss 增加了一个调制系数来降低容易样本的权值，使其训练过程更加关注困难样本

> 注：第一种和第二种方法都会明显的改变数据分布，我们的训练数据假设不再是真实数据的无偏表述。在第一种方法中，我们浪费了很多数据。而第二类方法中有无中生有或者重复使用了数据，会导致过拟合的发生。

## 参考

1. [如何处理数据中的「类别不平衡」？](https://zhuanlan.zhihu.com/p/32940093)