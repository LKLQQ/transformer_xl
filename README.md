# Contents

- [Contents](#Contents)
    - [Transformer_XL Description](#transformer-xl-description)
    - [Model Architecture](#model-architecture)
    - [Dataset](#dataset)
    - [Environment Requirements](#environment-requirements)
    - [Quick Start](#quick-start)
    - [Script Description](#script-description)
        - [Script and Sample Code](#script-and-sample-code)
        - [Script Parameters](#script-parameters)
            - [Training Script Parameters](#training-script-parameters)
            - [Running Options](#running-options)
            - [Network Parameters](#network-parameters)
    - [Dataset Preparation](#dataset-preparation)
    - [Training Process](#training-process)
    - [Evaluation Process](#evaluation-process)
    - [Model Description](#model-description)
        - [Performance](#performance)
            - [Training Performance](#training-performance)
            - [Evaluation Performance](#evaluation-performance)
    - [Description of Random Situation](#description-of-random-situation)
    - [ModelZoo Homepage](#modelzoo-homepage)

## [Transformer_XL Description](#contents)

Transformer-XL is an improvement to Transformer, mainly to solve the problem of long sequences. At the same time, it
combines the advantages of RNN sequence modeling and Transformer's self-attention mechanism, introduces a recurrent
mechanism and relative position encoding, uses Transformer's attention module on each segment of the input data, and
uses a recurrent mechanism to learn the relationship between consecutive segments. dependencies. And successfully
achieved SoTA effect on language modeling datasets such as enwik8 and text8.

[Paper](https://arxiv.org/abs/1901.02860):  Dai Z, Yang Z, Yang Y, et al. Transformer-xl: Attentive language models
beyond a fixed-length context[J]. arXiv preprint arXiv:1901.02860, 2019.

## [Model Architecture](#contents)

The backbone structure of Transformer-XL is Transformer, which adds Recurrence Mechanism and Relative Positional
Encoding on the original basis.

## [Dataset](#contents)

The following two datasets contain the training dataset and the evaluation dataset. Recommended for dataset `bash getdata.sh` is automatically downloaded and preprocessed.

[enwik8](http://mattmahoney.net/dc/enwik8.zip)

Enwik8 data set is based on Wikipedia and is usually used to measure the ability of the model to compress data. Contains 100MB of unprocessed Wikipedia text.

If you download the enwik8 dataset directly through the link, download and execute [prep_enwik8.py](https://raw.githubusercontent.com/salesforce/awd-lstm-lm/master/data/enwik8/prep_enwik8.py) Preprocess the downloaded data set.

Dataset size:

- Training set: 88,982,818 characters in total
- Validation set: 4,945,742 characters in total
- Test set: 36,191 characters in total

Dataset format: TXT text

Dataset directory structure:

```text
└─data
  ├─enwik8
    ├─train.txt       # Training set
    ├─train.txt.raw   # Training set(unprocessed)
    ├─valid.txt       # Validation set
    ├─valid.txt.raw   # Validation set(unprocessed)
    ├─test.txt        # Test set
    └─test.txt.raw    # Test set(unprocessed)
```

- [text8](http://mattmahoney.net/dc/text8.zip)

Text8 also contains 100MB of Wikipedia text. The difference is to move other characters except 26 letters and spaces based on the enwik8 dataset.

If you download the text8 dataset directly through the link, execute prep_ text8.py to preprocess the downloaded data set.

Dataset size:

- Training set: 89,999,999 characters in total
- Validation set: 4,999,999 characters in total
- Test set: 5,000,000 characters in total

Dataset format: TXT text

Dataset directory structure:

```text
└─data
  ├─text8
    ├─train.txt       # Training set
    ├─train.txt.raw   # Training set(unprocessed)
    ├─valid.txt       # Validation set
    ├─valid.txt.raw   # Validation set(unprocessed)
    ├─test.txt        # Test set
    └─test.txt.raw    # Test set(unprocessed)
```

## [Environment Requirements](#contents)

- Hardware（Ascend/GPU）
    - Prepare hardware environment with Ascend or GPU processor.
- Framework
    - [MindSpore](https://gitee.com/mindspore/mindspore)
- For more information, please check the resources below：
    - [MindSpore Tutorials](https://www.mindspore.cn/tutorials/en/master/index.html)
    - [MindSpore Python API](https://www.mindspore.cn/docs/api/en/master/index.html)

## [Quick Start](#contents)

- Running on GPU

After dataset preparation, you can start training and evaluation as follows:

```bash
# Fine-tuning of parameters: hyperparameters in enwik8_base.yaml
# Where [DATA_NAME] belongs to the default parameter [enwik8, text8]
# The [TRAIN_URL] parameter can be set to a character name like "experiments", which will automatically create the corresponding model training file under "/script/train/experiments-enwik8" according to this name, or it can be set to a path, such as "/home/mindspore/transformer-xl/enwik8_8p". In this way, the training model will be saved separately in this directory.

# run training example
bash run_standalone_train_gpu.sh [DEVICE_ID] [DATA_DIR] [DATA_NAME] [TRAIN_URL] [CONFIG_PATH]
# for example: bash run_standalone_train_gpu.sh 0 /home/mindspore/transformer-xl/data/enwik8/ enwik8 experiments /home/mindspore/transformer-xl/yaml/enwik8_base.yaml

# run distributed training example
bash run_distribute_train_gpu.sh [DEVICE_NUM] [VISIABLE_DEVICES(0,1,2,3,4,5,6,7)] [DATA_DIR] [DATA_NAME] [TRAIN_URL] [CONFIG_PATH]
# for example: bash run_distribute_train_gpu.sh 4 0,1,2,3 /home/mindspore/transformer-xl/data/enwik8/ enwik8 experiments /home/mindspore/transformer-xl/yaml/enwik8_base.yaml

# run evaluation example
bash run_eval_gpu.sh [DATA_URL] [DATA_NAME] [CKPT_PATH] [CONFIG_PATH] [DEVICE_ID(optional)]
# for example: bash run_eval_gpu.sh  /home/mindspore/transformer-xl/data/enwik8/ enwik8 /home/mindspore/transformer-xl/script/experiments-enwik8/20220416-140816/model7.ckpt /home/mindspore/transformer-xl/yaml/enwik8_base.yaml 0
```

## [Script Description](#contents)

### [Script and Sample Code](#contents)

```text
.
└─Transformer-XL
  ├─README.md             // descriptions about Transformer-XL
  ├─README_CN.md          // descriptions about Transformer-XL
  ├─scripts
    ├─run_distribute_train_gpu.sh   // shell script for distributed training on GPU
    ├─run_standalone_train_gpu.sh   // shell script for training on GPU
    └─run_eval_gpu.sh               // shell script for testing on GPU
  ├─src
    ├─callback
      ├─eval.py           // callback function(eval)
      ├─flag.py           // callback function(flag)
      └─log.py            // callback function(log)
    ├─loss_fn
      └─ProjectedAdaptiveLogSoftmaxLoss.py    // loss
    ├─metric
      └─calc.py               // get bpc and ppl
    ├─model
      ├─attn.py               // Attention code
      ├─dataset.py            // get dataset
      ├─embedding.py          // PositionalEmbedding and AdaptiveEmbedding
      ├─layer.py              // layer code
      ├─mem_transformer.py    // Transformer-XL model
      ├─positionwiseFF.py     // positionwiseFF
      └─vocabulary.py         // construct vocabulary
    ├─model_utils
      ├─config.py             // parameter configuration
      ├─device_adapter.py     // device adapter
      ├─local_adapter.py      // local adapter
      └─moxing_adapter.py     // moxing adapter
    ├─utils
      ├─additional_algorithms.py  // General method
      ├─dataset_util.py           // Interface to get dataset
      └─nnUtils.py                // Basic method
  ├─yaml
      ├─enwik8_base.yaml          // parameter configuration on gpu
      ├─enwik8_large.yaml         // parameter configuration on gpu
      └─text8_large.yaml          // parameter configuration on gpu
  ├─getdata.sh                    // shell script for preprocessing dataset
  ├─eval.py                       // evaluation script
  └─train.py                      // training script
```

### [Script Parameters](#contents)

#### Training Script Parameters

```text
usage:
train.py
If you need to set the parameters, you can modify the . /enwik8_base.yaml file to implement the parameters.
If you need to change the parameter configuration file, you can change the --config_path parameter of line130 in /src/model_utils/config.py.

```

#### Network Parameters

```text
Parameters for dataset and network (Training/Evaluation):
    n_layer       number of total layers: N, default is 12
    d_model       dimension of model, default is 512
    n_head        number of heads, default is 8
    d_head        head dimension, default is 64
    d_inner       inner dimension in FF, default is 2048
    dropout       global dropout rate: Q, default is 0.1
    dropatt       attention probability dropout rate: Q, default is 0.0
    max_step      maximum of step: N, default is 400000
    tgt_len       number of tokens to predict, default is 512
    mem_len       length of the retained previous heads, default is 512
    eval_tgt_len  number of tokens to predict for evaluation, default is 128
    batch_size    batch size of input dataset: N, default is 22

Parameters for learning rate:
    lr            value of learning rate: Q, default is 0.00025
    warmup_step   steps of the learning rate warm up: N, default is 0
```

### [Dataset Preparation](#contents)

- Download the dataset and configure DATA_PATH

### [Training Process](#contents)

- Set options in `enwik8_base.yaml`, including loss_scale, learning rate and network hyperparameters.

- Run `run_standalone_train_gpu.sh` for training of Transformer-XL model.

    ```
    # run training example
    bash run_standalone_train_gpu.sh [DEVICE_ID] [DATA_DIR] [DATA_NAME] [TRAIN_URL] [CONFIG_PATH]
    # for example: bash run_standalone_train_gpu.sh 0 /home/mindspore/transformer-xl/data/enwik8/ enwik8 experiments /home/mindspore/transformer-xl/yaml/enwik8_base.yaml
    ```

- Run `run_distribute_train_gpu.sh` for distributed training of Transformer-XL model.

    ```
    # run distributed training example
    bash run_distribute_train_gpu.sh [DEVICE_NUM] [VISIABLE_DEVICES(0,1,2,3,4,5,6,7)] [DATA_DIR] [DATA_NAME] [TRAIN_URL] [CONFIG_PATH]
    # for example: bash run_distribute_train_gpu.sh 4 0,1,2,3 /home/mindspore/transformer-xl/data/enwik8/ enwik8 experiments /home/mindspore/transformer-xl/yaml/enwik8_base.yaml
    ```

### [Evaluation Process](#contents)

- Set options in `enwik8_base.yaml`. Make sure the 'datadir' are set to your own path.

- Run `run_eval_gpu.sh` for evaluation of Transformer model.

    ```
    # run evaluation example
    bash run_eval_gpu.sh [DATA_URL] [DATA_NAME] [CKPT_PATH] [CONFIG_PATH] [DEVICE_ID(optional)]
    # for example: bash run_eval_gpu.sh  /home/mindspore/transformer-xl/data/enwik8/ enwik8 /home/mindspore/transformer-xl/script/experiments-enwik8/20220416-140816/model7.ckpt /home/mindspore/transformer-xl/yaml/enwik8_base.yaml 0
    ```

## [Model Description](#contents)

### [Performance](#contents)

#### Training Performance

| Parameters                 | GPU                                    |
| -------------------------- | -------------------------------------- |
| Resource                   | MindSpore                              |
| uploaded Date              | 22/04/2022 (month/day/year)            |
| MindSpore Version          | 1.6.1                                  |
| Dataset                    | enwik8                                 |
| Training Parameters        | batch_size=22                          |
| Optimizer                  | Adam                                   |
| Loss Function              | Softmax Cross Entropy                  |
| BPC Score                  | 1.07906                                |
| Speed                      | 421.24ms/step(1p,bsz=8)                      |
| Loss                       | 0.75                                   |
| Checkpoint for inference   | 1.45G(.ckpt文件)                        |
| Scripts                    | Transformer scripts                    |

#### Evaluation Performance

| Parameters          | GPU                         |
| ------------------- | --------------------------- |
| Resource            | MindSpore                   |
| Uploaded Date       | 22/04/2022 (month/day/year) |
| MindSpore Version   | 1.6.1                       |
| Dataset             | enwik8                      |
| batch_size          | 22                          |
| outputs             | loss,bpc                    |
| Loss                | 0.75                        |
| BPC Score           | 1.07906                     |

## [Description of Random Situation](#contents)

There are three random situations:

- Shuffle of the dataset.
- Initialization of some model weights.
- Dropout operations.

Some seeds have already been set in train.py to avoid the randomness of dataset shuffle and weight initialization. If
you want to disable dropout, please set the corresponding dropout_prob parameter to 0 in default_config.yaml.

## [ModelZoo Homepage](#contents)

Please check the official [homepage](https://gitee.com/mindspore/models).
