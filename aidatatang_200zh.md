[TOC]



# aidatatang_200zh

## 开源数据集

数据集结构如下：

├── corpus
│     ├── dev
│     ├── test
│     └── train
└── transcript
    └── aidatatang_200_zh_transcript.txt

T0055G0002S0500.wav：语音文件

T0055G0002S0500.txt：原始标注文本

T0055G0002S0500.trn：经结巴分词后文本

T0055G0002S0500.metadata：包含说话人信息等的元文件

dev: test: train = 60: 120: 420 (speakers)

【与aishell1数据集相比，缺少词典文件，这里我们采用脚本生成词典的方式。】



## 目录结构

| 文件名    | 说明          | 创建       | 备注                       |
| --------- | ------------- | ---------- | -------------------------- |
| **conf**  | 配置目录      | 手动       | 特征配置文件               |
| **local** | 脚本目录      | 手动       | 特定工程所需脚本           |
| **steps** | 脚本目录      | 符号链接   | Kaldi提供的数据处理工具    |
| **utils** | 脚本目录      | 符号链接   | Kaldi提供的模型工具        |
| cmd.sh    | 硬件配置      | 拷贝要检查 | 内存和单机集群配置         |
| path.sh   | 环境变量配置  | 拷贝       | 导入环境变量               |
| run.sh    | 总脚本        | 手动       | 总工程运行脚本             |
| README    | 示例介绍      | 可读       | 示例所需数据集             |
| RESULTS   | 实验结果(CER) | 生成       | 罗列所有训练模型的识别结果 |

通过运行run.sh后，会有几个子目录出现，分别是：

data：存放训练集开发集测试集等数据、映射文件、词典、语言模型。

exp：输出目录，模型的生成，训练的结果都在这里。

mfcc：存放语音数据原始的特征文件。

mfcc_perturbed：存放变速后语音数据的特征文件。

mfcc_perturbed_hires：存放变速后语音数据的高分辨率特征文件。

## conf

./conf/
├── cmu2pinyin
├── decode.config
├── g2p_model
├── mfcc.conf
├── mfcc_hires.conf
├── online_cmvn.conf
├── online_pitch.conf
├── pinyin2cmu
├── pinyin_initial
└── pitch.conf

0 directories, 10 files

s5/conf是一些训练用到的配置文件。

此部分借鉴了hkust和aishell1，其中hkust的音频采样率为8kHz，所以需要在此基础上加以修改。

### mfcc.conf

```shell
--use-energy=false   # only non-default option.
--sample-frequency=16000
```

### mfcc_hires.conf

```shell
# config for high-resolution MFCC features, intended for neural network training.
# Note: we keep all cepstra, so it has the same info as filterbank features,
# but MFCC is more easily compressible (because less correlated) which is why
# we prefer this method.
--use-energy=false   # use average of log energy, not energy.
--sample-frequency=16000 #  Switchboard is sampled at 8kHz
--num-mel-bins=40     # similar to Google's setup.
--num-ceps=40     # there is no dimensionality reduction.
--low-freq=40    # low cutoff frequency for mel bins
--high-freq=-200 # high cutoff frequently, relative to Nyquist of 8000 (=3800)
```

### pitch.conf

```shell
-sample-frequency=16000
```

### online_pitch.conf

```shell
--sample-frequency=16000
--simulate-first-pass-online=true
--normalization-right-context=25
--frames-per-chunk=10
```

### online_cmvn.sonf

```shell
# configuration file for apply-cmvn-online, used when invoking online2-wav-nnet3-latgen-faster.
```

### decode.conf

```shell
beam=11.0 # beam for decoding.  Was 13.0 in the scripts.
first_beam=8.0 # beam for 1st-pass decoding in SAT.
```

### pinyin_initial

```shell
B
C
CH
D
F
G
H
J
K
L
M
N
P
Q
R
S
SH
T
W
X
Y
Z
ZH
```

### pinyin2cmu

```shell
A AA
AI AY
AN AE N
ANG AE NG
AO AW
B B
CH CH
C T S
D D
E ER
EI EY
EN AH N
ENG AH NG
ER AA R
F F
G G
H HH
IA IY AA
IANG IY AE NG
IAN IY AE N
IAO IY AW
IE IY EH
I IY
ING IY NG
IN IY N
IONG IY UH NG
IU IY UH
J J
K K
L L
M M
N N
O AO
ONG UH NG
OU OW
P P
Q Q
R R
SH SH
S S
T T
UAI UW AY
UANG UW AE NG
UAN UW AE N
UA UW AA
UI UW IY
UN UW AH N
UO UW AO
U UW
UE IY EH
VE IY EH
V IY UW
VN IY N
W W
X X
Y Y
ZH JH
Z Z
```

### cmu2pinyin

```shell
AA A
AE A
AH A
AO UO
AW U
AY AI
B B
CH CH
D D
DH S I
EH AI
ER E
EY AI
F F
G G
HH H
IH I
IY I
JH ZH
K K
L L
M M
N N
NG N
OW UO
OY UO
P P
R R
S S
SH SH
T T
TH S
UH U
UW U
V W
W W
Y Y
Z Z
ZH X
```



## local

./local/
├── aidatatang_data_prep.sh
├── aidatatang_format_data.sh
├── aidatatang_prepare_dict.sh
├── aidatatang_train_lms2.sh
├── chain
│   ├── run_tdnn.sh -> tuning/run_tdnn_1a.sh
│   └── tuning
│       ├── run_tdnn_1a.sh
│       └── run_tdnn_2a.sh
├── create_oov_char_lexicon.pl
├── download_and_untar.sh
├── nnet3
│   ├── run_ivector_common.sh
│   ├── run_tdnn.sh -> tuning/run_tdnn_1a.sh
│   └── tuning
│       ├── run_tdnn_1a.sh
│       └── run_tdnn_2a.sh
├── score.sh
├── wer_hyp_filter
├── wer_output_filter
└── wer_ref_filter

4 directories, 19 files

s5/{local, steps, utils}里面则是run.sh所要用到的一些脚本文件。



### download_and_untar.sh

在aishell1的基础上做了改动。

```shell
#!/bin/bash

# Copyright   2014  Johns Hopkins University (author: Daniel Povey)
#             2017  Xingyu Na
# Apache 2.0

remove_archive=false

if [ "$1" == --remove-archive ]; then
  remove_archive=true
  shift
fi

if [ $# -ne 3 ]; then
  echo "Usage: $0 [--remove-archive] <data-base> <url-base> <corpus-part>"
  echo "e.g.: $0 /export/a05/xna/data www.openslr.org/resources/62 aidatatang_200zh"
  echo "With --remove-archive it will remove the archive after successfully un-tarring it."
  echo "<corpus-part> can be one of: aidatatang_200zh."
fi

data=$1
url=$2
part=$3

if [ ! -d "$data" ]; then
  echo "$0: no such directory $data"
  exit 1;
fi

part_ok=false
list="aidatatang_200zh"
for x in $list; do
  if [ "$part" == $x ]; then part_ok=true; fi
done
if ! $part_ok; then
  echo "$0: expected <corpus-part> to be one of $list, but got '$part'"
  exit 1;
fi

if [ -z "$url" ]; then
  echo "$0: empty URL base."
  exit 1;
fi

if [ -f $data/$part/.complete ]; then
  echo "$0: data part $part was already successfully extracted, nothing to do."
  exit 0;
fi

# sizes of the archive files in bytes.
sizes="18756983399"

if [ -f $data/$part.tgz ]; then
  size=$(/bin/ls -l $data/$part.tgz | awk '{print $5}')
  size_ok=false
  for s in $sizes; do if [ $s == $size ]; then size_ok=true; fi; done
  if ! $size_ok; then
    echo "$0: removing existing file $data/$part.tgz because its size in bytes $size"
    echo "does not equal the size of one of the archives."
    rm $data/$part.gz
  else
    echo "$data/$part.tgz exists and appears to be complete."
  fi
fi

if [ ! -f $data/$part.tgz ]; then
  if ! which wget >/dev/null; then
    echo "$0: wget is not installed."
    exit 1;
  fi
  full_url=$url/$part.tgz
  echo "$0: downloading data from $full_url.  This may take some time, please be patient."

  cd $data
  if ! wget --no-check-certificate $full_url; then
    echo "$0: error executing wget $full_url"
    exit 1;
  fi
fi

cd $data

if ! tar -xvzf $part.tgz; then
  echo "$0: error un-tarring archive $data/$part.tgz"
  exit 1;
fi

touch $data/$part/.complete

if [ $part == "aidatatang_200zh" ]; then
  cd $data/$part/corpus
  for set in {dev test train};do
    cd $set
    for wav in ./*.tar.gz; do
      echo "Extracting wav from $wav"
      tar -zxf $wav && rm $wav
    done
    cd ../
  done
fi

echo "$0: Successfully downloaded and un-tarred $data/$part.tgz"

if $remove_archive; then
  echo "$0: removing $data/$part.tgz file since --remove-archive option was supplied."
  rm $data/$part.tgz
fi

exit 0;
```



### aidatatang_data_prep.sh

```shell
#!/bin/bash

# Copyright 2017 Xingyu Na
# Apache 2.0

. ./path.sh || exit 1;

if [ $# != 2 ]; then
  echo "Usage: $0 <corpus-path> <text-path>"
  echo " $0 /export/a05/xna/data/data_aidatatang_200zh/corpus /export/a05/xna/data/data_aidatatang_200zh/transcript"
  exit 1;
fi

aidatatang_audio_dir=$1
aidatatang_text=$2/aidatatang_200_zh_transcript.txt

train_dir=data/local/train
dev_dir=data/local/dev
test_dir=data/local/test
tmp_dir=data/local/tmp

mkdir -p $train_dir
mkdir -p $dev_dir
mkdir -p $test_dir
mkdir -p $tmp_dir

# data directory check
if [ ! -d $aidatatang_audio_dir ] || [ ! -f $aidatatang_text ]; then
  echo "Error: $0 requires two directory arguments"
  exit 1;
fi

# find wav audio file for train, dev and test resp.
find $aidatatang_audio_dir -iname "*.wav" > $tmp_dir/wav.flist
n=`cat $tmp_dir/wav.flist | wc -l`
[ $n -ne 237265 ] && \
  echo Warning: expected 237265 data files, found $n

grep -i "corpus/train" $tmp_dir/wav.flist > $train_dir/wav.flist || exit 1;
grep -i "corpus/dev" $tmp_dir/wav.flist > $dev_dir/wav.flist || exit 1;
grep -i "corpus/test" $tmp_dir/wav.flist > $test_dir/wav.flist || exit 1;

rm -r $tmp_dir

# Transcriptions preparation
for dir in $train_dir $dev_dir $test_dir; do
  echo Preparing $dir transcriptions
  sed -e 's/\.wav//' $dir/wav.flist | awk -F '/' '{print $NF}' > $dir/utt.list
  sed -e 's/\.wav//' $dir/wav.flist | awk -F '/' '{i=NF-1;printf("%s %s\n",$NF,$i)}' > $dir/utt2spk_all
  paste -d' ' $dir/utt.list $dir/wav.flist > $dir/wav.scp_all
  utils/filter_scp.pl -f 1 $dir/utt.list $aidatatang_text > $dir/transcripts.txt
  awk '{print $1}' $dir/transcripts.txt > $dir/utt.list
  utils/filter_scp.pl -f 1 $dir/utt.list $dir/utt2spk_all | sort -u > $dir/utt2spk
  utils/filter_scp.pl -f 1 $dir/utt.list $dir/wav.scp_all | sort -u > $dir/wav.scp
  sort -u $dir/transcripts.txt > $dir/text
  utils/utt2spk_to_spk2utt.pl $dir/utt2spk > $dir/spk2utt
done

mkdir -p data/train data/dev data/test

for f in spk2utt utt2spk wav.scp text; do
  cp $train_dir/$f data/train/$f || exit 1;
  cp $dev_dir/$f data/dev/$f || exit 1;
  cp $test_dir/$f data/test/$f || exit 1;
done

echo "$0: aidatatang_200zh data preparation succeeded"
exit 0;
```



### aidatatang_prepare_dict.sh

```shell
#!/bin/bash
#Copyright 2016 LeSpeech (Author: Xingyu Na)

# prepare dictionary for aidatatang
# it is done for English and Chinese separately,
# For English, we use CMU dictionary, and Sequitur G2P
# for OOVs, while all englist phone set will concert to Chinese
# phone set at the end. For Chinese, we use an online dictionary,
# for OOV, we just produce pronunciation using Charactrt Mapping.

. ./path.sh

[ $# != 0 ] && echo "Usage: $0" && exit 1;

train_dir=data/local/train
dev_dir=data/local/dev
test_dir=data/local/test
dict_dir=data/local/dict
mkdir -p $dict_dir
mkdir -p $dict_dir/lexicon-{en,ch}

# extract full vocabulary
cat $train_dir/text $dev_dir/text $test_dir/text | awk '{for (i = 2; i <= NF; i++) print $i}' |\
  perl -ape 's/ /\n/g;' | sort -u | grep -v '\[LAUGHTER\]' | grep -v '\[NOISE\]' |\
  grep -v '\[VOCALIZED-NOISE\]' > $dict_dir/words.txt || exit 1;

# split into English and Chinese
cat $dict_dir/words.txt | grep '[a-zA-Z]' > $dict_dir/lexicon-en/words-en.txt || exit 1;
cat $dict_dir/words.txt | grep -v '[a-zA-Z]' > $dict_dir/lexicon-ch/words-ch.txt || exit 1;


##### produce pronunciations for english
if [ ! -f $dict_dir/cmudict/cmudict.0.7a ]; then
  echo "--- Downloading CMU dictionary ..."
  svn co -r 13068 https://svn.code.sf.net/p/cmusphinx/code/trunk/cmudict \
    $dict_dir/cmudict || exit 1;
fi

# format cmudict
echo "--- Striping stress and pronunciation variant markers from cmudict ..."
perl $dict_dir/cmudict/scripts/make_baseform.pl \
  $dict_dir/cmudict/cmudict.0.7a /dev/stdout |\
  sed -e 's:^\([^\s(]\+\)([0-9]\+)\(\s\+\)\(.*\):\1\2\3:' > $dict_dir/cmudict/cmudict-plain.txt || exit 1;

# extract in-vocab lexicon and oov words
echo "--- Searching for English OOV words ..."
awk 'NR==FNR{words[$1]; next;} !($1 in words)' \
  $dict_dir/cmudict/cmudict-plain.txt $dict_dir/lexicon-en/words-en.txt |\
  egrep -v '<.?s>' > $dict_dir/lexicon-en/words-en-oov.txt || exit 1;

awk 'NR==FNR{words[$1]; next;} ($1 in words)' \
  $dict_dir/lexicon-en/words-en.txt $dict_dir/cmudict/cmudict-plain.txt |\
  egrep -v '<.?s>' > $dict_dir/lexicon-en/lexicon-en-iv.txt || exit 1;

wc -l $dict_dir/lexicon-en/words-en-oov.txt
wc -l $dict_dir/lexicon-en/lexicon-en-iv.txt

# setup g2p and generate oov lexicon
if [ ! -f conf/g2p_model ]; then
  echo "--- Downloading a pre-trained Sequitur G2P model ..."
  wget http://sourceforge.net/projects/kaldi/files/sequitur-model4 -O conf/g2p_model
  if [ ! -f conf/g2p_model ]; then
    echo "Failed to download the g2p model!"
    exit 1
  fi
fi

echo "--- Preparing pronunciations for OOV words ..."
g2p=`which g2p.py`
if [ ! -x $g2p ]; then
  echo "g2p.py is not found. Checkout tools/extras/install_sequitur.sh."
  exit 1
fi
g2p.py --model=conf/g2p_model --apply $dict_dir/lexicon-en/words-en-oov.txt \
  > $dict_dir/lexicon-en/lexicon-en-oov.txt || exit 1;

# merge in-vocab and oov lexicon
cat $dict_dir/lexicon-en/lexicon-en-oov.txt $dict_dir/lexicon-en/lexicon-en-iv.txt |\
  sort > $dict_dir/lexicon-en/lexicon-en-phn.txt || exit 1;

# convert cmu phoneme to pinyin phonenme
mkdir $dict_dir/map
cat conf/cmu2pinyin | awk '{print $1;}' | sort -u > $dict_dir/map/cmu || exit 1;
cat conf/pinyin2cmu | awk -v cmu=$dict_dir/map/cmu \
  'BEGIN{while((getline<cmu)) dict[$1] = 1;}
   {for (i = 2; i <=NF; i++) if (dict[$i]) print $i;}' | sort -u > $dict_dir/map/cmu-used || exit 1;
cat $dict_dir/map/cmu | awk -v cmu=$dict_dir/map/cmu-used \
  'BEGIN{while((getline<cmu)) dict[$1] = 1;}
   {if (!dict[$1]) print $1;}' > $dict_dir/map/cmu-not-used || exit 1;

awk 'NR==FNR{words[$1]; next;} ($1 in words)' \
  $dict_dir/map/cmu-not-used conf/cmu2pinyin |\
  egrep -v '<.?s>' > $dict_dir/map/cmu-py || exit 1;

cat $dict_dir/map/cmu-py | \
  perl -e '
  open(MAPS, $ARGV[0]) or die("could not open map file");
  my %py2ph;
  foreach $line (<MAPS>) {
    @A = split(" ", $line);
    $py = shift(@A);
    $py2ph{$py} = [@A];
  }
  my @entry;
  while (<STDIN>) {
    @A = split(" ", $_);
    @entry = ();
    $W = shift(@A);
    push(@entry, $W);
    for($i = 0; $i < @A; $i++) { push(@entry, @{$py2ph{$A[$i]}}); }
    print "@entry";
    print "\n";
  }
' conf/pinyin2cmu > $dict_dir/map/cmu-cmu || exit 1;

cat $dict_dir/lexicon-en/lexicon-en-phn.txt | \
  perl -e '
  open(MAPS, $ARGV[0]) or die("could not open map file");
  my %py2ph;
  foreach $line (<MAPS>) {
    @A = split(" ", $line);
    $py = shift(@A);
    $py2ph{$py} = [@A];
  }
  my @entry;
  while (<STDIN>) {
    @A = split(" ", $_);
    @entry = ();
    $W = shift(@A);
    push(@entry, $W);
    for($i = 0; $i < @A; $i++) {
      if (exists $py2ph{$A[$i]}) { push(@entry, @{$py2ph{$A[$i]}}); }
      else {push(@entry, $A[$i])};
    }
    print "@entry";
    print "\n";
  }
' $dict_dir/map/cmu-cmu > $dict_dir/lexicon-en/lexicon-en.txt || exit 1;


##### produce pronunciations for chinese
if [ ! -f $dict_dir/cedict/cedict_1_0_ts_utf-8_mdbg.txt ]; then
  echo "------------- Downloading cedit dictionary ---------------"
  mkdir -p $dict_dir/cedict
  wget -P $dict_dir/cedict http://www.mdbg.net/chindict/export/cedict/cedict_1_0_ts_utf-8_mdbg.txt.gz
  gunzip $dict_dir/cedict/cedict_1_0_ts_utf-8_mdbg.txt.gz
fi

cat $dict_dir/cedict/cedict_1_0_ts_utf-8_mdbg.txt | grep -v '#' | awk -F '/' '{print $1}' |\
 perl -e '
  while (<STDIN>) {
    @A = split(" ", $_);
    print $A[1];
    for($n = 2; $n < @A; $n++) {
      $A[$n] =~ s:\[?([a-zA-Z0-9\:]+)\]?:$1:;
      $tmp = uc($A[$n]);
      print " $tmp";
    }
    print "\n";
  }
 ' | sort -k1 > $dict_dir/cedict/ch-dict.txt || exit 1;

echo "--- Searching for Chinese OOV words ..."
awk 'NR==FNR{words[$1]; next;} !($1 in words)' \
  $dict_dir/cedict/ch-dict.txt $dict_dir/lexicon-ch/words-ch.txt |\
  egrep -v '<.?s>' > $dict_dir/lexicon-ch/words-ch-oov.txt || exit 1;

awk 'NR==FNR{words[$1]; next;} ($1 in words)' \
  $dict_dir/lexicon-ch/words-ch.txt $dict_dir/cedict/ch-dict.txt |\
  egrep -v '<.?s>' > $dict_dir/lexicon-ch/lexicon-ch-iv.txt || exit 1;

wc -l $dict_dir/lexicon-ch/words-ch-oov.txt
wc -l $dict_dir/lexicon-ch/lexicon-ch-iv.txt


# validate Chinese dictionary and compose a char-based
# dictionary in order to get OOV pronunciations
cat $dict_dir/cedict/ch-dict.txt |\
  perl -e '
  use encoding utf8;
  while (<STDIN>) {
    @A = split(" ", $_);
    $word_len = length($A[0]);
    $proun_len = @A - 1 ;
    if ($word_len == $proun_len) {print $_;}
  }
  ' > $dict_dir/cedict/ch-dict-1.txt || exit 1;

# extract chars
cat $dict_dir/cedict/ch-dict-1.txt | awk '{print $1}' |\
  perl -e '
  use encoding utf8;
  while (<STDIN>) {
    @A = split(" ", $_);
    @chars = split("", $A[0]);
    foreach (@chars) {
      print "$_\n";
    }
  }
  ' | grep -v '^$' > $dict_dir/lexicon-ch/ch-char.txt || exit 1;

# extract individual pinyins
cat $dict_dir/cedict/ch-dict-1.txt |\
  awk '{for(i=2; i<=NF; i++) print $i}' |\
  perl -ape 's/ /\n/g;' > $dict_dir/lexicon-ch/ch-char-pinyin.txt || exit 1;

# first make sure number of characters and pinyins
# are equal, so that a char-based dictionary can
# be composed.
nchars=`wc -l < $dict_dir/lexicon-ch/ch-char.txt`
npinyin=`wc -l < $dict_dir/lexicon-ch/ch-char-pinyin.txt`
if [ $nchars -ne $npinyin ]; then
  echo "Found $nchars chars and $npinyin pinyin. Please check!"
  exit 1
fi

paste $dict_dir/lexicon-ch/ch-char.txt $dict_dir/lexicon-ch/ch-char-pinyin.txt |\
  sort -u > $dict_dir/lexicon-ch/ch-char-dict.txt || exit 1;

# create a multiple pronunciation dictionary
cat $dict_dir/lexicon-ch/ch-char-dict.txt |\
  perl -e '
  my $prev = "";
  my $out_line = "";
  while (<STDIN>) {
    @A = split(" ", $_);
    $cur = $A[0];
    $cur_py = $A[1];
    #print length($prev);
    if (length($prev) == 0) { $out_line = $_; chomp($out_line);}
    if (length($prev)>0 && $cur ne $prev) { print $out_line; print "\n"; $out_line = $_; chomp($out_line);}
    if (length($prev)>0 && $cur eq $prev) { $out_line = $out_line."/"."$cur_py";}
    $prev = $cur;
  }
  print $out_line;
  ' >  $dict_dir/lexicon-ch/ch-char-dict-mp.txt || exit 1;

# get lexicon for Chinese OOV words
local/create_oov_char_lexicon.pl $dict_dir/lexicon-ch/ch-char-dict-mp.txt \
  $dict_dir/lexicon-ch/words-ch-oov.txt > $dict_dir/lexicon-ch/lexicon-ch-oov.txt || exit 1;

# seperate multiple prons for Chinese OOV lexicon
cat $dict_dir/lexicon-ch/lexicon-ch-oov.txt |\
  perl -e '
  my @entry;
  my @entry1;
  while (<STDIN>) {
    @A = split(" ", $_);
    @entry = ();
    push(@entry, $A[0]);
    for($i = 1; $i < @A; $i++ ) {
      @py = split("/", $A[$i]);
      @entry1 = @entry;
      @entry = ();
      for ($j = 0; $j < @entry1; $j++) {
        for ($k = 0; $k < @py; $k++) {
          $tmp = $entry1[$j]." ".$py[$k];
          push(@entry, $tmp);
        }
      }
    }
    for ($i = 0; $i < @entry; $i++) {
      print $entry[$i];
      print "\n";
    }
  }
  ' > $dict_dir/lexicon-ch/lexicon-ch-oov-mp.txt || exit 1;

# compose IV and OOV lexicons for Chinese
cat $dict_dir/lexicon-ch/lexicon-ch-oov-mp.txt $dict_dir/lexicon-ch/lexicon-ch-iv.txt |\
  awk '{if (NF > 1 && $2 ~ /[A-Za-z0-9]+/) print $0;}' > $dict_dir/lexicon-ch/lexicon-ch.txt || exit 1;

# convert Chinese pinyin to CMU format
cat $dict_dir/lexicon-ch/lexicon-ch.txt | sed -e 's/U:/V/g' | sed -e 's/ R\([0-9]\)/ ER\1/g'|\
  utils/pinyin_map.pl conf/pinyin2cmu > $dict_dir/lexicon-ch/lexicon-ch-cmu.txt || exit 1;

# combine English and Chinese lexicons
cat $dict_dir/lexicon-en/lexicon-en.txt $dict_dir/lexicon-ch/lexicon-ch-cmu.txt |\
  sort -u > $dict_dir/lexicon1.txt || exit 1;

cat $dict_dir/lexicon1.txt | awk '{ for(n=2;n<=NF;n++){ phones[$n] = 1; }} END{for (p in phones) print p;}'| \
  sort -u |\
  perl -e '
  my %ph_cl;
  while (<STDIN>) {
    $phone = $_;
    chomp($phone);
    chomp($_);
    $phone =~ s:([A-Z]+)[0-9]:$1:;
    if (exists $ph_cl{$phone}) { push(@{$ph_cl{$phone}}, $_)  }
    else { $ph_cl{$phone} = [$_]; }
  }
  foreach $key ( keys %ph_cl ) {
     print "@{ $ph_cl{$key} }\n"
  }
  ' | sort -k1 > $dict_dir/nonsilence_phones.txt  || exit 1;

( echo SIL; echo SPN; echo NSN; echo LAU ) > $dict_dir/silence_phones.txt

echo SIL > $dict_dir/optional_silence.txt

# No "extra questions" in the input to this setup, as we don't
# have stress or tone

cat $dict_dir/silence_phones.txt| awk '{printf("%s ", $1);} END{printf "\n";}' > $dict_dir/extra_questions.txt || exit 1;
cat $dict_dir/nonsilence_phones.txt | perl -e 'while(<>){ foreach $p (split(" ", $_)) {
  $p =~ m:^([^\d]+)(\d*)$: || die "Bad phone $_"; $q{$2} .= "$p "; } } foreach $l (values %q) {print "$l\n";}' \
 >> $dict_dir/extra_questions.txt || exit 1;

# Add to the lexicon the silences, noises etc.
(echo '!SIL SIL'; echo '[VOCALIZED-NOISE] SPN'; echo '[NOISE] NSN'; echo '[LAUGHTER] LAU';
 echo '<UNK> SPN' ) | \
 cat - $dict_dir/lexicon1.txt  > $dict_dir/lexicon.txt || exit 1;

echo "$0: aidatatang_200zh dict preparation succeeded"
exit 0;
```



