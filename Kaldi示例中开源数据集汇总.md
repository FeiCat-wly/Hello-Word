# Kaldi示例中开源数据集汇总

## aishell

中文普通话语料库，

## aishell2

中文语音识别最大语料库

1000小时语音数据，1991位说话者

## ami

欧洲资助的AMI联盟，致力于研究和开发有助于团体更好互动的技术。

AMI Meeting corpus包含了100小时的会议记录。包括进场、远场麦克风，相机，幻灯片放映机和电子白板。

这组记录会议的集合还可以用于语言学、语音和语言工程、视频处理和多模态系统中。

## an4/s5

CMU于1991年内部记录的字母数字语料库。Sphere-format database

## apiai_decode/s5

脚本：使用预训练的英语链式模型和kaldi基础来识别任意wav。

训练数据使用librispeech混合api.ai数据。

api.ai公司。

## aspire

JHU约翰霍普金斯大学在ASpIRE挑战赛中提交的代码。它使用Fisher-English语料集来训练声学模型和语言模型。它使用RWCP，AIR和Reverb2014数据集中的脉冲响应和噪音来产生多种条件的数据。

ASpiRE挑战赛：Automatic Speech recognition In Reverberant Environments (ASpIRE) [Challenge](https://www.iarpa.gov/index.php/working-with-iarpa/prize-challenges/306-automatic-speech-in-reverberant-environments-aspire-challenge) sponsored by IARPA (the Intelligent Advanced Research Projects Activity). 混响环境下的自动语音识别技术挑战赛，由智能前沿研究项目活动赞助。

## aurora4

微软，JHU Daniel Povey，研究带噪音的语音识别，包括说话人分离、音乐分离、噪声分离。使用华尔街日报语料集wsj0和其中加各种噪音后的数据。

## babel

JHU Yenda Trmal。

BABEL数据集，

对低资源语言的语音识别和关键词检索，包括普什语、波斯语、土耳其语、越南语等等，wer达50以上。

## babel_multilang/s5

基于BABEL数据集的多语种识别

## bentham

JHU 2018年发布

手写识别代码

基于2014年ICFHR手写识别挑战赛发布的Bentham数据集

## bn_music_speech/v1

MUSAN数据集，包括音乐，语音和噪声。由Daniel Povey等人发布。

它由国家科学基金研究生研究奖学金和Spoken Communication赞助。

{MUSAN}: {A} {M}usic, {S}peech, and {N}oise {C}orpus 2015年

代码2015年发布，验证音乐和语音的区别。在MUSAN中的音乐、语音、噪音上训练GMM模型，用广播新闻数据集做测试，其中包含短片的语音或音乐。

示例代码后面有VAD代码。

## callhome_diarization

说话人分类示例代码

语料集是2000 NIST SRE评测中的CALLHOME数据集的一部分

## callhome_egyptian

基于埃及阿拉伯语料集的kaldi语音识别示例代码

## chime1

第一届chime挑战赛的kaldi示例

University of Sheffield 谢菲尔德大学

## chime2

## chime3

## chime4

## chime5

挑战项目数据，这个挑战是对电话、会议、远距离麦克风进行识别。

## cifar

示例代码

图像分类

基于CIFAR-10和CIFAR-100数据集

证明了用nnet3网络进行固定工大小的图像分类

## commonvoice/s5

示例代码

语料集 Mozilla Common Voice corpus v1

2017年12月发布在kaldi

## csj

示例代码，使用evolution strategy

自发性日语语料库

它由日本的重点研究项目“自发语音：语料与处理技术”开发。

它包含650小时语音，750万单词，超过1400位说话者。

## dihard_2018

示例代码

DIHARD挑战赛

说话人识别

## fame

语音识别 & 说话人识别

弗里西语语料库，包括不同地区的广播语音、语言模型、字典。

## farsdat

波斯语语料库

## fisher_callhome_spanish

kaldi示例代码

Fisher西班牙语语料集，LDC采集的163小时电话语音

Callhome西班牙语语料集，120小时电话语音

## fisher_english

Fisher英语语料集，双声道电话语音

## fisher_swbd/s5

联合Fisher和switchboard数据集

## formosa

示例代码

台湾语语料集

国立台北工业大学

## gale_arabic

参与全球自动语言开发（GALE）计划中的阿拉伯语音数据

由LDC（语言数据协会）开发的包含200小时阿拉伯广播节目交谈对话的语音数据。

数据分为对话式和报告式。

包括了基于音素的语音识别示例和基于字形的语音识别示例（包括nnet3和chain模型）

QCRI，卡塔尔计算研究所

## gale_mandarin

LDC2013S08，GALE第二阶段中文广播新闻语音集

126小时，由LDC 和香港大学采集。

## gp

globalphone语料集：全球语音：19种不同语言，每种15-20小时的语音。

Idiap 研究院

## heroico/s5

kaldi示例代码

针对于西班牙语的Heroico语料集LDC2006S37

## hkust

普通话语音识别

中文电话数据集

香港大学

## hub4_english/s5

示例代码

数据集 English Broadcast News (HUB4) corpus

LDC*

## hub4_spanish

示例代码

HUB4西班牙语料集

## iam

示例代码

基于IAM数据集的手写识别

## iban

伊班语Iban数据集

## ifnenit

示例代码

手写识别项目

```
Arabic IFN/ENIT数据集
```

## librispeech

语料集，1000小时，从有声读物中抽取的，带有一些口音。

免费下载

数据集由Vassil Panayotov准备而成

## lre

kaldi最初发布的说话人识别、语种识别的示例

使用的数据集是SRE 2008 training set:  LDC2011S05

## lre07

对2007 NIST 语种识别评测的示例代码

## madcat_ar	

多语种自动文本分类分析与翻译数据集MADCAT (Multilingual Automatic Document Classification Analysis and Translation)

阿拉伯语料集(LDC2012T15, LDC2013T09, LDC2013T15)适用于手写识别

数据集包含新闻相关段落和博客的摘要。每个页面的xml文件提供了行分段和分词信息，还提供了页面的写作条件(写作风格、速度、细心)。它是一个大的数据集，总共有42k页的图像和750k (600k训练，75k开发，75k eval)行图像和305个作者。主要文本是阿拉伯语，但也包含英文字母和数字。数据集包含约95k个独特的单词和160个独特的字符。该数据集已在2010年和2013年的NIST (Openhart arab large vocabulary unconstrained text recognition competition)评比(可能有不同的分割)中用于行级识别任务。在本次比赛中，获得了16.1%的WER用于行级识别。

## madcat_zh

MADCAT中文手写识别数据集LDC2014T13

示例代码

## mini_librispeech/s5

使用到了<http://www.openslr.org/11/>语言模型

数据集：<http://www.openslr.org/31/>Mini LibriSpeech ASR corpus

该数据集是librispeech的子集，用于regression test。

## multi_en/s5

WIP英语大词汇量连续语音识别LVCSR示例代码

使用到的数据集有：

【Fisher (1761 hours)
   Switchboard (317 hours)
   WSJ (81 hours)
    HUB4 (1996 & 1997) English Broadcast News (75 + 72 hours)
   TED-LIUM (118 hours)
   Librispeech (960 hours)】

This recipe was developed by Allen Guo (ICSI), Korbinian Riedhammer (Remeeting) and Xiaohui Zhang (JHU).



## ptb

Penn Treebank corpus

该示例代码仅提供训练语言模型的部分

## reverb

示例代码

REVERB挑战赛（REverberant Voice Enhancement and Recognition Benchmark）混响语音增强和识别基准

## rimes

手写识别

rimes法语手写识别数据集

由A2iA创建

## rm

DARPA Resource Management corpus，LDC93S3A，语音识别。

## sitw

示例代码

Wild Speaker Recognition挑战赛

SITW数据集

## spanish_dimex100

dimex100语料集

墨西哥西班牙语

100位说话人，6000句话，单通道44.1kHz，16bit，3种层级的标注

墨西哥自治大学

2019.4.24号提交的

## sprakbanken

语料集sprakbanken corpus

多语言：瑞典，挪威，丹麦语

该代码使用的是丹麦语

## sprakbanken_swe

该代码使用的是其中的瑞典语

## sre08

说话人识别，不包括语音识别

sre 2008 数据集

## sre10

sre 2010数据集

## sre16

sre 2016数据集

## svhn

示例代码

图像分类

SVHN (Street View House Numbers) dataset

使用nnet3进行试验

## swahili

swahili数据集

## swbd

基于switchboard语料集的示例

switchboard语料集由DARPA赞助，德州仪器公司研发。1993年发布。

## tedlium

TED-LIUM语料库收录了TED演讲英语语音数据。由 缅因大学信息实验室(LIUM)Laboratoire d’Informatique de l’Université du Maine (LIUM) 发布，452小时，21G，数据集是CC BY-NC-ND 3.0许可。（脚本中说明部分可参考其表达）

## thchs30

清华CSLT

## tidigits

tidigits数据集 LDC93S10

## timit

LDC93S1，最初的语料库。数据集有德州仪器TI公司录制，由麻省理工MTI转录，由NIST出版为CD-ROM。

## tunisian_msa/s5

对阿拉伯语的示例代码

语料集Tunisian_MSA

## uw3

OCR示例代码

UW3数据集

## voxceleb

说话人识别

数据集VoxCeleb1 & VoxCeleb2

## voxforge

开源语音数据收集项目，可以免费得到语音库

## vystadial_cz

免费捷克语电话语音语料库，CC-BY-SA 3.0许可。

数据集的采集和发布由捷克共和国教育部、青年部和体育部根据LK11221授权和布拉格查尔斯大学的核心研究经费提供资金支持。

## vystadial_en

英语

## wsj

使用华尔街日报语料库，似乎是所有脚本开始的地方

华尔街日报语料库是DAPRA-CSR大词汇连续语音识别项目引发建立的。

## yesno

用于测试kaldi脚本的小数据集

## yomdle_fa

OCR示例代码

Yomdle和Slam数据集

2018年十月发布

## yomdle_korean

2018年十二月发布

## yomdle_russian

2019年一月发布

## yomdle_tamil

2018年十二月发布

## yomdle_zh

2018年十月发布

## zeroth_korean/s5

示例代码

zeroth项目，免费韩语语料库