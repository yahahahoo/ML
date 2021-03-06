
## 集成学习

**集成学习**（Ensemble Learning)：集成学习就是组合多个弱模型以期得到一个更好更全面的强模型，集成学习潜在的思想是即便某一个弱分类器得到了错误的预测，其他的弱分类器也可以将错误纠正回来。

Bagging：bootstrap aggregating，bootstrap也称为自助法，它是一种有放回的抽样方法，利用bootstrap方法从整体数据集中采取有放回抽样得到N个数据集，在每个数据集上学习出一个模型，最后的预测结果利用N个模型的输出得到，具体地：分类问题采用N个模型预测投票的方式，回归问题采用N个模型预测平均的方式。例如随机森林（Random Forest）就属于Bagging。

Boosting：提升方法（Boosting）是一种可以用来减小监督学习中偏差的机器学习算法。主要也是学习一系列弱分类器，并将其组合为一个强分类器。Boosting中有代表性的是AdaBoost（Adaptive boosting）算法。刚开始训练时对每一个训练例赋相等的权重，然后用该算法对训练集训练t轮，每次训练后，对训练失败的训练例赋以较大的权重，也就是让学习算法在每次学习以后更注意学错的样本，从而得到多个预测函数。

Stacking：Stacking方法是指训练一个模型用于组合其他各个模型。首先我们先训练多个不同的模型，然后把之前训练的各个模型的输出为输入来训练一个模型，以得到一个最终的输出。理论上，Stacking可以表示上面提到的两种Ensemble方法，只要我们采用合适的模型组合策略即可。但在实际中，我们通常使用logistic回归作为组合策略。

 - Bagging和Boosting采用的都是采样-学习-组合的方式，但在细节上有一些不同，如Bagging中每个训练集互不相关，也就是每个基分类器互不相关，而Boosting中训练集要在上一轮的结果上进行调整，也使得其不能并行计算。
 - Bagging中预测函数是均匀平等的，但在Boosting中预测函数是加权的。
 - Bagging集成之后方差变小了，偏差不变。
 - Boosting主要还是靠降低bias来提升预测精度，并不能显著降低variance。

## 随机森林

随机森林是一种基于树模型的Bagging的优化版本，一棵树的生成肯定还是不如多棵树，因此就有了随机森林，解决决策树泛化能力弱的特点。而同一批数据，用同样的算法只能产生一棵树，这时Bagging策略可以帮助我们产生不同的数据集。Bagging策略来源于bootstrap aggregation：从样本集（假设样本集N个数据点）中重采样选出Nb个样本（有放回的采样，样本数据点个数仍然不变为N），在所有样本上，对这n个样本建立分类器（ID3\C4.5\CART\SVM\LOGISTIC），重复以上两步m次，获得m个分类器，最后根据这m个分类器的投票结果，决定数据属于哪一类。
每棵树的按照如下规则生成：

1. 如果训练集大小为N，对于每棵树而言，随机且有放回地从训练集中的抽取n个训练样本，作为该树的训练集；
2. 如果每个样本的特征维度为M，指定一个常数m<<M，随机地从M个特征中选取m个特征子集，每次树进行分裂时，从这m个特征中选择最优的；
3. 每棵树都尽最大程度的生长，并且没有剪枝过程。

一开始我们提到的随机森林中的“随机”就是指的这里的两个随机性。两个随机性的引入对随机森林的分类性能至关重要。由于它们的引入，使得随机森林不容易陷入过拟合，并且具有很好得抗噪能力（比如：对缺省值不敏感）。总的来说就是**随机选择样本数，随机选取特征，随机选择分类器**，建立多颗这样的决策树，然后通过这几棵决策树来投票，决定数据属于哪一类(投票机制有一票否决制、少数服从多数、加权多数)


### 随机森林分类效果的影响因素

- 森林中任意两棵树的相关性：相关性越大，错误率越大。
- 森林中每棵树的分类能力：每棵树的分类能力越强，整个森林的错误率越低。

减小特征选择个数m，树的相关性和分类能力也会相应的降低；增大m，两者也会随之增大。所以关键问题是如何选择最优的m（或者是范围），这也是随机森林唯一的一个参数。

### 随机森林有什么优缺点

优点：
1. 在当前的很多数据集上，相对其他算法有着很大的优势，表现良好。
2. 它能够处理很高维度（feature很多）的数据，并且不用做特征选择(因为特征子集是随机选择的)。
3. 在训练完后，它能够给出哪些feature比较重要。
4. 训练速度快，容易做成并行化方法(训练时树与树之间是相互独立的)。
5. 在训练过程中，能够检测到feature间的互相影响。
6. 对于不平衡的数据集来说，它可以平衡误差。
7. 如果有很大一部分的特征遗失，仍可以维持准确度。

缺点：
1. 随机森林已经被证明在某些噪音较大的分类或回归问题上会过拟合。
2. 对于有不同取值的属性的数据，取值划分较多的属性会对随机森林产生更大的影响，所以随机森林在这种数据上产出的属性权值是不可信的。

### 随机森林如何处理缺失值？

根据随机森林创建和训练的特点，随机森林对缺失值的处理还是比较特殊的。
- 首先，给缺失值预设一些估计值，比如数值型特征，选择其余数据的中位数或众数作为当前的估计值
- 然后，根据估计的数值，建立随机森林，把所有的数据放进随机森林里面跑一遍。记录每一组数据在决策树中一步一步分类的路径.
- 判断哪组数据和缺失数据路径最相似，引入一个相似度矩阵，来记录数据之间的相似度，比如有N组数据，相似度矩阵大小就是N*N
- 如果缺失值是类别变量，通过权重投票得到新估计值，如果是数值型变量，通过加权平均得到新的估计值，如此迭代，直到得到稳定的估计值。
其实，该缺失值填补过程类似于推荐系统中采用协同过滤进行评分预测，先计算缺失特征与其他特征的相似度，再加权得到缺失值的估计，而随机森林中计算相似度的方法（数据在决策树中一步一步分类的路径）乃其独特之处。

### 什么是OOB？随机森林中OOB是如何计算的，它有什么优缺点？

上面我们提到，构建随机森林的关键问题就是如何选择最优的m，要解决这个问题主要依据计算**袋外错误率**oob error（out-of-bag error）。bagging方法中Bootstrap每次约有1/3的样本**不会出现**在Bootstrap所采集的样本集合中，当然也就没有参加决策树的建立，把这1/3的数据称为袋外数据oob（out of bag）,它可以用于取代测试集误差估计方法。