### create_oov_char_lexicon.pl

```perl
#!/usr/bin/env perl
# Copyright 2016 Alibaba Robotics Corp. (Author: Xingyu Na)
#
# A script for char-based Chinese OOV lexicon generation.
#
# Input 1: char-based dictionary, example
# CHAR1 ph1 ph2
# CHAR2 ph3
# CHAR3 ph2 ph4
#
# Input 2: OOV word list, example
# WORD1
# WORD2
# WORD3
#
# where WORD1 is in the format of "CHAR1CHAR2".
#
# Output: OOV lexicon, in the format of normal lexicon

if($#ARGV != 1) {
  print STDERR "usage: perl create_oov_char_lexicon.pl chardict oovwordlist > oovlex\n\n";
  print STDERR "### chardict: a dict in which each line contains the pronunciation of one Chinese char\n";
  print STDERR "### oovwordlist: OOV word list\n";
  print STDERR "### oovlex: output OOV lexicon\n";
  exit;
}

use encoding utf8;
my %prons;
open(DICT, $ARGV[0]) || die("Can't open dict ".$ARGV[0]."\n");
foreach (<DICT>) {
  chomp; @A = split(" ", $_); $prons{$A[0]} = $A[1];
}
close DICT;

open(WORDS, $ARGV[1]) || die("Can't open oov word list ".$ARGV[1]."\n");
while (<WORDS>) {
  chomp;
  print $_;
  @A = split("", $_);
  foreach (@A) {
    print " $prons{$_}";
  }
  print "\n";
}
close WORDS;
```



### aidatatang_train_lms.sh

```shell
#!/bin/bash

# To be run from one directory above this script.

text=data/local/train/text
lexicon=data/local/dict/lexicon.txt

for f in "$text" "$lexicon"; do
  [ ! -f $x ] && echo "$0: No such file $f" && exit 1;
done

# This script takes no arguments.  It assumes you have already run
# aidatatang_data_prep.sh.
# It takes as input the files
#data/local/train/text
#data/local/dict/lexicon.txt
dir=data/local/lm
mkdir -p $dir

export LC_ALL=C # You'll get errors about things being not sorted, if you
                # have a different locale.
kaldi_lm=`which train_lm.sh`
if [ ! -x $kaldi_lm ]; then
  echo "$0: train_lm.sh is not found. That might mean it's not installed"
  echo "$0: or it is not added to PATH"
  echo "$0: Use the script tools/extra/install_kaldi_lm.sh to install it"
  exit 1
fi

cleantext=$dir/text.no_oov

cat $text | awk -v lex=$lexicon 'BEGIN{while((getline<lex) >0){ seen[$1]=1; } }
  {for(n=1; n<=NF;n++) {  if (seen[$n]) { printf("%s ", $n); } else {printf("<UNK> ");} } printf("\n");}' \
  > $cleantext || exit 1;


cat $cleantext | awk '{for(n=2;n<=NF;n++) print $n; }' | sort | uniq -c | \
   sort -nr > $dir/word.counts || exit 1;


# Get counts from acoustic training transcripts, and add  one-count
# for each word in the lexicon (but not silence, we don't want it
# in the LM-- we'll add it optionally later).
cat $cleantext | awk '{for(n=2;n<=NF;n++) print $n; }' | \
  cat - <(grep -w -v '!SIL' $lexicon | awk '{print $1}') | \
   sort | uniq -c | sort -nr > $dir/unigram.counts || exit 1;

# note: we probably won't really make use of <UNK> as there aren't any OOVs
cat $dir/unigram.counts  | awk '{print $2}' | get_word_map.pl "<s>" "</s>" "<UNK>" > $dir/word_map \
   || exit 1;

# note: ignore 1st field of train.txt, it's the utterance-id.
cat $cleantext | awk -v wmap=$dir/word_map 'BEGIN{while((getline<wmap)>0)map[$1]=$2;}
  { for(n=2;n<=NF;n++) { printf map[$n]; if(n<NF){ printf " "; } else { print ""; }}}' | gzip -c >$dir/train.gz \
   || exit 1;

train_lm.sh --arpa --lmtype 3gram-mincount $dir || exit 1;

# LM is small enough that we don't need to prune it (only about 0.7M N-grams).
# Perplexity over 128254.000000 words is 90.446690

# note: output is
# data/local/lm/3gram-mincount/lm_unpruned.gz

exit 0


# From here is some commands to do a baseline with SRILM (assuming
# you have it installed).
heldout_sent=10000 # Don't change this if you want result to be comparable with
    # kaldi_lm results
sdir=$dir/srilm # in case we want to use SRILM to double-check perplexities.
mkdir -p $sdir
cat $cleantext | awk '{for(n=2;n<=NF;n++){ printf $n; if(n<NF) printf " "; else print ""; }}' | \
  head -$heldout_sent > $sdir/heldout
cat $cleantext | awk '{for(n=2;n<=NF;n++){ printf $n; if(n<NF) printf " "; else print ""; }}' | \
  tail -n +$heldout_sent > $sdir/train

cat $dir/word_map | awk '{print $1}' | cat - <(echo "<s>"; echo "</s>" ) > $sdir/wordlist


ngram-count -text $sdir/train -order 3 -limit-vocab -vocab $sdir/wordlist -unk \
  -map-unk "<UNK>" -kndiscount -interpolate -lm $sdir/srilm.o3g.kn.gz
ngram -lm $sdir/srilm.o3g.kn.gz -ppl $sdir/heldout
# 0 zeroprobs, logprob= -250954 ppl= 90.5091 ppl1= 132.482

# Note: perplexity SRILM gives to Kaldi-LM model is same as kaldi-lm reports above.
# Difference in WSJ must have been due to different treatment of <UNK>.
ngram -lm $dir/3gram-mincount/lm_unpruned.gz  -ppl $sdir/heldout
# 0 zeroprobs, logprob= -250913 ppl= 90.4439 ppl1= 132.379
```



### aidatatang_format_data.sh

```shell
#!/bin/bash
#

if [ -f ./path.sh ]; then . ./path.sh; fi

silprob=0.5
mkdir -p data/lang_test data/train data/dev


arpa_lm=data/local/lm/3gram-mincount/lm_unpruned.gz
[ ! -f $arpa_lm ] && echo No such file $arpa_lm && exit 1;

# Copy stuff into its final locations...

for f in spk2utt utt2spk wav.scp text; do
  cp data/local/train/$f data/train/$f || exit 1;
done

for f in spk2utt utt2spk wav.scp text; do
  cp data/local/dev/$f data/dev/$f || exit 1;
done

rm -r data/lang_test
cp -r data/lang data/lang_test

gunzip -c "$arpa_lm" | \
  arpa2fst --disambig-symbol=#0 \
           --read-symbol-table=data/lang_test/words.txt - data/lang_test/G.fst


echo  "Checking how stochastic G is (the first of these numbers should be small):"
fstisstochastic data/lang_test/G.fst

## Check lexicon.
## just have a look and make sure it seems sane.
echo "First few lines of lexicon FST:"
fstprint   --isymbols=data/lang/phones.txt --osymbols=data/lang/words.txt data/lang/L.fst  | head

echo Performing further checks

# Checking that G.fst is determinizable.
fstdeterminize data/lang_test/G.fst /dev/null || echo Error determinizing G.

# Checking that L_disambig.fst is determinizable.
fstdeterminize data/lang_test/L_disambig.fst /dev/null || echo Error determinizing L.

# Checking that disambiguated lexicon times G is determinizable
# Note: we do this with fstdeterminizestar not fstdeterminize, as
# fstdeterminize was taking forever (presumbaly relates to a bug
# in this version of OpenFst that makes determinization slow for
# some case).
fsttablecompose data/lang_test/L_disambig.fst data/lang_test/G.fst | \
   fstdeterminizestar >/dev/null || echo Error

# Checking that LG is stochastic:
fsttablecompose data/lang/L_disambig.fst data/lang_test/G.fst | \
   fstisstochastic || echo LG is not stochastic


echo format_data succeeded.
```



### score.sh

```shell
#!/bin/bash

set -e -o pipefail
set -x
steps/score_kaldi.sh "$@"
steps/scoring/score_kaldi_cer.sh --stage 2 "$@"

echo "$0: Done"
```



### nnet3

#### run_ivector_common.sh

```shell
#!/bin/bash

set -euo pipefail

# This script is modified based on mini_librispeech/s5/local/nnet3/run_ivector_common.sh

# This script is called from local/nnet3/run_tdnn.sh and
# local/chain/run_tdnn.sh (and may eventually be called by more
# scripts).  It contains the common feature preparation and
# iVector-related parts of the script.  See those scripts for examples
# of usage.

stage=0
train_set=train
test_sets="dev test"
gmm=tri5a
online=false
nnet3_affix=

. ./cmd.sh
. ./path.sh
. utils/parse_options.sh

gmm_dir=exp/${gmm}
ali_dir=exp/${gmm}_sp_ali

for f in data/${train_set}/feats.scp ${gmm_dir}/final.mdl; do
  if [ ! -f $f ]; then
    echo "$0: expected file $f to exist"
    exit 1
  fi
done

online_affix=
if [ $online = true ]; then
  online_affix=_online
fi

if [ $stage -le 1 ]; then
  # Although the nnet will be trained by high resolution data, we still have to
  # perturb the normal data to get the alignment _sp stands for speed-perturbed
  echo "$0: preparing directory for low-resolution speed-perturbed data (for alignment)"
  utils/data/perturb_data_dir_speed_3way.sh data/${train_set} data/${train_set}_sp
  echo "$0: making MFCC features for low-resolution speed-perturbed data"
  steps/make_mfcc_pitch.sh --cmd "$train_cmd" --nj 70 data/${train_set}_sp \
    exp/make_mfcc/train_sp mfcc_perturbed || exit 1;
  steps/compute_cmvn_stats.sh data/${train_set}_sp \
    exp/make_mfcc/train_sp mfcc_perturbed || exit 1;
  utils/fix_data_dir.sh data/${train_set}_sp
fi

if [ $stage -le 2 ]; then
  echo "$0: aligning with the perturbed low-resolution data"
  steps/align_fmllr.sh --nj 30 --cmd "$train_cmd" \
    data/${train_set}_sp data/lang $gmm_dir $ali_dir || exit 1
fi

if [ $stage -le 3 ]; then
  # Create high-resolution MFCC features (with 40 cepstra instead of 13).
  # this shows how you can split across multiple file-systems.
  echo "$0: creating high-resolution MFCC features"
  mfccdir=mfcc_perturbed_hires$online_affix
  if [[ $(hostname -f) == *.clsp.jhu.edu ]] && [ ! -d $mfccdir/storage ]; then
    utils/create_split_dir.pl /export/b0{5,6,7,8}/$USER/kaldi-data/mfcc/aidatatang-$(date +'%m_%d_%H_%M')/s5/$mfccdir/storage $mfccdir/storage
  fi

  for datadir in ${train_set}_sp ${test_sets}; do
    utils/copy_data_dir.sh data/$datadir data/${datadir}_hires$online_affix
  done

  # do volume-perturbation on the training data prior to extracting hires
  # features; this helps make trained nnets more invariant to test data volume.
  utils/data/perturb_data_dir_volume.sh data/${train_set}_sp_hires$online_affix || exit 1;

  for datadir in ${train_set}_sp ${test_sets}; do
    steps/make_mfcc_pitch$online_affix.sh --nj 10 --mfcc-config conf/mfcc_hires.conf \
      --cmd "$train_cmd" data/${datadir}_hires$online_affix exp/make_hires/$datadir $mfccdir || exit 1;
    steps/compute_cmvn_stats.sh data/${datadir}_hires$online_affix exp/make_hires/$datadir $mfccdir || exit 1;
    utils/fix_data_dir.sh data/${datadir}_hires$online_affix || exit 1;
    # create MFCC data dir without pitch to extract iVector
    utils/data/limit_feature_dim.sh 0:39 data/${datadir}_hires$online_affix data/${datadir}_hires_nopitch || exit 1;
    steps/compute_cmvn_stats.sh data/${datadir}_hires_nopitch exp/make_hires/$datadir $mfccdir || exit 1;
  done
fi

if [ $stage -le 4 ]; then
  echo "$0: computing a subset of data to train the diagonal UBM."
  # We'll use about a quarter of the data.
  mkdir -p exp/nnet3${nnet3_affix}/diag_ubm
  temp_data_root=exp/nnet3${nnet3_affix}/diag_ubm

  num_utts_total=$(wc -l <data/${train_set}_sp_hires_nopitch/utt2spk)
  num_utts=$[$num_utts_total/4]
  utils/data/subset_data_dir.sh data/${train_set}_sp_hires_nopitch \
     $num_utts ${temp_data_root}/${train_set}_sp_hires_nopitch_subset

  echo "$0: computing a PCA transform from the hires data."
  steps/online/nnet2/get_pca_transform.sh --cmd "$train_cmd" \
      --splice-opts "--left-context=3 --right-context=3" \
      --max-utts 10000 --subsample 2 \
       ${temp_data_root}/${train_set}_sp_hires_nopitch_subset \
       exp/nnet3${nnet3_affix}/pca_transform

  echo "$0: training the diagonal UBM."
  # Use 512 Gaussians in the UBM.
  steps/online/nnet2/train_diag_ubm.sh --cmd "$train_cmd" --nj 30 \
    --num-frames 700000 \
    --num-threads 8 \
    ${temp_data_root}/${train_set}_sp_hires_nopitch_subset 512 \
    exp/nnet3${nnet3_affix}/pca_transform exp/nnet3${nnet3_affix}/diag_ubm
fi

if [ $stage -le 5 ]; then
  # Train the iVector extractor.  Use all of the speed-perturbed data since iVector extractors
  # can be sensitive to the amount of data.  The script defaults to an iVector dimension of
  # 100.
  echo "$0: training the iVector extractor"
  steps/online/nnet2/train_ivector_extractor.sh --cmd "$train_cmd" --nj 10 \
     data/${train_set}_sp_hires_nopitch exp/nnet3${nnet3_affix}/diag_ubm \
     exp/nnet3${nnet3_affix}/extractor || exit 1;
fi

train_set=train_sp

if [ $stage -le 6 ]; then
  # We extract iVectors on the speed-perturbed training data after combining
  # short segments, which will be what we train the system on.  With
  # --utts-per-spk-max 2, the script pairs the utterances into twos, and treats
  # each of these pairs as one speaker; this gives more diversity in iVectors..
  # Note that these are extracted 'online'.

  # note, we don't encode the 'max2' in the name of the ivectordir even though
  # that's the data we extract the ivectors from, as it's still going to be
  # valid for the non-'max2' data, the utterance list is the same.

  ivectordir=exp/nnet3${nnet3_affix}/ivectors_${train_set}
  if [[ $(hostname -f) == *.clsp.jhu.edu ]] && [ ! -d $ivectordir/storage ]; then
    utils/create_split_dir.pl /export/b0{5,6,7,8}/$USER/kaldi-data/ivectors/aidatatang-$(date +'%m_%d_%H_%M')/s5/$ivectordir/storage $ivectordir/storage
  fi


  # having a larger number of speakers is helpful for generalization, and to
  # handle per-utterance decoding well (iVector starts at zero).
  temp_data_root=${ivectordir}
  utils/data/modify_speaker_info.sh --utts-per-spk-max 2 \
    data/${train_set}_hires_nopitch ${temp_data_root}/${train_set}_sp_hires_nopitch_max2
  steps/online/nnet2/extract_ivectors_online.sh --cmd "$train_cmd" --nj 30 \
    ${temp_data_root}/${train_set}_sp_hires_nopitch_max2 \
    exp/nnet3${nnet3_affix}/extractor $ivectordir

  # Also extract iVectors for the test data, but in this case we don't need the speed
  # perturbation (sp).
  for data in $test_sets; do
    steps/online/nnet2/extract_ivectors_online.sh --cmd "$train_cmd" --nj 8 \
      data/${data}_hires_nopitch exp/nnet3${nnet3_affix}/extractor \
      exp/nnet3${nnet3_affix}/ivectors_${data}
  done
fi

exit 0
```



