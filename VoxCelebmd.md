VoxCeleb2：

数据库的目标是wild（有噪音、不受限制的条件）下的说话人识别。



## 混肴矩阵（confusion matrix）

TP: 预测为正，实际为正

TN: 预测为负，实际为负

FP:预测为正，实际为负

FN: 预测为负，实际为正

T/F：表示预测结果是否正确

P/N：表示预测结果是正或负样本

精确率、准确率：Accuracy=(TP+TN)/(TP+TN+FN+FP)

精准率、查准率： P = TP/ (TP+FP)

召回率、查全率： R = TP/ (TP+FN)

真正例率(同召回率、查全率)：TPR = TP/ (TP+FN)

用矩阵表示的结果就是

![img](https://images0.cnblogs.com/blog/558074/201312/12165103-5cf18403b72b485d8a6a669fa6723abe.png)

## ROC曲线

通过混肴矩阵，我们可以得到真正例率（TPR）：TPR = TP/(TP+FN)，即正样本被正确判定的概率

假正例率（FPR）：FPR = FP/(FP+TN)，即负样本被误判的概率

给定一个决策边界阈值，就可以得到一个对应的TPR和FPR。通过检测大量的阈值，得到一个TPR-FPR的相关图，如下所示

![img](https://images0.cnblogs.com/blog/558074/201312/12165107-6c5f5cc6a7684c42a222cef08bb6810d.png)

图中的红色曲线和蓝色曲线分别表示了两个不同的分类器的TPR-FPR曲线，曲线上的任意一点都对应了一个阈值。该曲线就是ROC曲线（receiver operating characteristic curve）。该曲线具有以下特征：

- 一定经过（0,0）点，此时阈值为1，没有预测为P的值，TP和FP都为0
- 一定经过（1,1）点，此时阈值为0，全都预测为P
- 最完美的分类器（完全区分正负样例）：（0,1）点，即没有FP，全是TP
- 曲线越是“凸”向左上角，说明分类器效果越好
- 随机预测会得到（0,0）和（1,1）的直线上的一个点
- 曲线上离（0,1）越近的点分类效果越好，对应着越合理的阈值

​    从图中可以看出，红色曲线所代表的分类器效果好于蓝色曲线所表示的分类器。

## EER（Equal Error Rate）

EER是ROC曲线中错分正负样本概率相等的点。也就是FPR=FNR的值，由于FNR=1-TPR，可以画一条从（0,1）到（1,0）的直线，找到交点，图中的A、B两点。

除了EER，利用ROC还有其他的评估标准，比如AUC(area under thecurve)，也就是ROC曲线的下夹面积，越大说明分类器越好，最大值是1，图中的蓝色条纹区域面积就是蓝色曲线对应的 AUC。

## Contrastive Loss

在孪生神经网络（Siamese network）中，其采用的损失函数是contrastive loss，这种损失函数可以有效处理孪生神经网络中的paired data的关系。该损失函数的表达式如下：

![img](https://img-blog.csdn.net/20180214171635734?)

其中*d*=||an−bn||2，代表两个样本特征的欧氏距离，y为两个样本是否匹配的标签，y=1代表两个样本相似或者匹配，y=0则代表不匹配，margin为设定的阈值。

这种损失函数最初来源于Yann LeCun的《Dimensionality Reduction by Learning an Invariant Mapping》，主要是用在降维中，即本来相似的样本，在经过降维（特征提取）后，在特征空间中，两个样本仍旧相似；而原本不相似的样本，在经过降维后，在特征空间中，两个样本仍旧不相似。

观察上述的contrastive loss的表达式可以发现，这种损失函数可以很好的表达成对样本的匹配程度，也能够很好用于训练提取特征的模型。当y=1（即样本相似）时，损失函数只剩下∑yd2，即原本相似的样本，如果在特征空间的欧式距离较大，则损失函数（增函数）越大，则说明当前的模型不好。而当y=0时（即样本不相似）时，损失函数为∑(1−y)max(margin−d,0)2（减函数），即当样本不相似时，其特征空间的欧式距离反而小的话，损失函数值会变大。

![img](https://img-blog.csdn.net/20161113163224993)

这张图表示的就是损失函数值与样本特征的欧式距离之间的关系，其中红色虚线表示的是相似样本的损失值（Y=1时），蓝色实线表示的不相似样本的损失值（Y=0时）。这里的m为阈值，要视具体问题而定，对于不同的目标m的值会有不同的大小。而事实表明Constractive Loss对于多分类的问题经常会在训练集上过拟合，显得比较乏力。

针对该问题的改进方法有Triplet Loss、四元组损失(Quadruplet loss)、难样本采样三元组损失(Triplet loss with batch hard mining, TriHard loss)、边界挖掘损失(Margin sample mining loss, MSML)

## Embedding

通俗理解：

Embedding在数学上表示一个maping, f: X -> Y，也就是一个function，其中该函数是injective（就是我们所说的单射函数，每个Y只有唯一的X对应，反之亦然）和structure-preserving (结构保存，比如在X所属的空间上X1 < X2,那么映射后在Y所属空间上同理 Y1 < Y2)。那么对于word embedding，就是将单词word映射到另外一个空间，其中这个映射具有injective和structure-preserving的特点。通俗的翻译可以认为是单词嵌入，就是把X所属空间的单词映射为到Y空间的多维向量，那么该多维向量相当于嵌入到Y所属空间中，一个萝卜一个坑。word embedding，就是找到一个映射或者函数，生成在一个新的空间上的表达，该表达就是word representation。  

官方定义：

一种分类特征，以连续值特征表示。通常，嵌套是指将高维度向量映射到低维度的空间。例如，您可以采用以下两种方式之一来表示英文句子中的单词：

- 表示成包含百万个元素（高维度）的[**稀疏向量**](https://link.jianshu.com?t=https%3A%2F%2Fdevelopers.google.cn%2Fmachine-learning%2Fglossary%2F%23sparse_features)，其中所有元素都是整数。向量中的每个单元格都表示一个单独的英文单词，单元格中的值表示相应单词在句子中出现的次数。由于单个英文句子包含的单词不太可能超过 50 个，因此向量中几乎每个单元格都包含 0。少数非 0 的单元格中将包含一个非常小的整数（通常为 1），该整数表示相应单词在句子中出现的次数。
- 表示成包含数百个元素（低维度）的[**密集向量**](https://link.jianshu.com?t=https%3A%2F%2Fdevelopers.google.cn%2Fmachine-learning%2Fglossary%2F%23dense_feature)，其中每个元素都包含一个介于 0 到 1 之间的浮点值。这就是一种嵌套。

> 嵌套本质上是降维，是化稀疏为密集。

### 嵌套的概念

嵌套是一种映射，将稀疏不连续的对象映射到实数向量。

比如下面是300维度的英语单词嵌套：

```
blue:  (0.01359, 0.00075997, 0.24608, ..., -0.2524, 1.0048, 0.06259)
blues:  (0.01396, 0.11887, -0.48963, ..., 0.033483, -0.10007, 0.1158)
orange:  (-0.24776, -0.12359, 0.20986, ..., 0.079717, 0.23865, -0.014213)
oranges:  (-0.35609, 0.21854, 0.080944, ..., -0.35413, 0.38511, -0.070976)
```

向量中的这些单独的维度并没有什么具体意义，它们是向量的位置、距离关系的整体图式表达，以便于机器学习使用。

嵌套对于机器学习的输入非常重要。分类器Classifier、神经网络Neura networks普遍工作于实数向量Real number vector。训练Train最好是基于密集向量dense vector，全部所有数值共同定义对象。但是，对于机器学习来说，很多重要的输入比如文本的单词，都没有自然的向量表现形式，嵌套函数就是这么一个标准且有效的函数，可以把稀疏离散discrete/sparse的对象变为连续的向量表示法。

嵌套也作为机器学习的输出值。由于嵌套将物体映射为向量，应用程序可以使用向量空间相似的方法，强健且灵活的估算物体之间的相似性。一个通用的用途就是发现最近邻对象。

比如上面的单词向量，下面是每个单词的三个最近邻单词，用角度来展示：

```
blue:  (red, 47.6°), (yellow, 51.9°), (purple, 52.4°)
blues:  (jazz, 53.3°), (folk, 59.1°), (bluegrass, 60.6°)
orange:  (yellow, 53.5°), (colored, 58.0°), (bright, 59.9°)
oranges:  (apples, 45.3°), (lemons, 48.3°), (mangoes, 50.4°)
```

最后一行数字告诉应用程序oranges和apples比较近似(分离45.3度），而和lemons，mangoes稍微区别大一些（48.3，50.4）。

### Tensorflow中的嵌套Embedding

为了创建单词的嵌套，我们首先把文本划分为单词，并为每个词汇指定一个整数张量。假设这已经完成，word_ids表示包含这些整数的向量。比如句子，I have a cat.被划分为['I','have','a','cat',',']，对应的word_ids张量是形状shape[5]，由5个整数组成，比如[32,177,4,23,16]。要把这些单词映射为向量，我们需要使用`tf.nn.embedding_lookup`来生成嵌套变量。

```
word_embeddings = tf.get_variable(“word_embeddings”,
    [vocabulary_size, embedding_size])
embedded_word_ids = tf.nn.embedding_lookup(word_embeddings, word_ids)
```

如上，张量embedded_word_ids的形状变为[5,embedding_size],包含了5个嵌套（密集矢量）对应每个单词。训练结束后，word_embeddings将包含所有词汇单词的嵌套。

```
#如果Embedding_size=3，那么embedded_word_ids可能是
[[0.438890,0.782233,0.52721],
 [0.645432,0.523233,0.62333],
 [0.412333,0.124522,0.67223],
 [0.145333,0.133422,0.67223],
 [0.988888,0.765556,0.13344],
]
```

嵌套可以被多种网络类型训练，各种损失函数和数据集。例如，一个使用卷积神经网络RNN依赖词库去预测某个单词后面的下一个单词，或者使用两个网络进行语言翻译。

### 视觉化嵌套Visualizing Embeddings

Tensorboard中包含的嵌套投影器Embedding projector，它可以交互的显示嵌套，他可以从模型中读取嵌套并渲染到3D空间。

Embedding projector有三个面板：

- Data panel
- Projections panel
- Inspector panel

此处省去很多内容的翻译...

## discriminative判别式模型



## CNN & Pooling

[CNN的推导](<https://www.cnblogs.com/believe-in-me/p/6640430.html>)

[卷积和池化、全连接层](<https://www.cnblogs.com/believe-in-me/p/6645402.html>)

## Attention



# FFT

FFT程序，输入是一组复数，输出也是一组复数。那么，输入的到底是什么？输出的复数具有什么含义？给定一组序列的抽样值，如何用FFT确定它的频率？

  首先，fft函数出来的应该是个复数，每一个点分实部虚部两部分。假设采用1024点fft，[采样频率](https://www.baidu.com/s?wd=%E9%87%87%E6%A0%B7%E9%A2%91%E7%8E%87&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)是fs，那么第一个点对应0频率点，第512点对应的就是fs/2的频率点。然后[从头开始](https://www.baidu.com/s?wd=%E4%BB%8E%E5%A4%B4%E5%BC%80%E5%A7%8B&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)找模值最大的那个点，其所对应的频率值应该就是你要的基波频率了。
FFT是离散傅立叶变换的快速算法，可以将一个信号变换到频域。有些信号在时域上是很难看出什么特征的，但是如果变换到频域之后，就很容易看出特征了。这就是很多信号分析采用FFT变换的原因。另外，FFT可以将一个信号的频谱提取出来，这在频谱分析方面也是经常用的。
虽然很多人都知道FFT是什么，可以用来做什么，怎么去做，但是却不知道FFT之后的结果是什么意思、如何决定要使用多少点来做FFT。一个模拟信号，经过ADC采样之后，就变成了数字信号。采样定理告诉我们，[采样频率](https://www.baidu.com/s?wd=%E9%87%87%E6%A0%B7%E9%A2%91%E7%8E%87&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)要大于[信号频率](https://www.baidu.com/s?wd=%E4%BF%A1%E5%8F%B7%E9%A2%91%E7%8E%87&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)的两倍，这些我就不在此罗嗦了。采样得到的数字信号，就可以做FFT变换了。N个采样点，经过FFT之后，就可以得到N个点的FFT结果。为了方便进行FFT运算，通常N取2的整数次方。

假设[采样频率](https://www.baidu.com/s?wd=%E9%87%87%E6%A0%B7%E9%A2%91%E7%8E%87&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)为Fs，[信号频率](https://www.baidu.com/s?wd=%E4%BF%A1%E5%8F%B7%E9%A2%91%E7%8E%87&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)F，采样点数为N。那么FFT之后结果就是一个为N点的复数。每一个点就对应着一个频率点。这个点的模值，就是该频率值下的幅度特性。具体跟原始信号的幅度有什么关系呢？假设原始信号的峰值为A，那么FFT的结果的每个点（除了第一个点[直流分量](https://www.baidu.com/s?wd=%E7%9B%B4%E6%B5%81%E5%88%86%E9%87%8F&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)之外）的模值就是A的N/2倍。而第一个点就是[直流分量](https://www.baidu.com/s?wd=%E7%9B%B4%E6%B5%81%E5%88%86%E9%87%8F&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)，它的模值就是[直流分量](https://www.baidu.com/s?wd=%E7%9B%B4%E6%B5%81%E5%88%86%E9%87%8F&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)的N倍。而每个点的相位呢，就是在该频率下的信号的相位。第一个点表示直流分量（即0Hz），而最后一个点N的再下一个点（实际上这个点是不存在的，这里是假设的第N+1个点，可以看做是将第一个点分做两半分，另一半移到最后）则表示采样频率Fs，这中间被N-1个点平均分成N等份，每个点的频率依次增加。例如某点n所表示的频率为：Fn =(n-1)Fs/N。由上面的公式可以看出，Fn所能分辨到频率为 Fs/N，如果采样频率Fs为1024Hz，采样点数为1024点，则可以分辨到1Hz。1024Hz的采样率采样1024点，刚好是1秒，也就是说，采样1秒时间的信号并做FFT，则结果可以分析到1Hz，如果采样2秒时间的信号并做FFT，则结果可以分析到0.5Hz。如果要提高频率分辨力，则必须增加采样点数，也即[采样时间](https://www.baidu.com/s?wd=%E9%87%87%E6%A0%B7%E6%97%B6%E9%97%B4&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)。频率分辨率和[采样时间](https://www.baidu.com/s?wd=%E9%87%87%E6%A0%B7%E6%97%B6%E9%97%B4&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)是倒数关系。假设FFT之后某点n用复数a+bi表示，那么这个复数的模就是An=根号aa+bb，相位就是Pn=[atan2](https://www.baidu.com/s?wd=atan2&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)(b,a)。根据以上的结果，就可以计算出n点（n≠1，且n<=N/2）对应的信号的表达式为：An/(N/2)cos(2piFnt+Pn)，即2An/Ncos(2piFnt+Pn)。*对于n=1点的信号，是直流分量，幅度即为A1/N。由于FFT结果的对称性，通常我们只使用前半部分的结果，即小于采样频率一半的结果。  



## 训练过程

VGGVox网络：神经嵌入系统（以不加预处理的原始音频的语谱图为训练数据，主干网络提取帧级别特征，然后利用pool（池化？）成句子级别的说话人嵌入向量speaker embedding）。

整个训练过程使用的是**contrastive loss**。

预训练使用soft-max层和交叉熵，在一定的说话人数据上展开预训练来提升模型的性能，因此，我们首先预训练话者识别主干网络。

### 评估

训练集：VoxCeleb2，训练时在线生成pairs

测试集：VoxCeleb1，数据集中含有的pairs

性能度量标准：

- EER（Equal Error Rate）：错分正负样本概率相等的值。

- loss function：

  ![1558435760497](C:\Users\wangliyuan\AppData\Roaming\Typora\typora-user-images\1558435760497.png)

  

### 主干网络结构

**VGG-M：**和VoxCeleb1中使用的CNN网络一样，它是对VGG-M网络（对图像有很好的分类效果）的修改。它把原始VGG-M的全连接层fc6改成了两层结构——一个全连接层9x1和一个平均池化层1xn，n取决于输入语音的长度，比如语音是3秒时n=8.**这种修改的意义在于网络不受时间位置的影响，而是受频率的影响，这对语音来说是可取的，但不适合图像。**该网络的输出维度和原始全连接层的输出维度一样，并通过fivefold减少了神经网络的参数。

**ResNets：**残差网络和一个标准的多层CNN网络相似，但添加了skip connection，（所以各层增加了残差？）。



### 训练损失策略

