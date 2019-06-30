在 kaldi 训练过程中，DNN 的训练是依赖于 GMM-HMM 模型的，通过 GMM-HMM 模型得到 DNN 声学模型的输出结果(在 get_egs.sh 脚本中可以看到这一过程)。因此训练一个好的 GMM-HMM 模型是 kaldi 语音识别的关键。

## NN in Kaldi

nnet1 & nnet2：以层设计为基础，当你新增加一种神经网络层时需要自己定义它的结构，都有哪些变量，正向怎么算，反向误差怎么传播等。难以支持过于复杂的连接方式。

nnet3：以图结构为基础的，通过配置文件实现对网络连接方式的定义，数据就像流水一样在你东一的网络图中游走，并自己实现误差的反向传播。它的有点事你可以专注网络拓扑结构的设计，而不用为网络计算的细节而费心。但对初学者而言，会造成只是在设计网络长成什么样子，但不清楚其中的实现细节。建议初学者多推推公式。

## TDNN

### run_ivector_common.sh

- **数据增广**：对训练数据集进行三种速度变换：0.9，1.0，1.1，存放在data/{train,test,dev}_sp。

  其中utt2dur是utterance到该音频时长（秒）的映射 utils/data/get_utt2dur.sh 。

  reco2utt记录recording到该recording的时长的映射 utils/data/get_reco2dur.sh。

  utils/data/perturb_data_dir_speed.sh，使用sox工具

- 对变速后的训练数据提取MFCC加音高特征、CMVN特征，保存在mfcc_perturbed。

- 使用速度变换后的数据做对齐

- 创建高分辨率MFCC特征（40维倒谱系数），保存在mfcc_perturbed_hires。

  调用conf/mfcc_hires.conf 配置文件，说明：40维MFCC特征适用于神经网络运算，保留了所有倒谱系数，和filterbank很像，但更不相关，因此选择MFCC特征。

  根据不加pitch的mfcc特征计算cmvn，用于生成ivector

- 使用1/4数据训练对角UBM，进行PCA变换，使用512个高斯训练UBM
- 训练iVector提取器
- 提取训练集、测试集、验证集的iVector

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
if

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
  echo "$0: aligning with the perturbed low-resolution data"
  steps/align_fmllr.sh --nj 30 --cmd "$train_cmd" \

if [ $stage -le 3 ]; then
  # Create high-resolution MFCC features (with 40 cepstra instead of 13).
  # this shows how you can split across multiple file-systems.
  echo "$0: creating high-resolution MFCC features"
  mfccdir=mfcc_perturbed_hires$online_affix
  if [[ $(hostname -f) == *.clsp.jhu.edu ]] && [ ! -d $mfccdir/storage ]; then
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
    utils/fix_data_dir.sh data/${datadir}_hires$online_affix || exit 1;
    # create MFCC data dir without pitch to extract iVector
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

### run_tdnn_1a.sh:

### run_tdnn_2a.sh:

- 参数：

  ivector_dim：feat-to-dim scp:exp/nnet3/ivectors_train_sp/ivector_online.scp -   100

  feat_dim：feat-to-dim scp:data/train_sp_hires/feats.scp -   43

  num_targets：tree-info exp/tri5a_sp_ali/tree | grep num-pdfs | awk '{print $2}'   2952

- 创建神经网络配置文件

  

- 

  

```shell
#!/bin/bash

# This script is based on aishell/s5/local/nnet3/tuning/run_tdnn_1a.sh

# In this script, the neural network in trained based on hires mfcc and online pitch.
# The online pitch setup requires a online_pitch.conf in the conf dir for both training
# and testing.

set -e

stage=0
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

### steps/nnet3/train_dnn.py:

```python
#!/usr/bin/env python

# Copyright 2016    Vijayaditya Peddinti.
#           2016    Vimal Manohar
#           2017 Johns Hopkins University (author: Daniel Povey)
# Apache 2.0.

""" This script is based on steps/nnet3/tdnn/train.sh
"""

from __future__ import print_function
from __future__ import division
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
import libs.nnet3.train.frame_level_objf as train_lib
import libs.nnet3.report.log_parse as nnet3_log_parse