#### run_tdnn_1a.sh

```shell
#!/bin/bash

# This script is based on swbd/s5c/local/nnet3/run_tdnn.sh

# this is the standard "tdnn" system, built in nnet3; it's what we use to
# call multi-splice.

# At this script level we don't support not running on GPU, as it would be painfully slow.
# If you want to run without GPU you'd have to call train_tdnn.sh with --gpu false,
# --num-threads 16 and --minibatch-size 128.
set -e

stage=8
train_stage=-10
affix=
common_egs_dir=

# training options
initial_effective_lrate=0.0015
final_effective_lrate=0.00015
num_epochs=4
num_jobs_initial=2
num_jobs_final=12
remove_egs=true

# feature options
use_ivectors=true

# End configuration section.

. ./cmd.sh
. ./path.sh
. ./utils/parse_options.sh

if ! cuda-compiled; then
  cat <<EOF && exit 1
This script is intended to be used with GPUs but you have not compiled Kaldi with CUDA
If you want to use GPUs (and have them), go to src/, and configure and make on a machine
where "nvcc" is installed.
EOF
fi

dir=exp/nnet3/tdnn_sp${affix:+_$affix}
gmm_dir=exp/tri5a
train_set=train_sp
ali_dir=${gmm_dir}_sp_ali
graph_dir=$gmm_dir/graph

local/nnet3/run_ivector_common.sh --stage $stage || exit 1;

if [ $stage -le 7 ]; then
  echo "$0: creating neural net configs";
 
  ivector_dim=$(feat-to-dim scp:exp/nnet3/ivectors_${train_set}/ivector_online.scp - || exit 1;)
  feat_dim=$(feat-to-dim scp:data/${train_set}_hires/feats.scp - || exit 1;)
  num_targets=$(tree-info $ali_dir/tree |grep num-pdfs|awk '{print $2}')

  mkdir -p $dir/configs
  cat <<EOF > $dir/configs/network.xconfig
  input dim=$ivector_dim name=ivector
  input dim=$feat_dim name=input

  # please note that it is important to have input layer with the name=input
  # as the layer immediately preceding the fixed-affine-layer to enable
  # the use of short notation for the descriptor
  fixed-affine-layer name=lda input=Append(-2,-1,0,1,2,ReplaceIndex(ivector, t, 0)) affine-transform-file=$dir/configs/lda.mat

  # the first splicing is moved before the lda layer, so no splicing here
  relu-batchnorm-layer name=tdnn1 dim=850
  relu-batchnorm-layer name=tdnn2 dim=850 input=Append(-1,0,2)
  relu-batchnorm-layer name=tdnn3 dim=850 input=Append(-3,0,3)
  relu-batchnorm-layer name=tdnn4 dim=850 input=Append(-7,0,2)
  relu-batchnorm-layer name=tdnn5 dim=850 input=Append(-3,0,3)
  relu-batchnorm-layer name=tdnn6 dim=850
  output-layer name=output input=tdnn6 dim=$num_targets max-change=1.5
EOF
  steps/nnet3/xconfig_to_configs.py --xconfig-file $dir/configs/network.xconfig --config-dir $dir/configs/
fi

if [ $stage -le 8 ]; then
  if [[ $(hostname -f) == *.clsp.jhu.edu ]] && [ ! -d $dir/egs/storage ]; then
    utils/create_split_dir.pl \
     /export/b0{5,6,7,8}/$USER/kaldi-data/egs/aidatatang-$(date +'%m_%d_%H_%M')/s5/$dir/egs/storage $dir/egs/storage
  fi

  steps/nnet3/train_dnn.py --stage=$train_stage \
    --cmd="$decode_cmd" \
    --feat.online-ivector-dir exp/nnet3/ivectors_${train_set} \
    --feat.cmvn-opts="--norm-means=false --norm-vars=false" \
    --trainer.num-epochs $num_epochs \
    --trainer.optimization.num-jobs-initial $num_jobs_initial \
    --trainer.optimization.num-jobs-final $num_jobs_final \
    --trainer.optimization.initial-effective-lrate $initial_effective_lrate \
    --trainer.optimization.final-effective-lrate $final_effective_lrate \
    --egs.dir "$common_egs_dir" \
    --cleanup.remove-egs $remove_egs \
    --cleanup.preserve-model-interval 500 \
    --use-gpu wait \
    --feat-dir=data/${train_set}_hires \
    --ali-dir $ali_dir \
    --lang data/lang \
    --reporting.email="$reporting_email" \
    --dir=$dir  || exit 1;
fi

if [ $stage -le 9 ]; then
  # this version of the decoding treats each utterance separately
  # without carrying forward speaker information.
  for decode_set in dev test; do
    num_jobs=`cat data/${decode_set}_hires/utt2spk|cut -d' ' -f2|sort -u|wc -l`
    decode_dir=${dir}/decode_$decode_set
    steps/nnet3/decode.sh --nj $num_jobs --cmd "$decode_cmd" \
       --online-ivector-dir exp/nnet3/ivectors_${decode_set} \
       $graph_dir data/${decode_set}_hires $decode_dir || exit 1;
  done
fi

wait;
exit 0;
```



#### run_tdnn_2a.sh

```shell
#!/bin/bash

# This script is based on aishell/s5/local/nnet3/tuning/run_tdnn_1a.sh

# In this script, the neural network in trained based on hires mfcc and online pitch.
# The online pitch setup requires a online_pitch.conf in the conf dir for both training
# and testing.

set -e

stage=7
train_stage=-10
affix=
common_egs_dir=

# training options
initial_effective_lrate=0.0015
final_effective_lrate=0.00015
num_epochs=4
num_jobs_initial=2
num_jobs_final=12
remove_egs=true

# feature options
use_ivectors=true

# End configuration section.

. ./cmd.sh
. ./path.sh
. ./utils/parse_options.sh

if ! cuda-compiled; then
  cat <<EOF && exit 1
This script is intended to be used with GPUs but you have not compiled Kaldi with CUDA
If you want to use GPUs (and have them), go to src/, and configure and make on a machine
where "nvcc" is installed.
EOF
fi

dir=exp/nnet3/tdnn_sp${affix:+_$affix}
gmm_dir=exp/tri5a
train_set=train_sp
ali_dir=${gmm_dir}_sp_ali
graph_dir=$gmm_dir/graph

local/nnet3/run_ivector_common.sh --stage $stage --online true || exit 1;

if [ $stage -le 7 ]; then
  echo "$0: creating neural net configs";

  ivector_dim=$(feat-to-dim scp:exp/nnet3/ivectors_${train_set}/ivector_online.scp - || exit 1;)
  feat_dim=$(feat-to-dim scp:data/${train_set}_hires_online/feats.scp - || exit 1;)
  num_targets=$(tree-info $ali_dir/tree |grep num-pdfs|awk '{print $2}')

  mkdir -p $dir/configs
  cat <<EOF > $dir/configs/network.xconfig
  input dim=$ivector_dim name=ivector
  input dim=$feat_dim name=input

  # please note that it is important to have input layer with the name=input
  # as the layer immediately preceding the fixed-affine-layer to enable
  # the use of short notation for the descriptor
  fixed-affine-layer name=lda input=Append(-2,-1,0,1,2,ReplaceIndex(ivector, t, 0)) affine-transform-file=$dir/configs/lda.mat

  # the first splicing is moved before the lda layer, so no splicing here
  relu-batchnorm-layer name=tdnn1 dim=850
  relu-batchnorm-layer name=tdnn2 dim=850 input=Append(-1,0,2)
  relu-batchnorm-layer name=tdnn3 dim=850 input=Append(-3,0,3)
  relu-batchnorm-layer name=tdnn4 dim=850 input=Append(-7,0,2)
  relu-batchnorm-layer name=tdnn5 dim=850 input=Append(-3,0,3)
  relu-batchnorm-layer name=tdnn6 dim=850
  output-layer name=output input=tdnn6 dim=$num_targets max-change=1.5
EOF
  steps/nnet3/xconfig_to_configs.py --xconfig-file $dir/configs/network.xconfig --config-dir $dir/configs/
fi

if [ $stage -le 8 ]; then
  if [[ $(hostname -f) == *.clsp.jhu.edu ]] && [ ! -d $dir/egs/storage ]; then
    utils/create_split_dir.pl \
     /export/b0{5,6,7,8}/$USER/kaldi-data/egs/aidatatang-$(date +'%m_%d_%H_%M')/s5/$dir/egs/storage $dir/egs/storage
  fi

  steps/nnet3/train_dnn.py --stage=$train_stage \
    --cmd="$decode_cmd" \
    --feat.online-ivector-dir exp/nnet3/ivectors_${train_set} \
    --feat.cmvn-opts="--norm-means=false --norm-vars=false" \
    --trainer.num-epochs $num_epochs \
    --trainer.optimization.num-jobs-initial $num_jobs_initial \
    --trainer.optimization.num-jobs-final $num_jobs_final \
    --trainer.optimization.initial-effective-lrate $initial_effective_lrate \
    --trainer.optimization.final-effective-lrate $final_effective_lrate \
    --egs.dir "$common_egs_dir" \
    --cleanup.remove-egs $remove_egs \
    --cleanup.preserve-model-interval 500 \
    --use-gpu true \
    --feat-dir=data/${train_set}_hires_online \
    --ali-dir $ali_dir \
    --lang data/lang \
    --reporting.email="$reporting_email" \
    --dir=$dir  || exit 1;
fi

if [ $stage -le 9 ]; then
  # this version of the decoding treats each utterance separately
  # without carrying forward speaker information.
  for decode_set in dev test; do
    num_jobs=`cat data/${decode_set}_hires_online/utt2spk|cut -d' ' -f2|sort -u|wc -l`
    decode_dir=${dir}/decode_$decode_set
    steps/nnet3/decode.sh --nj $num_jobs --cmd "$decode_cmd" \
       --online-ivector-dir exp/nnet3/ivectors_${decode_set} \
       $graph_dir data/${decode_set}_hires_online $decode_dir || exit 1;
  done
fi

if [ $stage -le 10 ]; then
  steps/online/nnet3/prepare_online_decoding.sh --mfcc-config conf/mfcc_hires.conf \
    --add-pitch true \
    data/lang exp/nnet3/extractor "$dir" ${dir}_online || exit 1;
fi

if [ $stage -le 11 ]; then
  # do the actual online decoding with iVectors, carrying info forward from
  # previous utterances of the same speaker.
  for decode_set in dev test; do
    num_jobs=`cat data/${decode_set}_hires_online/utt2spk|cut -d' ' -f2|sort -u|wc -l`
    decode_dir=${dir}_online/decode_$decode_set
    steps/online/nnet3/decode.sh --nj $num_jobs --cmd "$decode_cmd" \
       --config conf/decode.config \
       $graph_dir data/${decode_set}_hires_online $decode_dir || exit 1;
  done
fi

if [ $stage -le 12 ]; then
  # this version of the decoding treats each utterance separately
  # without carrying forward speaker information.
  for decode_set in dev test; do
    num_jobs=`cat data/${decode_set}_hires_online/utt2spk|cut -d' ' -f2|sort -u|wc -l`
    decode_dir=${dir}_online/decode_${decode_set}_per_utt
    steps/online/nnet3/decode.sh --nj $num_jobs --cmd "$decode_cmd" \
       --config conf/decode.config --per-utt true \
       $graph_dir data/${decode_set}_hires_online $decode_dir || exit 1;
  done
fi

wait;
exit 0;
```



#### steps/nnet3/get_egs.sh

