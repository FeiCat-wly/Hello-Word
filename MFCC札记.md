# MFCC札记

### 频谱spectrum



<https://www.cnblogs.com/BaroC/p/4283380.html>



从它的全称开始：Mel Frequency Cepstral Coefficents

以前没有听过Cepstral吧？它是spectral的spec倒过来后的单词。



声道的shape决定了发出的声音是怎样的，其表现为短时功率谱的包络，MFCC则是表征这这种envelope。

快速一瞥：

1.将语音分成短帧（一般20-40ms，为了简化，假设的语音信号的短时平稳性）

2.对每一帧，计算perioogram estimate。



**知乎好文**

[傅里叶分析之掐死教程（完整版）](<https://zhuanlan.zhihu.com/p/19763358>)

[从另一个角度看拉普拉斯变换](<https://zhuanlan.zhihu.com/p/40783304>)

[MFCC 梅尔频率倒谱系数](<https://zhuanlan.zhihu.com/p/23305179>)

[Matlab实现](<http://mirlab.org/jang/books/audioSignalProcessing/speechFeatureMfcc_chinese.asp?title=12-2%20MFCC>)

[深入理解FT，DTFT，DFT 之间的关系](<https://blog.csdn.net/weixin_40679412/article/details/80426463>)





[Kaldi features](<http://www.kaldi-asr.org/doc/feat.html>)

[kaldi中TransitionModel介绍](<https://blog.csdn.net/u013677156/article/details/79136418>)

# TDNN 

顾名思义，在考虑当前帧的时候，同时考虑一定数量的前后帧。

它具有以下特性：

- 多层的feedforward NN及节点之间紧密的连接使得其可以表示复杂的非线性分类面；
- Time-Delay使得其可以学习到特征之间的时序依赖；
- 学习到的特征具有时移不变性，同一个音素出现在语音的不同位置学到的特征应该尽可能相近；
- 学习过程中特征和标签不需要精确地对齐；
- 参数数量应该远小于训练样本的数量；
- 相同时延位置，权值共享

[语音识别TDNN](<https://blog.csdn.net/Xwei1226/article/details/80181655>)