logger = logging.getLogger('libs')
logger.setLevel(logging.INFO)
handler = logging.StreamHandler()
handler.setLevel(logging.INFO)
formatter = logging.Formatter("%(asctime)s [%(pathname)s:%(lineno)s - "
                              "%(funcName)s - %(levelname)s ] %(message)s")
handler.setFormatter(formatter)
logger.addHandler(handler)
logger.info('Starting DNN trainer (train_dnn.py)')


def get_args():
    """ Get args from stdin.

    We add compulsory arguments as named arguments for readability

    The common options are defined in the object
    libs.nnet3.train.common.CommonParser.parser.
    See steps/libs/nnet3/train/common.py
    """
    parser = argparse.ArgumentParser(
        description="""Trains a feed forward DNN acoustic model using the
        cross-entropy objective.  DNNs include simple DNNs, TDNNs and CNNs.""",
        formatter_class=argparse.ArgumentDefaultsHelpFormatter,
        conflict_handler='resolve',
        parents=[common_train_lib.CommonParser(include_chunk_context=False).parser])

    # egs extraction options
    parser.add_argument("--egs.frames-per-eg", type=int, dest='frames_per_eg',
                        default=8,
                        help="Number of output labels per example")

    # trainer options
    parser.add_argument("--trainer.input-model", type=str,
                        dest='input_model', default=None,
                        action=common_lib.NullstrToNoneAction,
                        help="""If specified, this model is used as initial
                        raw model (0.raw in the script) instead of initializing
                        the model from xconfig. Configs dir is not expected to
                        exist and left/right context is computed from this
                        model.""")
    parser.add_argument("--trainer.prior-subset-size", type=int,
                        dest='prior_subset_size', default=20000,
                        help="Number of samples for computing priors")
    parser.add_argument("--trainer.num-jobs-compute-prior", type=int,
                        dest='num_jobs_compute_prior', default=10,
                        help="The prior computation jobs are single "
                        "threaded and run on the CPU")

    # Parameters for the optimization
    parser.add_argument("--trainer.optimization.minibatch-size",
                        type=str, dest='minibatch_size', default='512',
                        help="""Size of the minibatch used in SGD training
                        (argument to nnet3-merge-egs); may be a more general
                        rule as accepted by the --minibatch-size option of
                        nnet3-merge-egs; run that program without args to see
                        the format.""")

    # General options
    parser.add_argument("--feat-dir", type=str, required=False,
                        help="Directory with features used for training "
                        "the neural network.")
    parser.add_argument("--lang", type=str, required=False,
                        help="Language directory")
    parser.add_argument("--ali-dir", type=str, required=True,
                        help="Directory with alignments used for training "
                        "the neural network.")
    parser.add_argument("--dir", type=str, required=True,
                        help="Directory to store the models and "
                        "all other files.")

    print(' '.join(sys.argv), file=sys.stderr)
    print(sys.argv, file=sys.stderr)

    args = parser.parse_args()

    [args, run_opts] = process_args(args)

    return [args, run_opts]