```shell
#!/bin/bash

# Copyright 2012-2016 Johns Hopkins University (Author: Daniel Povey).  Apache 2.0.
#
# This script, which will generally be called from other neural-net training
# scripts, extracts the training examples used to train the neural net (and also
# the validation examples used for diagnostics), and puts them in separate archives.
#
# This script dumps egs with several frames of labels, controlled by the
# frames_per_eg config variable (default: 8).  This takes many times less disk
# space because typically we have 4 to 7 frames of context on the left and
# right, and this ends up getting shared.  This is at the expense of slightly
# higher disk I/O while training.

set -o pipefail
trap "" PIPE

# Begin configuration section.
cmd=run.pl
frame_subsampling_factor=1
frames_per_eg=8   # number of frames of labels per example.  more->less disk space and
                  # less time preparing egs, but more I/O during training.
                  # Note: may in general be a comma-separated string of alternative
                  # durations (more useful when using large chunks, e.g. for BLSTMs);
                  # the first one (the principal num-frames) is preferred.
left_context=4    # amount of left-context per eg (i.e. extra frames of input features
                  # not present in the output supervision).
right_context=4   # amount of right-context per eg.
left_context_initial=-1    # if >=0, left-context for first chunk of an utterance
right_context_final=-1     # if >=0, right-context for last chunk of an utterance
compress=true   # set this to false to disable compression (e.g. if you want to see whether
                # results are affected).

num_utts_subset=300     # number of utterances in validation and training
                        # subsets used for shrinkage and diagnostics.
num_valid_frames_combine=0 # #valid frames for combination weights at the very end.
num_train_frames_combine=60000 # # train frames for the above.
num_frames_diagnostic=10000 # number of frames for "compute_prob" jobs
samples_per_iter=400000 # this is the target number of egs in each archive of egs
                        # (prior to merging egs).  We probably should have called
                        # it egs_per_iter. This is just a guideline; it will pick
                        # a number that divides the number of samples in the
                        # entire data.

stage=0
nj=6         # This should be set to the maximum number of jobs you are
             # comfortable to run in parallel; you can increase it if your disk
             # speed is greater and you have more machines.
srand=0     # rand seed for nnet3-copy-egs and nnet3-shuffle-egs
online_ivector_dir=  # can be used if we are including speaker information as iVectors.
cmvn_opts=  # can be used for specifying CMVN options, if feature type is not lda (if lda,
            # it doesn't make sense to use different options than were used as input to the
            # LDA transform).  This is used to turn off CMVN in the online-nnet experiments.
generate_egs_scp=false # If true, it will generate egs.JOB.*.scp per egs archive

echo "$0 $@"  # Print the command line for logging

if [ -f path.sh ]; then . ./path.sh; fi
. parse_options.sh || exit 1;

if [ $# != 3 ]; then
  echo "Usage: $0 [opts] <data> <ali-dir> <egs-dir>"
  echo " e.g.: $0 data/train exp/tri3_ali exp/tri4_nnet/egs"
  echo ""
  echo "Main options (for others, see top of script file)"
  echo "  --config <config-file>                           # config file containing options"
  echo "  --nj <nj>                                        # The maximum number of jobs you want to run in"
  echo "                                                   # parallel (increase this only if you have good disk and"
  echo "                                                   # network speed).  default=6"
  echo "  --cmd (utils/run.pl;utils/queue.pl <queue opts>) # how to run jobs."
  echo "  --samples-per-iter <#samples;400000>             # Target number of egs per archive (option is badly named)"
  echo "  --frames-per-eg <frames;8>                       # number of frames per eg on disk"
  echo "                                                   # May be either a single number or a comma-separated list"
  echo "                                                   # of alternatives (useful when training LSTMs, where the"
  echo "                                                   # frames-per-eg is the chunk size, to get variety of chunk"
  echo "                                                   # sizes).  The first in the list is preferred and is used"
  echo "                                                   # when working out the number of archives etc."
  echo "  --left-context <int;4>                           # Number of frames on left side to append for feature input"
  echo "  --right-context <int;4>                          # Number of frames on right side to append for feature input"
  echo "  --left-context-initial <int;-1>                  # If >= 0, left-context for first chunk of an utterance"
  echo "  --right-context-final <int;-1>                   # If >= 0, right-context for last chunk of an utterance"
  echo "  --num-frames-diagnostic <#frames;4000>           # Number of frames used in computing (train,valid) diagnostics"
  echo "  --num-valid-frames-combine <#frames;10000>       # Number of frames used in getting combination weights at the"
  echo "                                                   # very end."
  echo "  --stage <stage|0>                                # Used to run a partially-completed training process from somewhere in"
  echo "                                                   # the middle."

  exit 1;
fi

data=$1
alidir=$2
dir=$3

# Check some files.
[ ! -z "$online_ivector_dir" ] && \
  extra_files="$online_ivector_dir/ivector_online.scp $online_ivector_dir/ivector_period"

for f in $data/feats.scp $alidir/ali.1.gz $alidir/final.mdl $alidir/tree $extra_files; do
  [ ! -f $f ] && echo "$0: no such file $f" && exit 1;
done

sdata=$data/split$nj
utils/split_data.sh $data $nj

mkdir -p $dir/log $dir/info
cp $alidir/tree $dir

num_ali_jobs=$(cat $alidir/num_jobs) || exit 1;


num_utts=$(cat $data/utt2spk | wc -l)
if ! [ $num_utts -gt $[$num_utts_subset*4] ]; then
  echo "$0: number of utterances $num_utts in your training data is too small versus --num-utts-subset=$num_utts_subset"
  echo "... you probably have so little data that it doesn't make sense to train a neural net."
  exit 1
fi

# Get list of validation utterances.
awk '{print $1}' $data/utt2spk | utils/shuffle_list.pl 2>/dev/null | head -$num_utts_subset \
    > $dir/valid_uttlist

if [ -f $data/utt2uniq ]; then  # this matters if you use data augmentation.
  echo "File $data/utt2uniq exists, so augmenting valid_uttlist to"
  echo "include all perturbed versions of the same 'real' utterances."
  mv $dir/valid_uttlist $dir/valid_uttlist.tmp
  utils/utt2spk_to_spk2utt.pl $data/utt2uniq > $dir/uniq2utt
  cat $dir/valid_uttlist.tmp | utils/apply_map.pl $data/utt2uniq | \
    sort | uniq | utils/apply_map.pl $dir/uniq2utt | \
    awk '{for(n=1;n<=NF;n++) print $n;}' | sort  > $dir/valid_uttlist
  rm $dir/uniq2utt $dir/valid_uttlist.tmp
fi

awk '{print $1}' $data/utt2spk | utils/filter_scp.pl --exclude $dir/valid_uttlist | \
   utils/shuffle_list.pl 2>/dev/null | head -$num_utts_subset > $dir/train_subset_uttlist

echo "$0: creating egs.  To ensure they are not deleted later you can do:  touch $dir/.nodelete"

## Set up features.
echo "$0: feature type is raw"

feats="ark,s,cs:utils/filter_scp.pl --exclude $dir/valid_uttlist $sdata/JOB/feats.scp | apply-cmvn $cmvn_opts --utt2spk=ark:$sdata/JOB/utt2spk scp:$sdata/JOB/cmvn.scp scp:- ark:- |"
valid_feats="ark,s,cs:utils/filter_scp.pl $dir/valid_uttlist $data/feats.scp | apply-cmvn $cmvn_opts --utt2spk=ark:$data/utt2spk scp:$data/cmvn.scp scp:- ark:- |"
train_subset_feats="ark,s,cs:utils/filter_scp.pl $dir/train_subset_uttlist $data/feats.scp | apply-cmvn $cmvn_opts --utt2spk=ark:$data/utt2spk scp:$data/cmvn.scp scp:- ark:- |"
echo $cmvn_opts >$dir/cmvn_opts # caution: the top-level nnet training script should copy this to its own dir now.

if [ ! -z "$online_ivector_dir" ]; then
  ivector_dim=$(feat-to-dim scp:$online_ivector_dir/ivector_online.scp -) || exit 1;
  echo $ivector_dim > $dir/info/ivector_dim
  steps/nnet2/get_ivector_id.sh $online_ivector_dir > $dir/info/final.ie.id || exit 1
  ivector_period=$(cat $online_ivector_dir/ivector_period) || exit 1;
  ivector_opts="--online-ivectors=scp:$online_ivector_dir/ivector_online.scp --online-ivector-period=$ivector_period"
else
  ivector_opts=""
  echo 0 >$dir/info/ivector_dim
fi

if [ $stage -le 1 ]; then
  echo "$0: working out number of frames of training data"
  num_frames=$(steps/nnet2/get_num_frames.sh $data)
  echo $num_frames > $dir/info/num_frames
  echo "$0: working out feature dim"
  feats_one="$(echo $feats | sed s/JOB/1/g)"
  if feat_dim=$(feat-to-dim "$feats_one" - 2>/dev/null); then
    echo $feat_dim > $dir/info/feat_dim
  else # run without redirection to show the error.
    feat-to-dim "$feats_one" -; exit 1
  fi
else
  num_frames=$(cat $dir/info/num_frames) || exit 1;
  feat_dim=$(cat $dir/info/feat_dim) || exit 1;
fi


# the first field in frames_per_eg (which is a comma-separated list of numbers)
# is the 'principal' frames-per-eg, and for purposes of working out the number
# of archives we assume that this will be the average number of frames per eg.
frames_per_eg_principal=$(echo $frames_per_eg | cut -d, -f1)

# the + 1 is to round up, not down... we assume it doesn't divide exactly.
num_archives=$[$num_frames/($frames_per_eg_principal*$samples_per_iter)+1]
if [ $num_archives -eq 1 ]; then
  echo "*** $0: warning: the --frames-per-eg is too large to generate one archive with"
  echo "*** as many as --samples-per-iter egs in it.  Consider reducing --frames-per-eg."
  sleep 4
fi

# We may have to first create a smaller number of larger archives, with number
# $num_archives_intermediate, if $num_archives is more than the maximum number
# of open filehandles that the system allows per process (ulimit -n).
# This sometimes gives a misleading answer as GridEngine sometimes changes that
# somehow, so we limit it to 512.
max_open_filehandles=$(ulimit -n) || exit 1
[ $max_open_filehandles -gt 512 ] && max_open_filehandles=512
num_archives_intermediate=$num_archives
archives_multiple=1
while [ $[$num_archives_intermediate+4] -gt $max_open_filehandles ]; do
  archives_multiple=$[$archives_multiple+1]
  num_archives_intermediate=$[$num_archives/$archives_multiple+1];
done
# now make sure num_archives is an exact multiple of archives_multiple.
num_archives=$[$archives_multiple*$num_archives_intermediate]

echo $num_archives >$dir/info/num_archives
echo $frames_per_eg >$dir/info/frames_per_eg
# Work out the number of egs per archive
egs_per_archive=$[$num_frames/($frames_per_eg_principal*$num_archives)]
! [ $egs_per_archive -le $samples_per_iter ] && \
  echo "$0: script error: egs_per_archive=$egs_per_archive not <= samples_per_iter=$samples_per_iter" \
  && exit 1;

echo $egs_per_archive > $dir/info/egs_per_archive

echo "$0: creating $num_archives archives, each with $egs_per_archive egs, with"
echo "$0:   $frames_per_eg labels per example, and (left,right) context = ($left_context,$right_context)"
if [ $left_context_initial -ge 0 ] || [ $right_context_final -ge 0 ]; then
  echo "$0:   ... and (left-context-initial,right-context-final) = ($left_context_initial,$right_context_final)"
fi



if [ -e $dir/storage ]; then
  # Make soft links to storage directories, if distributing this way..  See
  # utils/create_split_dir.pl.
  echo "$0: creating data links"
  utils/create_data_link.pl $(for x in $(seq $num_archives); do echo $dir/egs.$x.ark; done)
  for x in $(seq $num_archives_intermediate); do
    utils/create_data_link.pl $(for y in $(seq $nj); do echo $dir/egs_orig.$y.$x.ark; done)
  done
fi

if [ $stage -le 2 ]; then
  echo "$0: copying data alignments"
  for id in $(seq $num_ali_jobs); do gunzip -c $alidir/ali.$id.gz; done | \
    copy-int-vector ark:- ark,scp:$dir/ali.ark,$dir/ali.scp || exit 1;
fi

egs_opts="--left-context=$left_context --right-context=$right_context --compress=$compress --num-frames=$frames_per_eg"
[ $left_context_initial -ge 0 ] && egs_opts="$egs_opts --left-context-initial=$left_context_initial"
[ $right_context_final -ge 0 ] && egs_opts="$egs_opts --right-context-final=$right_context_final"

echo $left_context > $dir/info/left_context
echo $right_context > $dir/info/right_context
echo $left_context_initial > $dir/info/left_context_initial
echo $right_context_final > $dir/info/right_context_final


num_pdfs=$(tree-info --print-args=false $alidir/tree | grep num-pdfs | awk '{print $2}')
if [ $stage -le 3 ]; then
  echo "$0: Getting validation and training subset examples."
  rm $dir/.error 2>/dev/null
  echo "$0: ... extracting validation and training-subset alignments."


  # do the filtering just once, as ali.scp may be long.
  utils/filter_scp.pl <(cat $dir/valid_uttlist $dir/train_subset_uttlist) \
    <$dir/ali.scp >$dir/ali_special.scp

  $cmd $dir/log/create_valid_subset.log \
    utils/filter_scp.pl $dir/valid_uttlist $dir/ali_special.scp \| \
    ali-to-pdf $alidir/final.mdl scp:- ark:- \| \
    ali-to-post ark:- ark:- \| \
    nnet3-get-egs --num-pdfs=$num_pdfs --frame-subsampling-factor=$frame_subsampling_factor \
      $ivector_opts $egs_opts "$valid_feats" \
      ark,s,cs:- "ark:$dir/valid_all.egs" || touch $dir/.error &
  $cmd $dir/log/create_train_subset.log \
    utils/filter_scp.pl $dir/train_subset_uttlist $dir/ali_special.scp \| \
    ali-to-pdf $alidir/final.mdl scp:- ark:- \| \
    ali-to-post ark:- ark:- \| \
    nnet3-get-egs --num-pdfs=$num_pdfs --frame-subsampling-factor=$frame_subsampling_factor \
      $ivector_opts $egs_opts "$train_subset_feats" \
      ark,s,cs:- "ark:$dir/train_subset_all.egs" || touch $dir/.error &
  wait;
  [ -f $dir/.error ] && echo "Error detected while creating train/valid egs" && exit 1
  echo "... Getting subsets of validation examples for diagnostics and combination."
  if $generate_egs_scp; then
    valid_diagnostic_output="ark,scp:$dir/valid_diagnostic.egs,$dir/valid_diagnostic.scp"
    train_diagnostic_output="ark,scp:$dir/train_diagnostic.egs,$dir/train_diagnostic.scp"
  else
    valid_diagnostic_output="ark:$dir/valid_diagnostic.egs"
    train_diagnostic_output="ark:$dir/train_diagnostic.egs"
  fi
  $cmd $dir/log/create_valid_subset_combine.log \
    nnet3-subset-egs --n=$[$num_valid_frames_combine/$frames_per_eg_principal] ark:$dir/valid_all.egs \
      ark:$dir/valid_combine.egs || touch $dir/.error &
  $cmd $dir/log/create_valid_subset_diagnostic.log \
    nnet3-subset-egs --n=$[$num_frames_diagnostic/$frames_per_eg_principal] ark:$dir/valid_all.egs \
    $valid_diagnostic_output || touch $dir/.error &

  $cmd $dir/log/create_train_subset_combine.log \
    nnet3-subset-egs --n=$[$num_train_frames_combine/$frames_per_eg_principal] ark:$dir/train_subset_all.egs \
      ark:$dir/train_combine.egs || touch $dir/.error &
  $cmd $dir/log/create_train_subset_diagnostic.log \
    nnet3-subset-egs --n=$[$num_frames_diagnostic/$frames_per_eg_principal] ark:$dir/train_subset_all.egs \
    $train_diagnostic_output || touch $dir/.error &
  wait
  sleep 5  # wait for file system to sync.
  cat $dir/valid_combine.egs $dir/train_combine.egs > $dir/combine.egs
  if $generate_egs_scp; then
    cat $dir/valid_combine.egs $dir/train_combine.egs  | \
    nnet3-copy-egs ark:- ark,scp:$dir/combine.egs,$dir/combine.scp
    rm $dir/{train,valid}_combine.scp
  else
    cat $dir/valid_combine.egs $dir/train_combine.egs > $dir/combine.egs
  fi
  for f in $dir/{combine,train_diagnostic,valid_diagnostic}.egs; do
    [ ! -s $f ] && echo "No examples in file $f" && exit 1;
  done
  rm $dir/valid_all.egs $dir/train_subset_all.egs $dir/{train,valid}_combine.egs
fi

if [ $stage -le 4 ]; then
  # create egs_orig.*.*.ark; the first index goes to $nj,
  # the second to $num_archives_intermediate.

  egs_list=
  for n in $(seq $num_archives_intermediate); do
    egs_list="$egs_list ark:$dir/egs_orig.JOB.$n.ark"
  done
  echo "$0: Generating training examples on disk"
  # The examples will go round-robin to egs_list.
  $cmd JOB=1:$nj $dir/log/get_egs.JOB.log \
    nnet3-get-egs --num-pdfs=$num_pdfs --frame-subsampling-factor=$frame_subsampling_factor \
    $ivector_opts $egs_opts "$feats" \
    "ark,s,cs:filter_scp.pl $sdata/JOB/utt2spk $dir/ali.scp | ali-to-pdf $alidir/final.mdl scp:- ark:- | ali-to-post ark:- ark:- |" ark:- \| \
    nnet3-copy-egs --random=true --srand=\$[JOB+$srand] ark:- $egs_list || exit 1;
fi

if [ $stage -le 5 ]; then
  echo "$0: recombining and shuffling order of archives on disk"
  # combine all the "egs_orig.*.JOB.scp" (over the $nj splits of the data) and
  # shuffle the order, writing to the egs.JOB.ark

  # the input is a concatenation over the input jobs.
  egs_list=
  for n in $(seq $nj); do
    egs_list="$egs_list $dir/egs_orig.$n.JOB.ark"
  done

  if [ $archives_multiple == 1 ]; then # normal case.
    if $generate_egs_scp; then
      output_archive="ark,scp:$dir/egs.JOB.ark,$dir/egs.JOB.scp"
    else
      output_archive="ark:$dir/egs.JOB.ark"
    fi
    $cmd --max-jobs-run $nj JOB=1:$num_archives_intermediate $dir/log/shuffle.JOB.log \
      nnet3-shuffle-egs --srand=\$[JOB+$srand] "ark:cat $egs_list|" $output_archive  || exit 1;

    if $generate_egs_scp; then
      #concatenate egs.JOB.scp in single egs.scp
      rm $dir/egs.scp 2> /dev/null || true
      for j in $(seq $num_archives_intermediate); do
        cat $dir/egs.$j.scp || exit 1;
      done > $dir/egs.scp || exit 1;
      for f in $dir/egs.*.scp; do rm $f; done
    fi
  else
    # we need to shuffle the 'intermediate archives' and then split into the
    # final archives.  we create soft links to manage this splitting, because
    # otherwise managing the output names is quite difficult (and we don't want
    # to submit separate queue jobs for each intermediate archive, because then
    # the --max-jobs-run option is hard to enforce).
    if $generate_egs_scp; then
      output_archives="$(for y in $(seq $archives_multiple); do echo ark,scp:$dir/egs.JOB.$y.ark,$dir/egs.JOB.$y.scp; done)"
    else
      output_archives="$(for y in $(seq $archives_multiple); do echo ark:$dir/egs.JOB.$y.ark; done)"
    fi
    for x in $(seq $num_archives_intermediate); do
      for y in $(seq $archives_multiple); do
        archive_index=$[($x-1)*$archives_multiple+$y]
        # egs.intermediate_archive.{1,2,...}.ark will point to egs.archive.ark
        ln -sf egs.$archive_index.ark $dir/egs.$x.$y.ark || exit 1
      done
    done
    $cmd --max-jobs-run $nj JOB=1:$num_archives_intermediate $dir/log/shuffle.JOB.log \
      nnet3-shuffle-egs --srand=\$[JOB+$srand] "ark:cat $egs_list|" ark:- \| \
      nnet3-copy-egs ark:- $output_archives || exit 1;

    if $generate_egs_scp; then
      #concatenate egs.JOB.scp in single egs.scp
      rm $dir/egs.scp 2> /dev/null || true
      for j in $(seq $num_archives_intermediate); do
        for y in $(seq $num_archives_intermediate); do
          cat $dir/egs.$j.$y.scp || exit 1;
        done
      done > $dir/egs.scp || exit 1;
      for f in $dir/egs.*.*.scp; do rm $f; done
    fi
  fi
fi

if [ $frame_subsampling_factor -ne 1 ]; then
  echo $frame_subsampling_factor > $dir/info/frame_subsampling_factor
fi

if [ $stage -le 6 ]; then
  echo "$0: removing temporary archives"
  for x in $(seq $nj); do
    for y in $(seq $num_archives_intermediate); do
      file=$dir/egs_orig.$x.$y.ark
      [ -L $file ] && rm $(utils/make_absolute.sh $file)
      rm $file
    done
  done
  if [ $archives_multiple -gt 1 ]; then
    # there are some extra soft links that we should delete.
    for f in $dir/egs.*.*.ark; do rm $f; done
  fi
  echo "$0: removing temporary alignments"
  # Ignore errors below because trans.* might not exist.
  rm $dir/ali.{ark,scp} 2>/dev/null
fi

echo "$0: Finished preparing training examples"
```



### chain

#### run_tdnn_1a.sh

```shell
#!/bin/bash

# This script is based on run_tdnn_7h.sh in swbd chain recipe.

set -e

# configs for 'chain'
affix=
stage=0
train_stage=-10
get_egs_stage=-10
dir=exp/chain/tdnn_1a  # Note: _sp will get added to this
decode_iter=

# training options
num_epochs=4
initial_effective_lrate=0.001
final_effective_lrate=0.0001
max_param_change=2.0
final_layer_normalize_target=0.5
num_jobs_initial=2
num_jobs_final=12
minibatch_size=128
frames_per_eg=150,110,90
remove_egs=true
common_egs_dir=
xent_regularize=0.1

# End configuration section.
echo "$0 $@"  # Print the command line for logging

. ./cmd.sh
. ./path.sh
. ./utils/parse_options.sh

if ! cuda-compiled; then
  cat <<EOF && exit 1
This script is intended to be used with GPUs but you have not compiled Kaldi with CUDA
If you want to use GPUs (and have them), go to src/, and configure and make on a machine
where "nvcc" is installed.
EOF
fi

# The iVector-extraction and feature-dumping parts are the same as the standard
# nnet3 setup, and you can skip them by setting "--stage 8" if you have already
# run those things.

dir=${dir}${affix:+_$affix}_sp
train_set=train_sp
ali_dir=exp/tri5a_sp_ali
treedir=exp/chain/tri6_7d_tree_sp
lang=data/lang_chain


# if we are using the speed-perturbed data we need to generate
# alignments for it.
local/nnet3/run_ivector_common.sh --stage $stage || exit 1;

if [ $stage -le 7 ]; then
  # Get the alignments as lattices (gives the LF-MMI training more freedom).
  # use the same num-jobs as the alignments
  nj=$(cat $ali_dir/num_jobs) || exit 1;
  steps/align_fmllr_lats.sh --nj $nj --cmd "$train_cmd" data/$train_set \
    data/lang exp/tri5a exp/tri5a_sp_lats
  rm exp/tri5a_sp_lats/fsts.*.gz # save space
fi

if [ $stage -le 8 ]; then
  # Create a version of the lang/ directory that has one state per phone in the
  # topo file. [note, it really has two states.. the first one is only repeated
  # once, the second one has zero or more repeats.]
  rm -rf $lang
  cp -r data/lang $lang
  silphonelist=$(cat $lang/phones/silence.csl) || exit 1;
  nonsilphonelist=$(cat $lang/phones/nonsilence.csl) || exit 1;
  # Use our special topology... note that later on may have to tune this
  # topology.
  steps/nnet3/chain/gen_topo.py $nonsilphonelist $silphonelist >$lang/topo
fi

if [ $stage -le 9 ]; then
  # Build a tree using our new topology. This is the critically different
  # step compared with other recipes.
  steps/nnet3/chain/build_tree.sh --frame-subsampling-factor 3 \
      --context-opts "--context-width=2 --central-position=1" \
      --cmd "$train_cmd" 5000 data/$train_set $lang $ali_dir $treedir
fi

if [ $stage -le 10 ]; then
  echo "$0: creating neural net configs using the xconfig parser";

  num_targets=$(tree-info $treedir/tree |grep num-pdfs|awk '{print $2}')
  learning_rate_factor=$(echo "print (0.5/$xent_regularize)" | python)

  mkdir -p $dir/configs
  cat <<EOF > $dir/configs/network.xconfig
  input dim=100 name=ivector
  input dim=43 name=input

  # please note that it is important to have input layer with the name=input
  # as the layer immediately preceding the fixed-affine-layer to enable
  # the use of short notation for the descriptor
  fixed-affine-layer name=lda input=Append(-1,0,1,ReplaceIndex(ivector, t, 0)) affine-transform-file=$dir/configs/lda.mat

  # the first splicing is moved before the lda layer, so no splicing here
  relu-batchnorm-layer name=tdnn1 dim=625
  relu-batchnorm-layer name=tdnn2 input=Append(-1,0,1) dim=625
  relu-batchnorm-layer name=tdnn3 input=Append(-1,0,1) dim=625
  relu-batchnorm-layer name=tdnn4 input=Append(-3,0,3) dim=625
  relu-batchnorm-layer name=tdnn5 input=Append(-3,0,3) dim=625
  relu-batchnorm-layer name=tdnn6 input=Append(-3,0,3) dim=625

  ## adding the layers for chain branch
  relu-batchnorm-layer name=prefinal-chain input=tdnn6 dim=625 target-rms=0.5
  output-layer name=output include-log-softmax=false dim=$num_targets max-change=1.5

  # adding the layers for xent branch
  # This block prints the configs for a separate output that will be
  # trained with a cross-entropy objective in the 'chain' models... this
  # has the effect of regularizing the hidden parts of the model.  we use
  # 0.5 / args.xent_regularize as the learning rate factor- the factor of
  # 0.5 / args.xent_regularize is suitable as it means the xent
  # final-layer learns at a rate independent of the regularization
  # constant; and the 0.5 was tuned so as to make the relative progress
  # similar in the xent and regular final layers.
  relu-batchnorm-layer name=prefinal-xent input=tdnn6 dim=625 target-rms=0.5
  output-layer name=output-xent dim=$num_targets learning-rate-factor=$learning_rate_factor max-change=1.5

EOF
  steps/nnet3/xconfig_to_configs.py --xconfig-file $dir/configs/network.xconfig --config-dir $dir/configs/
fi

if [ $stage -le 11 ]; then
  if [[ $(hostname -f) == *.clsp.jhu.edu ]] && [ ! -d $dir/egs/storage ]; then
    utils/create_split_dir.pl \
     /export/b0{5,6,7,8}/$USER/kaldi-data/egs/aidatatang-$(date +'%m_%d_%H_%M')/s5c/$dir/egs/storage $dir/egs/storage
  fi

  steps/nnet3/chain/train.py --stage $train_stage \
    --cmd "$decode_cmd" \
    --feat.online-ivector-dir exp/nnet3/ivectors_${train_set} \
    --feat.cmvn-opts "--norm-means=false --norm-vars=false" \
    --chain.xent-regularize $xent_regularize \
    --chain.leaky-hmm-coefficient 0.1 \
    --chain.l2-regularize 0.00005 \
    --chain.apply-deriv-weights false \
    --chain.lm-opts="--num-extra-lm-states=2000" \
    --egs.dir "$common_egs_dir" \
    --egs.stage $get_egs_stage \
    --egs.opts "--frames-overlap-per-eg 0" \
    --egs.chunk-width $frames_per_eg \
    --trainer.num-chunk-per-minibatch $minibatch_size \
    --trainer.frames-per-iter 1500000 \
    --trainer.num-epochs $num_epochs \
    --trainer.optimization.num-jobs-initial $num_jobs_initial \
    --trainer.optimization.num-jobs-final $num_jobs_final \
    --trainer.optimization.initial-effective-lrate $initial_effective_lrate \
    --trainer.optimization.final-effective-lrate $final_effective_lrate \
    --trainer.max-param-change $max_param_change \
    --cleanup.remove-egs $remove_egs \
    --feat-dir data/${train_set}_hires \
    --tree-dir $treedir \
    --lat-dir exp/tri5a_sp_lats \
    --dir $dir  || exit 1;
fi

if [ $stage -le 12 ]; then
  # Note: it might appear that this $lang directory is mismatched, and it is as
  # far as the 'topo' is concerned, but this script doesn't read the 'topo' from
  # the lang directory.
  utils/mkgraph.sh --self-loop-scale 1.0 data/lang_test $dir $dir/graph
fi

graph_dir=$dir/graph
if [ $stage -le 13 ]; then
  for test_set in dev test; do
    steps/nnet3/decode.sh --acwt 1.0 --post-decode-acwt 10.0 \
      --nj 10 --cmd "$decode_cmd" \
      --online-ivector-dir exp/nnet3/ivectors_$test_set \
      $graph_dir data/${test_set}_hires $dir/decode_${test_set} || exit 1;
  done
fi

exit;
```



#### run_tdnn_2a.sh

```shell
#!/bin/bash

# This script is based on run_tdnn_1a.sh.
# This setup used online pitch to train the neural network.
# It requires a online_pitch.conf in the conf dir.

set -e

# configs for 'chain'
affix=
stage=8
train_stage=-10
get_egs_stage=-10
dir=exp/chain/tdnn_2a  # Note: _sp will get added to this
decode_iter=

# training options
num_epochs=4
initial_effective_lrate=0.001
final_effective_lrate=0.0001
max_param_change=2.0
final_layer_normalize_target=0.5
num_jobs_initial=2
num_jobs_final=12
minibatch_size=128
frames_per_eg=150,110,90
remove_egs=true
common_egs_dir=
xent_regularize=0.1

# End configuration section.
echo "$0 $@"  # Print the command line for logging

. ./cmd.sh
. ./path.sh
. ./utils/parse_options.sh

if ! cuda-compiled; then
  cat <<EOF && exit 1
This script is intended to be used with GPUs but you have not compiled Kaldi with CUDA
If you want to use GPUs (and have them), go to src/, and configure and make on a machine
where "nvcc" is installed.
EOF
fi

# The iVector-extraction and feature-dumping parts are the same as the standard
# nnet3 setup, and you can skip them by setting "--stage 8" if you have already
# run those things.

dir=${dir}${affix:+_$affix}_sp
train_set=train_sp
ali_dir=exp/tri5a_sp_ali
treedir=exp/chain/tri6_7d_tree_sp
lang=data/lang_chain


# if we are using the speed-perturbed data we need to generate
# alignments for it.
local/nnet3/run_ivector_common.sh --stage $stage --online true || exit 1;

if [ $stage -le 7 ]; then
  # Get the alignments as lattices (gives the LF-MMI training more freedom).
  # use the same num-jobs as the alignments
  nj=$(cat $ali_dir/num_jobs) || exit 1;
  steps/align_fmllr_lats.sh --nj $nj --cmd "$train_cmd" data/$train_set \
    data/lang exp/tri5a exp/tri5a_sp_lats
  rm exp/tri5a_sp_lats/fsts.*.gz # save space
fi

if [ $stage -le 8 ]; then
  # Create a version of the lang/ directory that has one state per phone in the
  # topo file. [note, it really has two states.. the first one is only repeated
  # once, the second one has zero or more repeats.]
  rm -rf $lang
  cp -r data/lang $lang
  silphonelist=$(cat $lang/phones/silence.csl) || exit 1;
  nonsilphonelist=$(cat $lang/phones/nonsilence.csl) || exit 1;
  # Use our special topology... note that later on may have to tune this
  # topology.
  steps/nnet3/chain/gen_topo.py $nonsilphonelist $silphonelist >$lang/topo
fi

if [ $stage -le 9 ]; then
  # Build a tree using our new topology. This is the critically different
  # step compared with other recipes.
  steps/nnet3/chain/build_tree.sh --frame-subsampling-factor 3 \
      --context-opts "--context-width=2 --central-position=1" \
      --cmd "$train_cmd" 5000 data/$train_set $lang $ali_dir $treedir
fi

if [ $stage -le 10 ]; then
  echo "$0: creating neural net configs using the xconfig parser";

  num_targets=$(tree-info $treedir/tree |grep num-pdfs|awk '{print $2}')
  learning_rate_factor=$(echo "print (0.5/$xent_regularize)" | python)

  mkdir -p $dir/configs
  cat <<EOF > $dir/configs/network.xconfig
  input dim=100 name=ivector
  input dim=43 name=input

  # please note that it is important to have input layer with the name=input
  # as the layer immediately preceding the fixed-affine-layer to enable
  # the use of short notation for the descriptor
  fixed-affine-layer name=lda input=Append(-1,0,1,ReplaceIndex(ivector, t, 0)) affine-transform-file=$dir/configs/lda.mat

  # the first splicing is moved before the lda layer, so no splicing here
  relu-batchnorm-layer name=tdnn1 dim=625
  relu-batchnorm-layer name=tdnn2 input=Append(-1,0,1) dim=625
  relu-batchnorm-layer name=tdnn3 input=Append(-1,0,1) dim=625
  relu-batchnorm-layer name=tdnn4 input=Append(-3,0,3) dim=625
  relu-batchnorm-layer name=tdnn5 input=Append(-3,0,3) dim=625
  relu-batchnorm-layer name=tdnn6 input=Append(-3,0,3) dim=625

  ## adding the layers for chain branch
  relu-batchnorm-layer name=prefinal-chain input=tdnn6 dim=625 target-rms=0.5
  output-layer name=output include-log-softmax=false dim=$num_targets max-change=1.5

  # adding the layers for xent branch
  # This block prints the configs for a separate output that will be
  # trained with a cross-entropy objective in the 'chain' models... this
  # has the effect of regularizing the hidden parts of the model.  we use
  # 0.5 / args.xent_regularize as the learning rate factor- the factor of
  # 0.5 / args.xent_regularize is suitable as it means the xent
  # final-layer learns at a rate independent of the regularization
  # constant; and the 0.5 was tuned so as to make the relative progress
  # similar in the xent and regular final layers.
  relu-batchnorm-layer name=prefinal-xent input=tdnn6 dim=625 target-rms=0.5
  output-layer name=output-xent dim=$num_targets learning-rate-factor=$learning_rate_factor max-change=1.5

EOF
  steps/nnet3/xconfig_to_configs.py --xconfig-file $dir/configs/network.xconfig --config-dir $dir/configs/
fi

if [ $stage -le 11 ]; then
  if [[ $(hostname -f) == *.clsp.jhu.edu ]] && [ ! -d $dir/egs/storage ]; then
    utils/create_split_dir.pl \
     /export/b0{5,6,7,8}/$USER/kaldi-data/egs/aishell-$(date +'%m_%d_%H_%M')/s5c/$dir/egs/storage $dir/egs/storage
  fi

  steps/nnet3/chain/train.py --stage $train_stage \
    --cmd "$decode_cmd" \
    --feat.online-ivector-dir exp/nnet3/ivectors_${train_set} \
    --feat.cmvn-opts "--norm-means=false --norm-vars=false" \
    --chain.xent-regularize $xent_regularize \
    --chain.leaky-hmm-coefficient 0.1 \
    --chain.l2-regularize 0.00005 \
    --chain.apply-deriv-weights false \
    --chain.lm-opts="--num-extra-lm-states=2000" \
    --egs.dir "$common_egs_dir" \
    --egs.stage $get_egs_stage \
    --egs.opts "--frames-overlap-per-eg 0" \
    --egs.chunk-width $frames_per_eg \
    --trainer.num-chunk-per-minibatch $minibatch_size \
    --trainer.frames-per-iter 1500000 \
    --trainer.num-epochs $num_epochs \
    --trainer.optimization.num-jobs-initial $num_jobs_initial \
    --trainer.optimization.num-jobs-final $num_jobs_final \
    --trainer.optimization.initial-effective-lrate $initial_effective_lrate \
    --trainer.optimization.final-effective-lrate $final_effective_lrate \
    --trainer.max-param-change $max_param_change \
    --cleanup.remove-egs $remove_egs \
    --feat-dir data/${train_set}_hires_online \
    --tree-dir $treedir \
    --lat-dir exp/tri5a_sp_lats \
    --dir $dir  || exit 1;
fi

if [ $stage -le 12 ]; then
  # Note: it might appear that this $lang directory is mismatched, and it is as
  # far as the 'topo' is concerned, but this script doesn't read the 'topo' from
  # the lang directory.
  utils/mkgraph.sh --self-loop-scale 1.0 data/lang_test $dir $dir/graph
fi

graph_dir=$dir/graph
if [ $stage -le 13 ]; then
  for test_set in dev test; do
    steps/nnet3/decode.sh --acwt 1.0 --post-decode-acwt 10.0 \
      --nj 10 --cmd "$decode_cmd" \
      --online-ivector-dir exp/nnet3/ivectors_$test_set \
      $graph_dir data/${test_set}_hires_online $dir/decode_${test_set} || exit 1;
  done
fi

if [ $stage -le 14 ]; then
  steps/online/nnet3/prepare_online_decoding.sh --mfcc-config conf/mfcc_hires.conf \
    --add-pitch true \
    $lang exp/nnet3/extractor "$dir" ${dir}_online || exit 1;
fi

dir=${dir}_online
if [ $stage -le 15 ]; then
  for test_set in dev test; do
    steps/online/nnet3/decode.sh --acwt 1.0 --post-decode-acwt 10.0 \
      --nj 10 --cmd "$decode_cmd" \
      --config conf/decode.config \
      $graph_dir data/${test_set}_hires_online $dir/decode_${test_set} || exit 1;
  done
fi

if [ $stage -le 16 ]; then
  for test_set in dev test; do
    steps/online/nnet3/decode.sh --acwt 1.0 --post-decode-acwt 10.0 \
      --nj 10 --cmd "$decode_cmd" --per-utt true \
      --config conf/decode.config \
      $graph_dir data/${test_set}_hires_online $dir/decode_${test_set}_per_utt || exit 1;
  done
fi

exit;
```



#### steps/nnet3/chain/train.py

注：经过了修改，添加了参数--use-gpu=wait