def process_args(args):
    """ Process the options got from get_args()
    """

    if args.frames_per_eg < 1:
        raise Exception("--egs.frames-per-eg should have a minimum value of 1")

    if not common_train_lib.validate_minibatch_size_str(args.minibatch_size):
        raise Exception("--trainer.rnn.num-chunk-per-minibatch has an invalid value")

    if (not os.path.exists(args.dir)):
        raise Exception("This script expects --dir={0} to exist.")
    if (not os.path.exists(args.dir+"/configs") and
        (args.input_model is None or not os.path.exists(args.input_model))):
        raise Exception("Either --trainer.input-model option should be supplied, "
                        "and exist; or the {0}/configs directory should exist."
                        "{0}/configs is the output of make_configs.py"
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
        run_opts.combine_gpu_opt = "--use-gpu={}".format(args.use_gpu)
        run_opts.combine_queue_opt = "--gpu 1"
        run_opts.prior_gpu_opt = "--use-gpu={}".format(args.use_gpu)
        run_opts.prior_queue_opt = "--gpu 1"

    else:
        logger.warning("Without using a GPU this will be very slow. "
                       "nnet3 does not yet support multiple threads.")

        run_opts.train_queue_opt = ""
        run_opts.parallel_train_opts = "--use-gpu=no"
        run_opts.combine_gpu_opt = "--use-gpu=no"
        run_opts.combine_queue_opt = ""
        run_opts.prior_gpu_opt = "--use-gpu=no"
        run_opts.prior_queue_opt = ""

    run_opts.command = args.command
    run_opts.egs_command = (args.egs_command
                            if args.egs_command is not None else
                            args.command)
    run_opts.num_jobs_compute_prior = args.num_jobs_compute_prior

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

    # Copy phones.txt from ali-dir to dir. Later, steps/nnet3/decode.sh will
    # use it to check compatibility between training and decoding phone-sets.
    shutil.copy('{0}/phones.txt'.format(args.ali_dir), args.dir)

    # Set some variables.
    # num_leaves = common_lib.get_number_of_leaves_from_tree(args.ali_dir)
    num_jobs = common_lib.get_number_of_jobs(args.ali_dir)
    feat_dim = common_lib.get_feat_dim(args.feat_dir)
    ivector_dim = common_lib.get_ivector_dim(args.online_ivector_dir)
    ivector_id = common_lib.get_ivector_extractor_id(args.online_ivector_dir)

    # split the training data into parts for individual jobs
    # we will use the same number of jobs as that used for alignment
    common_lib.execute_command("utils/split_data.sh {0} {1}".format(
        args.feat_dir, num_jobs))
    shutil.copy('{0}/tree'.format(args.ali_dir), args.dir)

    with open('{0}/num_jobs'.format(args.dir), 'w') as f:
        f.write('{}'.format(num_jobs))

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

    left_context = model_left_context
    right_context = model_right_context

    # Initialize as "raw" nnet, prior to training the LDA-like preconditioning
    # matrix.  This first config just does any initial splicing that we do;
    # we do this as it's a convenient way to get the stats for the 'lda-like'
    # transform.

    if (args.stage <= -5) and os.path.exists(args.dir+"/configs/init.config") and \
       (args.input_model is None):
        logger.info("Initializing a basic network for estimating "
                    "preconditioning matrix")
        common_lib.execute_command(
            """{command} {dir}/log/nnet_init.log \
                    nnet3-init --srand=-2 {dir}/configs/init.config \
                    {dir}/init.raw""".format(command=run_opts.command,
                                             dir=args.dir))

    default_egs_dir = '{0}/egs'.format(args.dir)
    if (args.stage <= -4) and args.egs_dir is None:
        logger.info("Generating egs")

        if args.feat_dir is None:
            raise Exception("--feat-dir option is required if you don't supply --egs-dir")

        train_lib.acoustic_model.generate_egs(
            data=args.feat_dir, alidir=args.ali_dir, egs_dir=default_egs_dir,
            left_context=left_context, right_context=right_context,
            run_opts=run_opts,
            frames_per_eg_str=str(args.frames_per_eg),
            srand=args.srand,
            egs_opts=args.egs_opts,
            cmvn_opts=args.cmvn_opts,
            online_ivector_dir=args.online_ivector_dir,
            samples_per_iter=args.samples_per_iter,
            stage=args.egs_stage)

    if args.egs_dir is None:
        egs_dir = default_egs_dir
    else:
        egs_dir = args.egs_dir

    [egs_left_context, egs_right_context,
     frames_per_eg_str, num_archives] = (
         common_train_lib.verify_egs_dir(egs_dir, feat_dim,
                                         ivector_dim, ivector_id,
                                         left_context, right_context))
    assert str(args.frames_per_eg) == frames_per_eg_str

    if args.num_jobs_final > num_archives:
        raise Exception('num_jobs_final cannot exceed the number of archives '
                        'in the egs directory')

    # copy the properties of the egs to dir for
    # use during decoding
    common_train_lib.copy_egs_properties_to_exp_dir(egs_dir, args.dir)

    if args.stage <= -3 and os.path.exists(args.dir+"/configs/init.config") and (args.input_model is None):
        logger.info('Computing the preconditioning matrix for input features')

        train_lib.common.compute_preconditioning_matrix(
            args.dir, egs_dir, num_archives, run_opts,
            max_lda_jobs=args.max_lda_jobs,
            rand_prune=args.rand_prune)

    if args.stage <= -2 and (args.input_model is None):
        logger.info("Computing initial vector for FixedScaleComponent before"
                    " softmax, using priors^{prior_scale} and rescaling to"
                    " average 1".format(
                        prior_scale=args.presoftmax_prior_scale_power))

        common_train_lib.compute_presoftmax_prior_scale(
            args.dir, args.ali_dir, num_jobs, run_opts,
            presoftmax_prior_scale_power=args.presoftmax_prior_scale_power)

    if args.stage <= -1:
        logger.info("Preparing the initial acoustic model.")
        train_lib.acoustic_model.prepare_initial_acoustic_model(
            args.dir, args.ali_dir, run_opts,
            input_model=args.input_model)

    # set num_iters so that as close as possible, we process the data
    # $num_epochs times, i.e. $num_iters*$avg_num_jobs) ==
    # $num_epochs*$num_archives, where
    # avg_num_jobs=(num_jobs_initial+num_jobs_final)/2.
    num_archives_expanded = num_archives * args.frames_per_eg
    num_archives_to_process = int(args.num_epochs * num_archives_expanded)
    num_archives_processed = 0
    num_iters = int(num_archives_to_process * 2 / (args.num_jobs_initial + args.num_jobs_final))

    # If do_final_combination is True, compute the set of models_to_combine.
    # Otherwise, models_to_combine will be none.
    if args.do_final_combination:
        models_to_combine = common_train_lib.get_model_combine_iters(
            num_iters, args.num_epochs,
            num_archives_expanded, args.max_models_combine,
            args.num_jobs_final)
    else:
        models_to_combine = None

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

            train_lib.common.train_one_iteration(
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
                minibatch_size_str=args.minibatch_size,
                frames_per_eg=args.frames_per_eg,
                momentum=args.momentum,
                max_param_change=args.max_param_change,
                shrinkage_value=shrinkage_value,
                shuffle_buffer_size=args.shuffle_buffer_size,
                run_opts=run_opts)

            if args.cleanup:
                # do a clean up everythin but the last 2 models, under certain
                # conditions
                common_train_lib.remove_model(
                    args.dir, iter-2, num_iters, models_to_combine,
                    args.preserve_model_interval)

            if args.email is not None:
                reporting_iter_interval = num_iters * args.reporting_interval
                if iter % reporting_iter_interval == 0:
                    # lets do some reporting
                    [report, times, data] = (
                        nnet3_log_parse.generate_acc_logprob_report(args.dir))
                    message = report
                    subject = ("Update : Expt {dir} : "
                               "Iter {iter}".format(dir=args.dir, iter=iter))
                    common_lib.send_mail(message, subject, args.email)

        num_archives_processed = num_archives_processed + current_num_jobs

    if args.stage <= num_iters:
        if args.do_final_combination:
            logger.info("Doing final combination to produce final.mdl")
            train_lib.common.combine_models(
                dir=args.dir, num_iters=num_iters,
                models_to_combine=models_to_combine,
                egs_dir=egs_dir,
                minibatch_size_str=args.minibatch_size, run_opts=run_opts,
                max_objective_evaluations=args.max_objective_evaluations)

    if args.stage <= num_iters + 1:
        logger.info("Getting average posterior for purposes of "
                    "adjusting the priors.")

        # If args.do_final_combination is true, we will use the combined model.
        # Otherwise, we will use the last_numbered model.
        real_iter = 'combined' if args.do_final_combination else num_iters
        avg_post_vec_file = train_lib.common.compute_average_posterior(
            dir=args.dir, iter=real_iter,
            egs_dir=egs_dir, num_archives=num_archives,
            prior_subset_size=args.prior_subset_size, run_opts=run_opts)

        logger.info("Re-adjusting priors based on computed posteriors")
        combined_or_last_numbered_model = "{dir}/{iter}.mdl".format(dir=args.dir,
                iter=real_iter)
        final_model = "{dir}/final.mdl".format(dir=args.dir)
        train_lib.common.adjust_am_priors(args.dir, combined_or_last_numbered_model,
                avg_post_vec_file, final_model, run_opts)


    if args.cleanup:
        logger.info("Cleaning up the experiment directory "
                    "{0}".format(args.dir))
        remove_egs = args.remove_egs
        if args.egs_dir is not None:
            # this egs_dir was not created by this experiment so we will not
            # delete it
            remove_egs = False

        common_train_lib.clean_nnet_dir(
            nnet_dir=args.dir, num_iters=num_iters, egs_dir=egs_dir,
            preserve_model_interval=args.preserve_model_interval,
            remove_egs=remove_egs)

    # do some reporting
    [report, times, data] = nnet3_log_parse.generate_acc_logprob_report(args.dir)
    if args.email is not None:
        common_lib.send_mail(report, "Update : Expt {0} : "
                                     "complete".format(args.dir), args.email)

    with open("{dir}/accuracy.report".format(dir=args.dir), "w") as f:
        f.write(report)

    common_lib.execute_command("steps/info/nnet3_dir_info.pl "
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



## chain

### local/chain/run_tdnn.sh

```shell
#!/bin/bash

# This script is based on run_tdnn_7h.sh in swbd chain recipe.

set -e

# configs for 'chain'
affix=
stage=8
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

# 记录

## kaldi中各种代码缩写的意思：

mic=ihm：基于独立耳机麦克风的各种语聊和模型

mic=sdm：基于单程麦克风的各种语聊和模型

mic=mdm：基于多程麦克风的各种语聊和模型

data_sp：数据集经过了速度变换（utils/data/perturb_data_dir_speed_3way.sh）

data_hires：数据集经过了音量变换（utils/data/perturb_data_dir_volume.sh）

***.vad：数据集经过了vad算法处理，把闲时噪音（阶段）去掉，只保留说话时候的音段

raw：表示神经网络使用的为13维的MFCC原始特征（没有经过一阶差分，二阶差分）

## 获取特征的维度：feat-to-dim 

feat-to-dim scp:feats.scp -

## 查看lattice统计信息

decode_test/scoring_kaldi/wer_details

**DNN训练阶段与GMM训练阶段的测试集时长不同**

在mono到tri5a阶段的decode_test中生成的analyze_lattice_depth_stats.log中，显示计算得到的数据时长为40小时，在nnet3训练解码时统计得到的时长也为40hr，而chain训练得到的时长为13.4小时。

## 查看kaldi各种文件的命令

**ark特征文件**
`copy-feats` 可以用来改变特征数据的格式，因此可以转换ark格式文件为txt格式：
用法:` copy-feats [options] <feats-rxfilename> <feats-wxfilename>`

**FST文件**
`fstprint`可以打印fst为文本格式：
`fstprint --isymbols=phones.txt --osymbols=words.txt L.fst L.txt`

`fstdraw`查看pdf格式的图：
fstdraw [--isymbols=phones.txt --osymbols=words.txt] L.fst | dot –Tps | ps2pdf – L.pdf

**mdl模型文件**

gmm模型：`gmm-copy [options] <model-in> <model-out>`
如：gmm-copy --binary=false 1.mdl 1_txt.mdl

dnn模型：用`nnet-copy`:
如：nnet-copy --binary=false 0.mdl final.txt

**决策树文件**

转化为文本格式指令为：
`copy-tree [--binary=false] <tree-in> <tree-out>`
如:copy-tree [--binary=false] tree tree.txt>

转化为图形格式指令为：
`draw-tree [options] <phone-symbols> <tree>`
如:draw-tree phones.txt tree | dot -Gsize=8,10.5 -Tps | ps2pdf - tree.pdf

**ali.gz对齐文件**
对齐文件可以通过`copy-int-vecto`r查看：
`copy-int-vector [options] (vector-in-rspecifier) (vector-out-wspecifier)`
实例：copy-int-vector "ark:gunzip -c ali.1.gz|" ark,t:ali.txt

也可以先解压，然后用`show-alignments`查看 :
`show-alignments [options] <phone-syms> <model> <alignments-rspecifier>`
实例：~/kaldi/src/bin/show-alignments phones.txt final.mdl ark:ali.1 > ali.1.txt

类似的有：`ali-to-phones`, `copy-int-vector`