```python
#!/usr/bin/env python

# Copyright 2016    Vijayaditya Peddinti.
#           2016    Vimal Manohar
# Apache 2.0.

""" This script is based on steps/nnet3/chain/train.sh
"""
from __future__ import division
from __future__ import print_function

import argparse
import logging
import os
import pprint
import shutil
import sys
import traceback

sys.path.insert(0, 'steps')
import libs.nnet3.train.common as common_train_lib
import libs.common as common_lib
import libs.nnet3.train.chain_objf.acoustic_model as chain_lib
import libs.nnet3.report.log_parse as nnet3_log_parse


logger = logging.getLogger('libs')
logger.setLevel(logging.INFO)
handler = logging.StreamHandler()
handler.setLevel(logging.INFO)
formatter = logging.Formatter("%(asctime)s [%(pathname)s:%(lineno)s - "
                              "%(funcName)s - %(levelname)s ] %(message)s")
handler.setFormatter(formatter)
logger.addHandler(handler)
logger.info('Starting chain model trainer (train.py)')


def get_args():
    """ Get args from stdin.

    We add compulsary arguments as named arguments for readability

    The common options are defined in the object
    libs.nnet3.train.common.CommonParser.parser.
    See steps/libs/nnet3/train/common.py
    """

    parser = argparse.ArgumentParser(
        description="""Trains RNN and DNN acoustic models using the 'chain'
        objective function.""",
        formatter_class=argparse.ArgumentDefaultsHelpFormatter,
        conflict_handler='resolve',
        parents=[common_train_lib.CommonParser().parser])

    # egs extraction options
    parser.add_argument("--egs.chunk-width", type=str, dest='chunk_width',
                        default="20",
                        help="""Number of frames per chunk in the examples
                        used to train the RNN.   Caution: if you double this you
                        should halve --trainer.samples-per-iter.  May be
                        a comma-separated list of alternatives: first width
                        is the 'principal' chunk-width, used preferentially""")

    # chain options
    parser.add_argument("--chain.lm-opts", type=str, dest='lm_opts',
                        default=None, action=common_lib.NullstrToNoneAction,
                        help="options to be be passed to chain-est-phone-lm")
    parser.add_argument("--chain.l2-regularize", type=float,
                        dest='l2_regularize', default=0.0,
                        help="""Weight of regularization function which is the
                        l2-norm of the output of the network. It should be used
                        without the log-softmax layer for the outputs.  As
                        l2-norm of the log-softmax outputs can dominate the
                        objective function.""")
    parser.add_argument("--chain.xent-regularize", type=float,
                        dest='xent_regularize', default=0.0,
                        help="Weight of regularization function which is the "
                        "cross-entropy cost the outputs.")
    parser.add_argument("--chain.right-tolerance", type=int,
                        dest='right_tolerance', default=5, help="")
    parser.add_argument("--chain.left-tolerance", type=int,
                        dest='left_tolerance', default=5, help="")
    parser.add_argument("--chain.leaky-hmm-coefficient", type=float,
                        dest='leaky_hmm_coefficient', default=0.00001,
                        help="")
    parser.add_argument("--chain.apply-deriv-weights", type=str,
                        dest='apply_deriv_weights', default=True,
                        action=common_lib.StrToBoolAction,
                        choices=["true", "false"],
                        help="")
    parser.add_argument("--chain.frame-subsampling-factor", type=int,
                        dest='frame_subsampling_factor', default=3,
                        help="ratio of frames-per-second of features we "
                        "train on, to chain model's output")
    parser.add_argument("--chain.alignment-subsampling-factor", type=int,
                        dest='alignment_subsampling_factor',
                        default=3,
                        help="ratio of frames-per-second of input "
                        "alignments to chain model's output")
    parser.add_argument("--chain.left-deriv-truncate", type=int,
                        dest='left_deriv_truncate',
                        default=None,
                        help="Deprecated. Kept for back compatibility")

    # trainer options
    parser.add_argument("--trainer.input-model", type=str,
                        dest='input_model', default=None,
                        action=common_lib.NullstrToNoneAction,
                        help="If specified, this model is used as initial "
                             "'raw' model (0.raw in the script) instead of "
                             "initializing the model from the xconfig. "
                             "Also configs dir is not expected to exist "
                             "and left/right context is computed from this "
                             "model.")
    parser.add_argument("--trainer.num-epochs", type=float, dest='num_epochs',
                        default=10.0,
                        help="Number of epochs to train the model")
    parser.add_argument("--trainer.frames-per-iter", type=int,
                        dest='frames_per_iter', default=800000,
                        help="""Each iteration of training, see this many
                        [input] frames per job.  This option is passed to
                        get_egs.sh.  Aim for about a minute of training
                        time""")

    parser.add_argument("--trainer.num-chunk-per-minibatch", type=str,
                        dest='num_chunk_per_minibatch', default='128',
                        help="""Number of sequences to be processed in
                        parallel every minibatch.  May be a more general
                        rule as accepted by the --minibatch-size option of
                        nnet3-merge-egs; run that program without args to see
                        the format.""")

    # Parameters for the optimization
    parser.add_argument("--trainer.optimization.initial-effective-lrate",
                        type=float, dest='initial_effective_lrate',
                        default=0.0002,
                        help="Learning rate used during the initial iteration")
    parser.add_argument("--trainer.optimization.final-effective-lrate",
                        type=float, dest='final_effective_lrate',
                        default=0.00002,
                        help="Learning rate used during the final iteration")
    parser.add_argument("--trainer.optimization.shrink-value", type=float,
                        dest='shrink_value', default=1.0,
                        help="""Scaling factor used for scaling the parameter
                        matrices when the derivative averages are below the
                        shrink-threshold at the non-linearities.  E.g. 0.99.
                        Only applicable when the neural net contains sigmoid or
                        tanh units.""")
    parser.add_argument("--trainer.optimization.shrink-saturation-threshold",
                        type=float,
                        dest='shrink_saturation_threshold', default=0.40,
                        help="""Threshold that controls when we apply the
                        'shrinkage' (i.e. scaling by shrink-value).  If the
                        saturation of the sigmoid and tanh nonlinearities in
                        the neural net (as measured by
                        steps/nnet3/get_saturation.pl) exceeds this threshold
                        we scale the parameter matrices with the
                        shrink-value.""")
    # RNN-specific training options
    parser.add_argument("--trainer.deriv-truncate-margin", type=int,
                        dest='deriv_truncate_margin', default=None,
                        help="""(Relevant only for recurrent models). If
                        specified, gives the margin (in input frames) around
                        the 'required' part of each chunk that the derivatives
                        are backpropagated to. If unset, the derivatives are
                        backpropagated all the way to the boundaries of the
                        input data. E.g. 8 is a reasonable setting. Note: the
                        'required' part of the chunk is defined by the model's
                        {left,right}-context.""")

    # General options
    parser.add_argument("--feat-dir", type=str, required=True,
                        help="Directory with features used for training "
                        "the neural network.")
    parser.add_argument("--tree-dir", type=str, required=True,
                        help="""Directory containing the tree to use for this
                        model (we also expect final.mdl and ali.*.gz in that
                        directory""")
    parser.add_argument("--lat-dir", type=str, required=True,
                        help="Directory with numerator lattices "
                        "used for training the neural network.")
    parser.add_argument("--dir", type=str, required=True,
                        help="Directory to store the models and "
                        "all other files.")

    print(' '.join(sys.argv))
    print(sys.argv)

    args = parser.parse_args()

    [args, run_opts] = process_args(args)

    return [args, run_opts]


def process_args(args):
    """ Process the options got from get_args()
    """

    if not common_train_lib.validate_chunk_width(args.chunk_width):
        raise Exception("--egs.chunk-width has an invalid value")

    if not common_train_lib.validate_minibatch_size_str(args.num_chunk_per_minibatch):
        raise Exception("--trainer.num-chunk-per-minibatch has an invalid value")

    if args.chunk_left_context < 0:
        raise Exception("--egs.chunk-left-context should be non-negative")

    if args.chunk_right_context < 0:
        raise Exception("--egs.chunk-right-context should be non-negative")

    if args.left_deriv_truncate is not None:
        args.deriv_truncate_margin = -args.left_deriv_truncate
        logger.warning(
            "--chain.left-deriv-truncate (deprecated) is set by user, and "
            "--trainer.deriv-truncate-margin is set to negative of that "
            "value={0}. We recommend using the option "
            "--trainer.deriv-truncate-margin.".format(
                args.deriv_truncate_margin))

    if (not os.path.exists(args.dir)):
        raise Exception("This script expects --dir={0} to exist.")
    if (not os.path.exists(args.dir+"/configs") and
        (args.input_model is None or not os.path.exists(args.input_model))):
        raise Exception("Either --trainer.input-model option should be supplied, "
                        "and exist; or the {0}/configs directory should exist."
                        "".format(args.dir))

    # set the options corresponding to args.use_gpu
    run_opts = common_train_lib.RunOpts()
    if args.use_gpu in ["true", "false"]:
        args.use_gpu = ("yes" if args.use_gpu == "true" else "no")
    if args.use_gpu in ["yes", "wait"]:
        if not common_lib.check_if_cuda_compiled():
            logger.warning(
                """You are running with one thread but you have not compiled
                   for CUDA.  You may be running a setup optimized for GPUs.
                   If you have GPUs and have nvcc installed, go to src/ and do
                   ./configure; make""")

        run_opts.train_queue_opt = "--gpu 1"
        run_opts.parallel_train_opts = "--use-gpu={}".format(args.use_gpu)
        run_opts.combine_queue_opt = "--gpu 1"
        run_opts.combine_gpu_opt = "--use-gpu={}".format(args.use_gpu)

    else:
        logger.warning("Without using a GPU this will be very slow. "
                       "nnet3 does not yet support multiple threads.")

        run_opts.train_queue_opt = ""
        run_opts.parallel_train_opts = "--use-gpu=no"
        run_opts.combine_queue_opt = ""
        run_opts.combine_gpu_opt = "--use-gpu=no"

    run_opts.command = args.command
    run_opts.egs_command = (args.egs_command
                            if args.egs_command is not None else
                            args.command)

    return [args, run_opts]


def train(args, run_opts):
    """ The main function for training.

    Args:
        args: a Namespace object with the required parameters
            obtained from the function process_args()
        run_opts: RunOpts object obtained from the process_args()
    """

    arg_string = pprint.pformat(vars(args))
    logger.info("Arguments for the experiment\n{0}".format(arg_string))

    # Check files
    chain_lib.check_for_required_files(args.feat_dir, args.tree_dir,
                                       args.lat_dir if args.egs_dir is None
                                       else None)

    # Copy phones.txt from tree-dir to dir. Later, steps/nnet3/decode.sh will
    # use it to check compatibility between training and decoding phone-sets.
    shutil.copy('{0}/phones.txt'.format(args.tree_dir), args.dir)

    # Set some variables.
    num_jobs = common_lib.get_number_of_jobs(args.tree_dir)
    feat_dim = common_lib.get_feat_dim(args.feat_dir)
    ivector_dim = common_lib.get_ivector_dim(args.online_ivector_dir)
    ivector_id = common_lib.get_ivector_extractor_id(args.online_ivector_dir)

    # split the training data into parts for individual jobs
    # we will use the same number of jobs as that used for alignment
    common_lib.execute_command("utils/split_data.sh {0} {1}"
                               "".format(args.feat_dir, num_jobs))
    with open('{0}/num_jobs'.format(args.dir), 'w') as f:
        f.write(str(num_jobs))

    if args.input_model is None:
        config_dir = '{0}/configs'.format(args.dir)
        var_file = '{0}/vars'.format(config_dir)

        variables = common_train_lib.parse_generic_config_vars_file(var_file)
    else:
        # If args.input_model is specified, the model left and right contexts
        # are computed using input_model.
        variables = common_train_lib.get_input_model_info(args.input_model)

    # Set some variables.
    try:
        model_left_context = variables['model_left_context']
        model_right_context = variables['model_right_context']
    except KeyError as e:
        raise Exception("KeyError {0}: Variables need to be defined in "
                        "{1}".format(str(e), '{0}/configs'.format(args.dir)))

    left_context = args.chunk_left_context + model_left_context
    right_context = args.chunk_right_context + model_right_context
    left_context_initial = (args.chunk_left_context_initial + model_left_context if
                            args.chunk_left_context_initial >= 0 else -1)
    right_context_final = (args.chunk_right_context_final + model_right_context if
                           args.chunk_right_context_final >= 0 else -1)

    # Initialize as "raw" nnet, prior to training the LDA-like preconditioning
    # matrix.  This first config just does any initial splicing that we do;
    # we do this as it's a convenient way to get the stats for the 'lda-like'
    # transform.
    if (args.stage <= -6):
        logger.info("Creating phone language-model")
        chain_lib.create_phone_lm(args.dir, args.tree_dir, run_opts,
                                  lm_opts=args.lm_opts)

    if (args.stage <= -5):
        logger.info("Creating denominator FST")
        shutil.copy('{0}/tree'.format(args.tree_dir), args.dir)
        chain_lib.create_denominator_fst(args.dir, args.tree_dir, run_opts)

    if ((args.stage <= -4) and
            os.path.exists("{0}/configs/init.config".format(args.dir))
            and (args.input_model is None)):
        logger.info("Initializing a basic network for estimating "
                    "preconditioning matrix")
        common_lib.execute_command(
            """{command} {dir}/log/nnet_init.log \
            nnet3-init --srand=-2 {dir}/configs/init.config \
            {dir}/init.raw""".format(command=run_opts.command,
                                     dir=args.dir))

    egs_left_context = left_context + args.frame_subsampling_factor // 2
    egs_right_context = right_context + args.frame_subsampling_factor // 2
    # note: the '+ args.frame_subsampling_factor / 2' is to allow for the
    # fact that we'll be shifting the data slightly during training to give
    # variety to the training data.
    egs_left_context_initial = (left_context_initial +
                                args.frame_subsampling_factor // 2 if
                                left_context_initial >= 0 else -1)
    egs_right_context_final = (right_context_final +
                               args.frame_subsampling_factor // 2 if
                               right_context_final >= 0 else -1)

    default_egs_dir = '{0}/egs'.format(args.dir)
    if ((args.stage <= -3) and args.egs_dir is None):
        logger.info("Generating egs")
        if (not os.path.exists("{0}/den.fst".format(args.dir)) or
                not os.path.exists("{0}/normalization.fst".format(args.dir)) or
                not os.path.exists("{0}/tree".format(args.dir))):
            raise Exception("Chain egs generation expects {0}/den.fst, "
                            "{0}/normalization.fst and {0}/tree "
                            "to exist.".format(args.dir))
        # this is where get_egs.sh is called.
        chain_lib.generate_chain_egs(
            dir=args.dir, data=args.feat_dir,
            lat_dir=args.lat_dir, egs_dir=default_egs_dir,
            left_context=egs_left_context,
            right_context=egs_right_context,
            left_context_initial=egs_left_context_initial,
            right_context_final=egs_right_context_final,
            run_opts=run_opts,
            left_tolerance=args.left_tolerance,
            right_tolerance=args.right_tolerance,
            frame_subsampling_factor=args.frame_subsampling_factor,
            alignment_subsampling_factor=args.alignment_subsampling_factor,
            frames_per_eg_str=args.chunk_width,
            srand=args.srand,
            egs_opts=args.egs_opts,
            cmvn_opts=args.cmvn_opts,
            online_ivector_dir=args.online_ivector_dir,
            frames_per_iter=args.frames_per_iter,
            stage=args.egs_stage)

    if args.egs_dir is None:
        egs_dir = default_egs_dir
    else:
        egs_dir = args.egs_dir

    [egs_left_context, egs_right_context,
     frames_per_eg_str, num_archives] = (
         common_train_lib.verify_egs_dir(egs_dir, feat_dim,
                                         ivector_dim, ivector_id,
                                         egs_left_context, egs_right_context,
                                         egs_left_context_initial,
                                         egs_right_context_final))
    assert(args.chunk_width == frames_per_eg_str)
    num_archives_expanded = num_archives * args.frame_subsampling_factor

    if (args.num_jobs_final > num_archives_expanded):
        raise Exception('num_jobs_final cannot exceed the '
                        'expanded number of archives')

    # copy the properties of the egs to dir for
    # use during decoding
    logger.info("Copying the properties from {0} to {1}".format(egs_dir, args.dir))
    common_train_lib.copy_egs_properties_to_exp_dir(egs_dir, args.dir)

    if not os.path.exists('{0}/valid_diagnostic.cegs'.format(egs_dir)):
        if (not os.path.exists('{0}/valid_diagnostic.scp'.format(egs_dir))):
            raise Exception('Neither {0}/valid_diagnostic.cegs nor '
                            '{0}/valid_diagnostic.scp exist.'
                            'This script expects one of them.'.format(egs_dir))
        use_multitask_egs = True
    else:
        use_multitask_egs = False

    if ((args.stage <= -2) and (os.path.exists(args.dir+"/configs/init.config"))
            and (args.input_model is None)):
        logger.info('Computing the preconditioning matrix for input features')

        chain_lib.compute_preconditioning_matrix(
            args.dir, egs_dir, num_archives, run_opts,
            max_lda_jobs=args.max_lda_jobs,
            rand_prune=args.rand_prune,
            use_multitask_egs=use_multitask_egs)

    if (args.stage <= -1):
        logger.info("Preparing the initial acoustic model.")
        chain_lib.prepare_initial_acoustic_model(args.dir, run_opts,
                                                 input_model=args.input_model)

    with open("{0}/frame_subsampling_factor".format(args.dir), "w") as f:
        f.write(str(args.frame_subsampling_factor))

    # set num_iters so that as close as possible, we process the data
    # $num_epochs times, i.e. $num_iters*$avg_num_jobs) ==
    # $num_epochs*$num_archives, where
    # avg_num_jobs=(num_jobs_initial+num_jobs_final)/2.
    num_archives_to_process = int(args.num_epochs * num_archives_expanded)
    num_archives_processed = 0
    num_iters = ((num_archives_to_process * 2)
                 // (args.num_jobs_initial + args.num_jobs_final))

    # If do_final_combination is True, compute the set of models_to_combine.
    # Otherwise, models_to_combine will be none.
    if args.do_final_combination:
        models_to_combine = common_train_lib.get_model_combine_iters(
            num_iters, args.num_epochs,
            num_archives_expanded, args.max_models_combine,
            args.num_jobs_final)
    else:
        models_to_combine = None

    min_deriv_time = None
    max_deriv_time_relative = None
    if args.deriv_truncate_margin is not None:
        min_deriv_time = -args.deriv_truncate_margin - model_left_context
        max_deriv_time_relative = \
           args.deriv_truncate_margin + model_right_context

    logger.info("Training will run for {0} epochs = "
                "{1} iterations".format(args.num_epochs, num_iters))

    for iter in range(num_iters):
        if (args.exit_stage is not None) and (iter == args.exit_stage):
            logger.info("Exiting early due to --exit-stage {0}".format(iter))
            return
        current_num_jobs = int(0.5 + args.num_jobs_initial
                               + (args.num_jobs_final - args.num_jobs_initial)
                               * float(iter) / num_iters)

        if args.stage <= iter:
            model_file = "{dir}/{iter}.mdl".format(dir=args.dir, iter=iter)

            lrate = common_train_lib.get_learning_rate(iter, current_num_jobs,
                                                       num_iters,
                                                       num_archives_processed,
                                                       num_archives_to_process,
                                                       args.initial_effective_lrate,
                                                       args.final_effective_lrate)
            shrinkage_value = 1.0 - (args.proportional_shrink * lrate)
            if shrinkage_value <= 0.5:
                raise Exception("proportional-shrink={0} is too large, it gives "
                                "shrink-value={1}".format(args.proportional_shrink,
                                                          shrinkage_value))
            if args.shrink_value < shrinkage_value:
                shrinkage_value = (args.shrink_value
                                   if common_train_lib.should_do_shrinkage(
                                       iter, model_file,
                                       args.shrink_saturation_threshold)
                                   else shrinkage_value)

            percent = num_archives_processed * 100.0 / num_archives_to_process
            epoch = (num_archives_processed * args.num_epochs
                     / num_archives_to_process)
            shrink_info_str = ''
            if shrinkage_value != 1.0:
                shrink_info_str = 'shrink: {0:0.5f}'.format(shrinkage_value)
            logger.info("Iter: {0}/{1}    "
                        "Epoch: {2:0.2f}/{3:0.1f} ({4:0.1f}% complete)    "
                        "lr: {5:0.6f}    {6}".format(iter, num_iters - 1,
                                                     epoch, args.num_epochs,
                                                     percent,
                                                     lrate, shrink_info_str))

            chain_lib.train_one_iteration(
                dir=args.dir,
                iter=iter,
                srand=args.srand,
                egs_dir=egs_dir,
                num_jobs=current_num_jobs,
                num_archives_processed=num_archives_processed,
                num_archives=num_archives,
                learning_rate=lrate,
                dropout_edit_string=common_train_lib.get_dropout_edit_string(
                    args.dropout_schedule,
                    float(num_archives_processed) / num_archives_to_process,
                    iter),
                train_opts=' '.join(args.train_opts),
                shrinkage_value=shrinkage_value,
                num_chunk_per_minibatch_str=args.num_chunk_per_minibatch,
                apply_deriv_weights=args.apply_deriv_weights,
                min_deriv_time=min_deriv_time,
                max_deriv_time_relative=max_deriv_time_relative,
                l2_regularize=args.l2_regularize,
                xent_regularize=args.xent_regularize,
                leaky_hmm_coefficient=args.leaky_hmm_coefficient,
                momentum=args.momentum,
                max_param_change=args.max_param_change,
                shuffle_buffer_size=args.shuffle_buffer_size,
                frame_subsampling_factor=args.frame_subsampling_factor,
                run_opts=run_opts,
                backstitch_training_scale=args.backstitch_training_scale,
                backstitch_training_interval=args.backstitch_training_interval,
                use_multitask_egs=use_multitask_egs)

            if args.cleanup:
                # do a clean up everything but the last 2 models, under certain
                # conditions
                common_train_lib.remove_model(
                    args.dir, iter-2, num_iters, models_to_combine,
                    args.preserve_model_interval)

            if args.email is not None:
                reporting_iter_interval = num_iters * args.reporting_interval
                if iter % reporting_iter_interval == 0:
                    # lets do some reporting
                    [report, times, data] = (
                        nnet3_log_parse.generate_acc_logprob_report(
                            args.dir, "log-probability"))
                    message = report
                    subject = ("Update : Expt {dir} : "
                               "Iter {iter}".format(dir=args.dir, iter=iter))
                    common_lib.send_mail(message, subject, args.email)

        num_archives_processed = num_archives_processed + current_num_jobs

    if args.stage <= num_iters:
        if args.do_final_combination:
            logger.info("Doing final combination to produce final.mdl")
            chain_lib.combine_models(
                dir=args.dir, num_iters=num_iters,
                models_to_combine=models_to_combine,
                num_chunk_per_minibatch_str=args.num_chunk_per_minibatch,
                egs_dir=egs_dir,
                leaky_hmm_coefficient=args.leaky_hmm_coefficient,
                l2_regularize=args.l2_regularize,
                xent_regularize=args.xent_regularize,
                run_opts=run_opts,
                max_objective_evaluations=args.max_objective_evaluations,
                use_multitask_egs=use_multitask_egs)
        else:
            logger.info("Copying the last-numbered model to final.mdl")
            common_lib.force_symlink("{0}.mdl".format(num_iters),
                                     "{0}/final.mdl".format(args.dir))
            chain_lib.compute_train_cv_probabilities(
                dir=args.dir, iter=num_iters, egs_dir=egs_dir,
                l2_regularize=args.l2_regularize, xent_regularize=args.xent_regularize,
                leaky_hmm_coefficient=args.leaky_hmm_coefficient,
                run_opts=run_opts,
                use_multitask_egs=use_multitask_egs)
            common_lib.force_symlink("compute_prob_valid.{iter}.log"
                                     "".format(iter=num_iters),
                                     "{dir}/log/compute_prob_valid.final.log".format(
                                         dir=args.dir))

    if args.cleanup:
        logger.info("Cleaning up the experiment directory "
                    "{0}".format(args.dir))
        remove_egs = args.remove_egs
        if args.egs_dir is not None:
            # this egs_dir was not created by this experiment so we will not
            # delete it
            remove_egs = False

        # leave the last-two-numbered models, for diagnostic reasons.
        common_train_lib.clean_nnet_dir(
            args.dir, num_iters - 1, egs_dir,
            preserve_model_interval=args.preserve_model_interval,
            remove_egs=remove_egs)

    # do some reporting
    [report, times, data] = nnet3_log_parse.generate_acc_logprob_report(
        args.dir, "log-probability")
    if args.email is not None:
        common_lib.send_mail(report, "Update : Expt {0} : "
                                     "complete".format(args.dir), args.email)

    with open("{dir}/accuracy.report".format(dir=args.dir), "w") as f:
        f.write(report)

    common_lib.execute_command("steps/info/chain_dir_info.pl "
                               "{0}".format(args.dir))


def main():
    [args, run_opts] = get_args()
    try:
        train(args, run_opts)
        common_lib.wait_for_background_commands()
    except BaseException as e:
        # look for BaseException so we catch KeyboardInterrupt, which is
        # what we get when a background thread dies.
        if args.email is not None:
            message = ("Training session for experiment {dir} "
                       "died due to an error.".format(dir=args.dir))
            common_lib.send_mail(message, message, args.email)
        if not isinstance(e, KeyboardInterrupt):
            traceback.print_exc()
        sys.exit(1)

if __name__ == "__main__":
    main()
```



#### steps/nnet3/chain/get_egs.sh

该脚本从其他神经网络训练脚本中被调用，用于抽取训练样例和验证样例来训练和诊断chain系统。

该脚本运行的结果是产生一些样例，这些样例带有许多帧标签，再加上左右上下文，由frames_per_eg（默认是25）控制。由于CTC训练包含数据对齐，我们不能一帧一帧地训练。监督方法包括了时间对齐，尽管这是一种宽松的方式，对齐时每一个symbol可以出现在frame-range范围。

**代码经修改**

```shell
#!/bin/bash

# Copyright 2012-2015 Johns Hopkins University (Author: Daniel Povey).  Apache 2.0.
#
# This script, which will generally be called from other neural-net training
# scripts, extracts the training examples used to train the 'chain' system
# (and also the validation examples used for diagnostics), and puts them in
# separate archives.
#
# This script dumps egs with many frames of labels, controlled by the
# frames_per_eg config variable (default: 25), plus left and right context.
# Because CTC training involves alignment of data, we can't meaningfully train
# frame by frame.   The supervision approach involves the time alignment, though--
# it is just applied in a loose way, where each symbol can appear in the
# frame-range that it was in in the alignment, extended by a certain margin.
#


# Begin configuration section.
cmd=run.pl
frames_per_eg=25   # number of feature frames example (not counting added context).
                   # more->less disk space and less time preparing egs, but more
                   # I/O during training.
frames_overlap_per_eg=0  # number of supervised frames of overlap that we aim for per eg.
                  # can be useful to avoid wasted data if you're using --left-deriv-truncate
                  # and --right-deriv-truncate.
frame_subsampling_factor=3 # frames-per-second of features we train on divided
                           # by frames-per-second at output of chain model
alignment_subsampling_factor=3 # frames-per-second of input alignments divided
                               # by frames-per-second at output of chain model
left_context=4    # amount of left-context per eg (i.e. extra frames of input features
                  # not present in the output supervision).
right_context=4   # amount of right-context per eg.
constrained=true  # 'constrained=true' is the traditional setup; 'constrained=false'
                  # gives you the 'unconstrained' egs creation in which the time
                  # boundaries are not enforced inside chunks.

left_context_initial=-1    # if >=0, left-context for first chunk of an utterance
right_context_final=-1     # if >=0, right-context for last chunk of an utterance
compress=true   # set this to false to disable compression (e.g. if you want to see whether
                # results are affected).

num_utts_subset=300     # number of utterances in validation and training
                        # subsets used for shrinkage and diagnostics.
num_valid_egs_combine=0  # #validation examples for combination weights at the very end.
num_train_egs_combine=1000 # number of train examples for the above.
num_egs_diagnostic=400 # number of frames for "compute_prob" jobs
frames_per_iter=400000 # each iteration of training, see this many frames per
                       # job, measured at the sampling rate of the features
                       # used.  This is just a guideline; it will pick a number
                       # that divides the number of samples in the entire data.

right_tolerance=  # chain right tolerance == max label delay.
left_tolerance=

stage=0
max_jobs_run=15         # This should be set to the maximum number of nnet3-chain-get-egs jobs you are
                        # comfortable to run in parallel; you can increase it if your disk
                        # speed is greater and you have more machines.
max_shuffle_jobs_run=50  # the shuffle jobs now include the nnet3-chain-normalize-egs command,
                         # which is fairly CPU intensive, so we can run quite a few at once
                         # without overloading the disks.
srand=0     # rand seed for nnet3-chain-get-egs, nnet3-chain-copy-egs and nnet3-chain-shuffle-egs
online_ivector_dir=  # can be used if we are including speaker information as iVectors.
cmvn_opts=  # can be used for specifying CMVN options, if feature type is not lda (if lda,
            # it doesn't make sense to use different options than were used as input to the
            # LDA transform).  This is used to turn off CMVN in the online-nnet experiments.
lattice_lm_scale=     # If supplied, the graph/lm weight of the lattices will be
                      # used (with this scale) in generating supervisions
                      # This is 0 by default for conventional supervised training,
                      # but may be close to 1 for the unsupervised part of the data
                      # in semi-supervised training. The optimum is usually
                      # 0.5 for unsupervised data.
lattice_prune_beam=         # If supplied, the lattices will be pruned to this beam,
                            # before being used to get supervisions.
acwt=0.1   # For pruning
deriv_weights_scp=
generate_egs_scp=false

echo "$0 $@"  # Print the command line for logging

if [ -f path.sh ]; then . ./path.sh; fi
. parse_options.sh || exit 1;


if [ $# != 4 ]; then
  echo "Usage: $0 [opts] <data> <chain-dir> <lattice-dir> <egs-dir>"
  echo " e.g.: $0 data/train exp/tri4_nnet exp/tri3_lats exp/tri4_nnet/egs"
  echo ""
  echo "From <chain-dir>, 0.trans_mdl (the transition-model), tree (the tree)"
  echo "and normalization.fst (the normalization FST, derived from the denominator FST)"
  echo "are read."
  echo ""
  echo "Main options (for others, see top of script file)"
  echo "  --config <config-file>                           # config file containing options"
  echo "  --max-jobs-run <max-jobs-run>                    # The maximum number of jobs you want to run in"
  echo "                                                   # parallel (increase this only if you have good disk and"
  echo "                                                   # network speed).  default=6"
  echo "  --cmd (utils/run.pl;utils/queue.pl <queue opts>) # how to run jobs."
  echo "  --frames-per-iter <#samples;400000>              # Number of frames of data to process per iteration, per"
  echo "                                                   # process."
  echo "  --frame-subsampling-factor <factor;3>            # factor by which num-frames at nnet output is reduced "
  echo "  --frames-per-eg <frames;25>                      # number of supervised frames per eg on disk"
  echo "  --frames-overlap-per-eg <frames;25>              # number of supervised frames of overlap between egs"
  echo "  --left-context <int;4>                           # Number of frames on left side to append for feature input"
  echo "  --right-context <int;4>                          # Number of frames on right side to append for feature input"
  echo "  --left-context-initial <int;-1>                  # If >= 0, left-context for first chunk of an utterance"
  echo "  --right-context-final <int;-1>                   # If >= 0, right-context for last chunk of an utterance"
  echo "  --num-egs-diagnostic <#frames;4000>              # Number of egs used in computing (train,valid) diagnostics"
  echo "  --num-valid-egs-combine <#frames;10000>          # Number of egs used in getting combination weights at the"
  echo "                                                   # very end."
  echo "  --lattice-lm-scale <float>                       # If supplied, the graph/lm weight of the lattices will be "
  echo "                                                   # used (with this scale) in generating supervisions"
  echo "  --lattice-prune-beam <float>                     # If supplied, the lattices will be pruned to this beam, "
  echo "                                                   # before being used to get supervisions."
  echo "  --acwt <float;0.1>                               # Acoustic scale -- affects pruning"
  echo "  --deriv-weights-scp <str>                        # If supplied, adds per-frame weights to the supervision."
  echo "  --generate-egs-scp <bool;false>                  # Generates scp files -- Required if the egs will be "
  echo "                                                   # used for multilingual/multitask training."
  echo "  --stage <stage|0>                                # Used to run a partially-completed training process from somewhere in"
  echo "                                                   # the middle."

  exit 1;
fi

data=$1
chaindir=$2
latdir=$3
dir=$4

# Check some files.
[ ! -z "$online_ivector_dir" ] && \
  extra_files="$online_ivector_dir/ivector_online.scp $online_ivector_dir/ivector_period"

for f in $data/feats.scp $latdir/lat.1.gz $latdir/final.mdl \
         $chaindir/{0.trans_mdl,tree,normalization.fst} $extra_files; do
  [ ! -f $f ] && echo "$0: no such file $f" && exit 1;
done

nj=$(cat $latdir/num_jobs) || exit 1
if [ -f $latdir/per_utt ]; then
  sdata=$data/split${nj}utt
  utils/split_data.sh --per-utt $data $nj
else
  sdata=$data/split$nj
  utils/split_data.sh $data $nj
fi

mkdir -p $dir/log $dir/info

# Get list of validation utterances.
frame_shift=$(utils/data/get_frame_shift.sh $data) || exit 1

if [ -f $data/utt2uniq ]; then
  # Must hold out all augmented versions of the same utterance.
  echo "$0: File $data/utt2uniq exists, so ensuring the hold-out set" \
       "includes all perturbed versions of the same source utterance."
  #utils/utt2spk_to_spk2utt.pl $data/utt2uniq 2>/dev/null | \
      #utils/shuffle_list.pl 2>/dev/null | \
    #awk -v max_utt=$num_utts_subset '{
        f#or (n=2;n<=NF;n++) print $n;
        #printed += NF-1;
        #if (printed >= max_utt) nextfile; }' |
   #sort > $dir/valid_uttlist
  
  utils/utt2spk_to_spk2utt.pl $data/utt2uniq 2>/dev/null | \
      utils/shuffle_list.pl 2>/dev/null | \
    awk -v max_utt=$num_utts_subset '{
    	if (printed >= max_utt) next;
        for (n=2;n<=NF;n++) print $n;
        printed += NF-1; }' |
    sort > $dir/valid_uttlist
else
  awk '{print $1}' $data/utt2spk | \
    utils/shuffle_list.pl 2>/dev/null | \
    head -$num_utts_subset > $dir/valid_uttlist
fi
len_valid_uttlist=$(wc -l < $dir/valid_uttlist)

awk '{print $1}' $data/utt2spk | \
   utils/filter_scp.pl --exclude $dir/valid_uttlist | \
   utils/shuffle_list.pl 2>/dev/null | \
   head -$num_utts_subset > $dir/train_subset_uttlist
len_trainsub_uttlist=$(wc -l <$dir/train_subset_uttlist)

if [[ $len_valid_uttlist -lt $num_utts_subset ||
      $len_trainsub_uttlist -lt $num_utts_subset ]]; then
  echo "$0: Number of utterances is very small. Please check your data." && exit 1;
fi

echo "$0: Holding out $len_valid_uttlist utterances in validation set and" \
     "$len_trainsub_uttlist in training diagnostic set, out of total" \
     "$(wc -l < $data/utt2spk)."


echo "$0: creating egs.  To ensure they are not deleted later you can do:  touch $dir/.nodelete"

## Set up features.
echo "$0: feature type is raw"
feats="ark,s,cs:utils/filter_scp.pl --exclude $dir/valid_uttlist $sdata/JOB/feats.scp | apply-cmvn $cmvn_opts --utt2spk=ark:$sdata/JOB/utt2spk scp:$sdata/JOB/cmvn.scp scp:- ark:- |"
valid_feats="ark,s,cs:utils/filter_scp.pl $dir/valid_uttlist $data/feats.scp | apply-cmvn $cmvn_opts --utt2spk=ark:$data/utt2spk scp:$data/cmvn.scp scp:- ark:- |"
train_subset_feats="ark,s,cs:utils/filter_scp.pl $dir/train_subset_uttlist $data/feats.scp | apply-cmvn $cmvn_opts --utt2spk=ark:$data/utt2spk scp:$data/cmvn.scp scp:- ark:- |"
echo $cmvn_opts >$dir/cmvn_opts # caution: the top-level nnet training script should copy this to its own dir now.

tree-info $chaindir/tree | grep num-pdfs | awk '{print $2}' > $dir/info/num_pdfs || exit 1

if [ ! -z "$online_ivector_dir" ]; then
  ivector_dim=$(feat-to-dim scp:$online_ivector_dir/ivector_online.scp -) || exit 1;
  echo $ivector_dim > $dir/info/ivector_dim
  steps/nnet2/get_ivector_id.sh $online_ivector_dir > $dir/info/final.ie.id || exit 1
  ivector_period=$(cat $online_ivector_dir/ivector_period) || exit 1;
  ivector_opts="--online-ivectors=scp:$online_ivector_dir/ivector_online.scp --online-ivector-period=$ivector_period"
else
  ivector_opts=""
  echo 0 >$dir/info/ivector_dim
fi

if [ $stage -le 1 ]; then
  echo "$0: working out number of frames of training data"
  num_frames=$(steps/nnet2/get_num_frames.sh $data)
  echo $num_frames > $dir/info/num_frames
  echo "$0: working out feature dim"
  feats_one="$(echo $feats | sed s/JOB/1/g)"
  if ! feat_dim=$(feat-to-dim "$feats_one" - 2>/dev/null); then
    echo "Command failed (getting feature dim): feat-to-dim \"$feats_one\""
    exit 1
  fi
  echo $feat_dim > $dir/info/feat_dim
else
  num_frames=$(cat $dir/info/num_frames) || exit 1;
  feat_dim=$(cat $dir/info/feat_dim) || exit 1;
fi

# the + 1 is to round up, not down... we assume it doesn't divide exactly.
num_archives=$[$num_frames/$frames_per_iter+1]

# We may have to first create a smaller number of larger archives, with number
# $num_archives_intermediate, if $num_archives is more than the maximum number
# of open filehandles that the system allows per process (ulimit -n).
# This sometimes gives a misleading answer as GridEngine sometimes changes the
# limit, so we limit it to 512.
max_open_filehandles=$(ulimit -n) || exit 1
[ $max_open_filehandles -gt 512 ] && max_open_filehandles=512
num_archives_intermediate=$num_archives
archives_multiple=1
while [ $[$num_archives_intermediate+4] -gt $max_open_filehandles ]; do
  archives_multiple=$[$archives_multiple+1]
  num_archives_intermediate=$[$num_archives/$archives_multiple] || exit 1;
done
# now make sure num_archives is an exact multiple of archives_multiple.
num_archives=$[$archives_multiple*$num_archives_intermediate] || exit 1;

echo $num_archives >$dir/info/num_archives
echo $frames_per_eg >$dir/info/frames_per_eg
# Work out the number of egs per archive
egs_per_archive=$[$num_frames/($frames_per_eg*$num_archives)] || exit 1;
! [ $egs_per_archive -le $frames_per_iter ] && \
  echo "$0: script error: egs_per_archive=$egs_per_archive not <= frames_per_iter=$frames_per_iter" \
  && exit 1;

echo $egs_per_archive > $dir/info/egs_per_archive

echo "$0: creating $num_archives archives, each with $egs_per_archive egs, with"
echo "$0:   $frames_per_eg labels per example, and (left,right) context = ($left_context,$right_context)"
if [ $left_context_initial -ge 0 ] || [ $right_context_final -ge 0 ]; then
  echo "$0:   ... and (left-context-initial,right-context-final) = ($left_context_initial,$right_context_final)"
fi


if [ -e $dir/storage ]; then
  # Make soft links to storage directories, if distributing this way..  See
  # utils/create_split_dir.pl.
  echo "$0: creating data links"
  utils/create_data_link.pl $(for x in $(seq $num_archives); do echo $dir/cegs.$x.ark; done)
  for x in $(seq $num_archives_intermediate); do
    utils/create_data_link.pl $(for y in $(seq $nj); do echo $dir/cegs_orig.$y.$x.ark; done)
  done
fi

egs_opts="--left-context=$left_context --right-context=$right_context --num-frames=$frames_per_eg --frame-subsampling-factor=$frame_subsampling_factor --compress=$compress"
[ $left_context_initial -ge 0 ] && egs_opts="$egs_opts --left-context-initial=$left_context_initial"
[ $right_context_final -ge 0 ] && egs_opts="$egs_opts --right-context-final=$right_context_final"

[ ! -z "$deriv_weights_scp" ] && egs_opts="$egs_opts --deriv-weights-rspecifier=scp:$deriv_weights_scp"

chain_supervision_all_opts="--lattice-input=true --frame-subsampling-factor=$alignment_subsampling_factor"
[ ! -z $right_tolerance ] && \
  chain_supervision_all_opts="$chain_supervision_all_opts --right-tolerance=$right_tolerance"

[ ! -z $left_tolerance ] && \
  chain_supervision_all_opts="$chain_supervision_all_opts --left-tolerance=$left_tolerance"

if ! $constrained; then
  chain_supervision_all_opts="$chain_supervision_all_opts --convert-to-pdfs=false"
  trans_mdl_opt=--transition-model=$chaindir/0.trans_mdl
else
  trans_mdl_opt=
fi


lats_rspecifier="ark:gunzip -c $latdir/lat.JOB.gz |"
if [ ! -z $lattice_prune_beam ]; then
  if [ "$lattice_prune_beam" == "0" ] || [ "$lattice_prune_beam" == "0.0" ]; then
    lats_rspecifier="$lats_rspecifier lattice-1best --acoustic-scale=$acwt ark:- ark:- |"
  else
    lats_rspecifier="$lats_rspecifier lattice-prune --acoustic-scale=$acwt --beam=$lattice_prune_beam ark:- ark:- |"
  fi
fi

normalization_fst_scale=1.0

if [ ! -z "$lattice_lm_scale" ]; then
  chain_supervision_all_opts="$chain_supervision_all_opts --lm-scale=$lattice_lm_scale"

  normalization_fst_scale=$(perl -e "
  if ($lattice_lm_scale >= 1.0 || $lattice_lm_scale < 0) {
    print STDERR \"Invalid --lattice-lm-scale $lattice_lm_scale\";
    exit(1);
  }
  print (1.0 - $lattice_lm_scale);") || exit 1
fi

echo $left_context > $dir/info/left_context
echo $right_context > $dir/info/right_context
echo $left_context_initial > $dir/info/left_context_initial
echo $right_context_final > $dir/info/right_context_final

if [ $stage -le 2 ]; then
  echo "$0: Getting validation and training subset examples in background."
  rm $dir/.error 2>/dev/null

  (
    $cmd --max-jobs-run 6 JOB=1:$nj $dir/log/lattice_copy.JOB.log \
      lattice-copy --include="cat $dir/valid_uttlist $dir/train_subset_uttlist |" --ignore-missing \
      "$lats_rspecifier" \
      ark,scp:$dir/lat_special.JOB.ark,$dir/lat_special.JOB.scp || exit 1

    for id in $(seq $nj); do cat $dir/lat_special.$id.scp; done > $dir/lat_special.scp

    $cmd $dir/log/create_valid_subset.log \
      utils/filter_scp.pl $dir/valid_uttlist $dir/lat_special.scp \| \
      lattice-align-phones --replace-output-symbols=true $latdir/final.mdl scp:- ark:- \| \
      chain-get-supervision $chain_supervision_all_opts $chaindir/tree $chaindir/0.trans_mdl \
        ark:- ark:- \| \
      nnet3-chain-get-egs $ivector_opts --srand=$srand \
         $egs_opts --normalization-fst-scale=$normalization_fst_scale \
         $trans_mdl_opt $chaindir/normalization.fst \
        "$valid_feats" ark,s,cs:- "ark:$dir/valid_all.cegs" || exit 1
    $cmd $dir/log/create_train_subset.log \
      utils/filter_scp.pl $dir/train_subset_uttlist $dir/lat_special.scp \| \
      lattice-align-phones --replace-output-symbols=true $latdir/final.mdl scp:- ark:- \| \
      chain-get-supervision $chain_supervision_all_opts \
        $chaindir/tree $chaindir/0.trans_mdl ark:- ark:- \| \
      nnet3-chain-get-egs $ivector_opts --srand=$srand \
        $egs_opts --normalization-fst-scale=$normalization_fst_scale \
        $trans_mdl_opt $chaindir/normalization.fst \
        "$train_subset_feats" ark,s,cs:- "ark:$dir/train_subset_all.cegs" || exit 1
    sleep 5  # wait for file system to sync.
    echo "$0: Getting subsets of validation examples for diagnostics and combination."
    if $generate_egs_scp; then
      valid_diagnostic_output="ark,scp:$dir/valid_diagnostic.cegs,$dir/valid_diagnostic.scp"
      train_diagnostic_output="ark,scp:$dir/train_diagnostic.cegs,$dir/train_diagnostic.scp"
    else
      valid_diagnostic_output="ark:$dir/valid_diagnostic.cegs"
      train_diagnostic_output="ark:$dir/train_diagnostic.cegs"
    fi
    $cmd $dir/log/create_valid_subset_combine.log \
      nnet3-chain-subset-egs --n=$num_valid_egs_combine ark:$dir/valid_all.cegs \
      ark:$dir/valid_combine.cegs || exit 1
    $cmd $dir/log/create_valid_subset_diagnostic.log \
      nnet3-chain-subset-egs --n=$num_egs_diagnostic ark:$dir/valid_all.cegs \
      $valid_diagnostic_output || exit 1

    $cmd $dir/log/create_train_subset_combine.log \
      nnet3-chain-subset-egs --n=$num_train_egs_combine ark:$dir/train_subset_all.cegs \
      ark:$dir/train_combine.cegs || exit 1
    $cmd $dir/log/create_train_subset_diagnostic.log \
      nnet3-chain-subset-egs --n=$num_egs_diagnostic ark:$dir/train_subset_all.cegs \
      $train_diagnostic_output || exit 1
    sleep 5  # wait for file system to sync.
    if $generate_egs_scp; then
      cat $dir/valid_combine.cegs $dir/train_combine.cegs | \
        nnet3-chain-copy-egs ark:- ark,scp:$dir/combine.cegs,$dir/combine.scp
    else
      cat $dir/valid_combine.cegs $dir/train_combine.cegs > $dir/combine.cegs
    fi

    for f in $dir/{combine,train_diagnostic,valid_diagnostic}.cegs; do
      [ ! -s $f ] && echo "$0: No examples in file $f" && exit 1;
    done
    rm $dir/valid_all.cegs $dir/train_subset_all.cegs $dir/{train,valid}_combine.cegs
  ) || touch $dir/.error &
fi

if [ $stage -le 4 ]; then
  # create cegs_orig.*.*.ark; the first index goes to $nj,
  # the second to $num_archives_intermediate.

  egs_list=
  for n in $(seq $num_archives_intermediate); do
    egs_list="$egs_list ark:$dir/cegs_orig.JOB.$n.ark"
  done
  echo "$0: Generating training examples on disk"

  # The examples will go round-robin to egs_list.  Note: we omit the
  # 'normalization.fst' argument while creating temporary egs: the phase of egs
  # preparation that involves the normalization FST is quite CPU-intensive and
  # it's more convenient to do it later, in the 'shuffle' stage.  Otherwise to
  # make it efficient we need to use a large 'nj', like 40, and in that case
  # there can be too many small files to deal with, because the total number of
  # files is the product of 'nj' by 'num_archives_intermediate', which might be
  # quite large.

  $cmd --max-jobs-run $max_jobs_run JOB=1:$nj $dir/log/get_egs.JOB.log \
    lattice-align-phones --replace-output-symbols=true $latdir/final.mdl \
      "$lats_rspecifier" ark:- \| \
    chain-get-supervision $chain_supervision_all_opts \
      $chaindir/tree $chaindir/0.trans_mdl ark:- ark:- \| \
    nnet3-chain-get-egs $ivector_opts --srand=\$[JOB+$srand] $egs_opts \
      --num-frames-overlap=$frames_overlap_per_eg $trans_mdl_opt \
     "$feats" ark,s,cs:- ark:- \| \
    nnet3-chain-copy-egs --random=true --srand=\$[JOB+$srand] ark:- $egs_list || exit 1;
fi

if [ -f $dir/.error ]; then
  echo "$0: Error detected while creating train/valid egs" && exit 1
fi

if [ $stage -le 5 ]; then
  echo "$0: recombining and shuffling order of archives on disk"
  # combine all the "egs_orig.*.JOB.scp" (over the $nj splits of the data) and
  # shuffle the order, writing to the egs.JOB.ark

  # the input is a concatenation over the input jobs.
  egs_list=
  for n in $(seq $nj); do
    egs_list="$egs_list $dir/cegs_orig.$n.JOB.ark"
  done

  if [ $archives_multiple == 1 ]; then # normal case.
    if $generate_egs_scp; then
      output_archive="ark,scp:$dir/cegs.JOB.ark,$dir/cegs.JOB.scp"
    else
      output_archive="ark:$dir/cegs.JOB.ark"
    fi
    $cmd --max-jobs-run $max_shuffle_jobs_run --mem 8G \
      JOB=1:$num_archives_intermediate $dir/log/shuffle.JOB.log \
      nnet3-chain-normalize-egs --normalization-fst-scale=$normalization_fst_scale \
        $chaindir/normalization.fst "ark:cat $egs_list|" ark:- \| \
      nnet3-chain-shuffle-egs --srand=\$[JOB+$srand] ark:- $output_archive || exit 1;

    if $generate_egs_scp; then
      #concatenate cegs.JOB.scp in single cegs.scp
      for j in $(seq $num_archives_intermediate); do
        cat $dir/cegs.$j.scp || exit 1;
      done > $dir/cegs.scp || exit 1;
      for f in $dir/cegs.*.scp; do rm $f; done
    fi
  else
    # we need to shuffle the 'intermediate archives' and then split into the
    # final archives.  we create soft links to manage this splitting, because
    # otherwise managing the output names is quite difficult (and we don't want
    # to submit separate queue jobs for each intermediate archive, because then
    # the --max-jobs-run option is hard to enforce).
    if $generate_egs_scp; then
      output_archives="$(for y in $(seq $archives_multiple); do echo ark,scp:$dir/cegs.JOB.$y.ark,$dir/cegs.JOB.$y.scp; done)"
    else
      output_archives="$(for y in $(seq $archives_multiple); do echo ark:$dir/cegs.JOB.$y.ark; done)"
    fi
    for x in $(seq $num_archives_intermediate); do
      for y in $(seq $archives_multiple); do
        archive_index=$[($x-1)*$archives_multiple+$y]
        # egs.intermediate_archive.{1,2,...}.ark will point to egs.archive.ark
        ln -sf cegs.$archive_index.ark $dir/cegs.$x.$y.ark || exit 1
      done
    done
    $cmd --max-jobs-run $max_shuffle_jobs_run --mem 8G \
      JOB=1:$num_archives_intermediate $dir/log/shuffle.JOB.log \
      nnet3-chain-normalize-egs --normalization-fst-scale=$normalization_fst_scale \
        $chaindir/normalization.fst "ark:cat $egs_list|" ark:- \| \
      nnet3-chain-shuffle-egs --srand=\$[JOB+$srand] ark:- ark:- \| \
      nnet3-chain-copy-egs ark:- $output_archives || exit 1;

    if $generate_egs_scp; then
      #concatenate cegs.JOB.scp in single cegs.scp
      rm -rf $dir/cegs.scp
      for j in $(seq $num_archives_intermediate); do
        for y in $(seq $archives_multiple); do
          cat $dir/cegs.$j.$y.scp || exit 1;
        done
      done > $dir/cegs.scp || exit 1;
      for f in $dir/cegs.*.*.scp; do rm $f; done
    fi
  fi
fi

wait
if [ -f $dir/.error ]; then
  echo "$0: Error detected while creating train/valid egs" && exit 1
fi

if [ $stage -le 6 ]; then
  echo "$0: Removing temporary archives, alignments and lattices"
  (
    cd $dir
    for f in $(ls -l . | grep 'cegs_orig' | awk '{ X=NF-1; Y=NF-2; if ($X == "->")  print $Y, $NF; }'); do rm $f; done
    # the next statement removes them if we weren't using the soft links to a
    # 'storage' directory.
    rm cegs_orig.*.ark 2>/dev/null
  )
  if ! $generate_egs_scp && [ $archives_multiple -gt 1 ]; then
    # there are some extra soft links that we should delete.
    for f in $dir/cegs.*.*.ark; do rm $f; done
  fi
  rm $dir/ali.{ark,scp} 2>/dev/null
  rm $dir/lat_special.*.{ark,scp} 2>/dev/null
fi

echo "$0: Finished preparing training examples"
```



### wer_hyp_filter

```perl
#!/usr/bin/env perl

@filters=('[NOISE]','[LAUGHTER]','[VOCALIZED-NOISE]','<UNK>','%HESITATION');

foreach $w (@filters) {
  $bad{$w} = 1;
}

while(<STDIN>) {
  @A  = split(" ", $_);
  $id = shift @A;
  print "$id ";
  foreach $a (@A) {
    if (!defined $bad{$a}) {
      print "$a ";
    }
  }
  print "\n";
}
```



### wer_output_filter

```perl
#!/usr/bin/env perl
# Copyright 2012-2014  Johns Hopkins University (Author: Yenda Trmal)
# Apache 2.0
use utf8;

use open qw(:encoding(utf8));
binmode STDIN, ":utf8";
binmode STDOUT, ":utf8";
binmode STDERR, ":utf8";

while (<>) {
  @F = split " ";
  print $F[0] . " "; 
  foreach $s (@F[1..$#F]) {
    if (($s =~ /\[.*\]/) || ($s =~ /\<.*\>/) || ($s =~ "!SIL")) {
      print "";
    } else {
      print "$s"
    }
    print " ";
  }
  print "\n";
}
```



### wer_ref_filter

```perl
#!/usr/bin/env perl

@filters=('[NOISE]','[LAUGHTER]','[VOCALIZED-NOISE]','<UNK>','%HESITATION');

foreach $w (@filters) {
  $bad{$w} = 1;
}

while(<STDIN>) {
  @A  = split(" ", $_);
  $id = shift @A;
  print "$id ";
  foreach $a (@A) {
    if (!defined $bad{$a}) {
      print "$a ";
    }
  }
  print "\n";
}
```



## cmd.sh

单机跑实验需要把queue.pl修改成run.pl，并把这些环境变量export出去。

```shell
export train_cmd="run.pl"
export decode_cmd="run.pl"
export mkgraph_cmd="run.pl"
```

## path.sh

```shell
export KALDI_ROOT=`pwd`/../../
[ -f $KALDI_ROOT/tools/env.sh ] && . $KALDI_ROOT/tools/env.sh
export PATH=$PWD/utils/:$KALDI_ROOT/tools/openfst/bin:$PWD:$PATH
[ ! -f $KALDI_ROOT/tools/config/common_path.sh ] && echo >&2 "The standard file $KALDI_ROOT/tools/config/common_path.sh is not present -> Exit!" && exit 1
. $KALDI_ROOT/tools/config/common_path.sh
export LC_ALL=C
```



## run.sh

s5/run.sh包含了在这个数据集上所有的训练步骤，包括数据预处理、训练以及测试gmm/dnn/lstm/blstm/tdnn等模型、实验结果统计在内的各种脚本。理论上来说只要相关环境配置正确，运行run.sh就能完成整个训练步骤。



### 版权说明

在run.sh开头，通常会介绍一下该代码工程的版权说明，如

```shell
# Copyright 2018 Beijing DataTang Tech. Co. Ltd. (Authors: )
#           2017 Jiayu Du
#           2017 Hao Zheng
# Apache 2.0
```



### 环境配置

接着，需要激活必要的环境变量，分别是path.sh中声明的kaldi运行环境变量和cmd.sh中声明的单机集群配置。

```shell
. ./cmd.sh ## You'll want to change cmd.sh to something that will work on your system.
           ## This relates to the queue.
. ./path.sh
```



### 数据下载与解压

在run.sh最开始的部分主要是一些数据的预处理步骤， 在运行它之前首先把数据集放在某个固定的地方，然后切换到run.sh所在的路径。在我的计算机上，它位于~/corpus/aidatatang_200zh/。也可以从openSLR上下载并解压。

```shell
# corpus directory and download URL
data=/home/ldf/corpus/aidatatang_200zh
data_url=www.openslr.org/resources/62

# You can obtain the database by uncommting the follwing lines
#[ -d $data ] || mkdir -p $data || exit 1;
local/download_and_untar.sh $data $data_url data_aidatatang || exit 1;
```



### 数据准备阶段：

手动生成映射文件，存放在data/{local}/{train,dev,test}文件夹中。

text文件包含每条语音对应的转录文本。每一行的格式为 uttid \t transcription，

wav.scp记录语音位置的索引文件，格式为 <recording-id> <extended-filename>。注意：wav文件必须是单通道的，否则必须通过sox命令提取其中的单个通道。

utt2spk指明了每句语音由哪位speaker发出。格式为<utterance-id> <speaker-id>。

spk2utt跟utt2spk逆过来。

```shell
# Data Preparation: generate text, wav.scp, utt2spk, spk2utt
local/aidatatang_data_prep.sh $data/corpus $data/transcript || exit 1;
```

这里，将数据集处理成aishell1数据集的格式，依照aishell1数据准备的脚本处理数据。【hkust示例使用mmseg进行文本分词，并且做了segment】



### 准备词典：

根据转录文本的分词后产生的词汇，分别生成中英文词典，最后合并中英文词典生成大词典。过程文件存放在data/local/dict文件夹下。

```shell
# Lexicon Preparation: build a large lexicon that invovles words in both the training and decoding
local/aidatatang_prepare_dict.sh || exit 1;
```

这里使用的是hkust生成中英文混合词典的脚本。具体实现步骤如下：

- 根据text抽取语料中全部词汇
- 分离中英文词汇
- 下载并格式化CMU英文词典（其中全是大写英文）
- 分别抽取in-vocab和oov词汇
- 利用g2p模型生成OOV词典
- 将CMU音素转成拼音音素，使用conf/cmu2pinyin,conf/pinyin2cmu,生成最终版英文词汇词典
- 下载cedit词典（免费维护的中-英词典）
- 生成中文in-vocab和oov词汇
- 将汉语拼音转成CMU格式
- 合并中英文词典



### 准备语言模型

准备triphone模型的决策树question集合以及编译Transducer L。Transducer L 用于将音素序列映射成词语序列。

```shell
# Prepare Language Stuff
# Phone Sets, questions, L compilation
utils/prepare_lang.sh --position-dependent-phones false data/local/dict "<UNK>" data/local/lang data/lang || exit 1;
```



训练3-gram语言模型，由这个模型生成Transducer G，然后将L和G拼接起来，产生Transducer LG。Transducer LG可以用来对给定的音素序列，使用3-gram语言模型找出最可能的词语序列。[参照hkust示例脚本]

```shell
# LM training
local/aidatatang_train_lms2.sh || exit 1;
# G compilation, check LG composition
local/aidatatang_format_data.sh
```



### 提取特征

生成特征文件：

```shell
# Now make MFCC plus pitch features.
# mfccdir should be some place with a largish disk where you
# want to store MFCC features.
mfccdir=mfcc
for x in train dev test; do
  steps/make_mfcc_pitch.sh --cmd "$train_cmd" --nj 10 data/$x exp/make_mfcc/$x $mfccdir || exit 1;
  steps/compute_cmvn_stats.sh data/$x exp/make_mfcc/$x $mfccdir || exit 1;
  utils/fix_data_dir.sh data/$x || exit 1;
done
```

feats.scp，格式为<utterance-id> <extended-filename-of-features>，每一个特征都包含一个矩阵，这里矩阵的维度为16。

```shell
steps/make_mfcc_pitch.sh --cmd "$train_cmd" --nj 10 data/$x exp/make_mfcc/$x $mfccdir
```

cmvn.scp倒谱均值和协方差正则化，它由speaker-id索引，为每一位话者计算特征，统计结果记录在矩阵里，规模为2x17。

```shell
steps/compute_cmvn_stats.sh data/train exp/make_mfcc/train $mfccdir
```

由于数据准备阶段产生的错误会影响后续程序的运行，因此，这里还需要检查data目录的格式是否正确。

```shell
utils/validate_data_dir.sh data/train
```

以下脚本会修复排序的错误，并删除缺失必需数据（比如特征或转录文本）的语音。

```shell
utils/fix_data_dir.sh data/train
```

经过数据准备阶段，这里会产生两个目录，一个和“数据”有关（比如data/train），另一个和“语言”有关（比如data/lang）。“数据”相关部分和你所使用的的数据集有关，“语言”部分更倾向于语言本身，比如词典、音素集和Kaldi需要的各种音素相关信息。



### GMM-HMM模型训练

单音子训练、解码、对齐：

```shell
steps/train_mono.sh --cmd "$train_cmd" --nj 10 \
  data/train data/lang exp/mono || exit 1;
  # Monophone decoding
utils/mkgraph.sh data/lang_test exp/mono exp/mono/graph || exit 1;
steps/decode.sh --cmd "$decode_cmd" --config conf/decode.config --nj 10 \
  exp/mono/graph data/dev exp/mono/decode_dev

steps/decode.sh --cmd "$decode_cmd" --config conf/decode.config --nj 10 \
  exp/mono/graph data/test exp/mono/decode_test

# Get alignments from monophone system.
steps/align_si.sh --cmd "$train_cmd" --nj 10 \
  data/train data/lang exp/mono exp/mono_ali || exit 1;
```

三音子训练、解码、对齐：

```shell
# train tri1 [first triphone pass]
steps/train_deltas.sh --cmd "$train_cmd" \
 2500 20000 data/train data/lang exp/mono_ali exp/tri1 || exit 1;

# decode tri1
utils/mkgraph.sh data/lang_test exp/tri1 exp/tri1/graph || exit 1;
steps/decode.sh --cmd "$decode_cmd" --config conf/decode.config --nj 10 \
  exp/tri1/graph data/dev exp/tri1/decode_dev
steps/decode.sh --cmd "$decode_cmd" --config conf/decode.config --nj 10 \
  exp/tri1/graph data/test exp/tri1/decode_test

# align tri1
steps/align_si.sh --cmd "$train_cmd" --nj 10 \
  data/train data/lang exp/tri1 exp/tri1_ali || exit 1;
```

```shell
# train tri2 [delta+delta-deltas]
steps/train_deltas.sh --cmd "$train_cmd" \
 2500 20000 data/train data/lang exp/tri1_ali exp/tri2 || exit 1;

# decode tri2
utils/mkgraph.sh data/lang_test exp/tri2 exp/tri2/graph
steps/decode.sh --cmd "$decode_cmd" --config conf/decode.config --nj 10 \
  exp/tri2/graph data/dev exp/tri2/decode_dev
steps/decode.sh --cmd "$decode_cmd" --config conf/decode.config --nj 10 \
  exp/tri2/graph data/test exp/tri2/decode_test

#align tri2
steps/align_si.sh --cmd "$train_cmd" --nj 10 \
  data/train data/lang exp/tri2 exp/tri2_ali || exit 1;
```

```shell
# Train tri3a, which is LDA+MLLT,
steps/train_lda_mllt.sh --cmd "$train_cmd" \
 2500 20000 data/train data/lang exp/tri2_ali exp/tri3a || exit 1;

utils/mkgraph.sh data/lang_test exp/tri3a exp/tri3a/graph || exit 1;
steps/decode.sh --cmd "$decode_cmd" --nj 10 --config conf/decode.config \
  exp/tri3a/graph data/dev exp/tri3a/decode_dev
steps/decode.sh --cmd "$decode_cmd" --nj 10 --config conf/decode.config \
  exp/tri3a/graph data/test exp/tri3a/decode_test

# From now, we start building a more serious system (with SAT), and we'll
# do the alignment with fMLLR.
steps/align_fmllr.sh --cmd "$train_cmd" --nj 10 \
  data/train data/lang exp/tri3a exp/tri3a_ali || exit 1;
```

```shell
steps/train_sat.sh --cmd "$train_cmd" \
  2500 20000 data/train data/lang exp/tri3a_ali exp/tri4a || exit 1;

utils/mkgraph.sh data/lang_test exp/tri4a exp/tri4a/graph
steps/decode_fmllr.sh --cmd "$decode_cmd" --nj 10 --config conf/decode.config \
  exp/tri4a/graph data/dev exp/tri4a/decode_dev
steps/decode_fmllr.sh --cmd "$decode_cmd" --nj 10 --config conf/decode.config \
  exp/tri4a/graph data/test exp/tri4a/decode_test

steps/align_fmllr.sh  --cmd "$train_cmd" --nj 10 \
  data/train data/lang exp/tri4a exp/tri4a_ali
```

```shell
# Building a larger SAT system.

steps/train_sat.sh --cmd "$train_cmd" \
  3500 100000 data/train data/lang exp/tri4a_ali exp/tri5a || exit 1;

utils/mkgraph.sh data/lang_test exp/tri5a exp/tri5a/graph || exit 1;
steps/decode_fmllr.sh --cmd "$decode_cmd" --nj 10 --config conf/decode.config \
   exp/tri5a/graph data/dev exp/tri5a/decode_dev || exit 1;
steps/decode_fmllr.sh --cmd "$decode_cmd" --nj 10 --config conf/decode.config \
   exp/tri5a/graph data/test exp/tri5a/decode_test || exit 1;

steps/align_fmllr.sh --cmd "$train_cmd" --nj 10 \
  data/train data/lang exp/tri5a exp/tri5a_ali || exit 1;
```



### DNN-HMM模型训练

```shell
# nnet3
local/nnet3/run_tdnn.sh
```

```shell
# chain
local/chain/run_tdnn.sh
```

```shell
# getting results (see RESULTS file)
for x in exp/*/decode_test; do [ -d $x ] && grep WER $x/cer_* | utils/best_wer.sh; done 2>/dev/null
```

