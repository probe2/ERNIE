**key features**

1. pretrain and finetune ERNIE with [PaddlePaddle v1.7](https://github.com/PaddlePaddle/Paddle/tree/release/1.7)
1. supports eager execution with `paddle.fluid.dygraph`
1. supports `distributed training`

![](https://ernie-github.cdn.bcebos.com/gif/dygrpah_show.gif)

# Quick Tour

```python
import numpy as np
import paddle.fluid.dygraph as D
from ernie.tokenizing_ernie import ErnieTokenizer
from ernie.modeling_ernie import ErnieModel

D.guard().__enter__() # activate paddle `dygrpah` mode

model = ErnieModel.from_pretrained('ernie-1.0')    # Try to get pretrained model from server, make sure you have network connection
tokenizer = ErnieTokenizer.from_pretrained('ernie-1.0')

ids, _ = tokenizer.encode('hello world')
ids = D.to_variable(np.expand_dims(ids, 0))  # insert extra `batch` dimension
pooled, encoded = model(ids)                 # eager execution
print(pooled.numpy())                        # convert  results to numpy

```

# Tutorials

Don't have GPU? try ERNIE in [AIStudio](https://aistudio.baidu.com/aistudio/index)!

1. [Text classification](https://aistudio.baidu.com/aistudio/projectdetail/427482)
2. [Cloze test](https://aistudio.baidu.com/aistudio/projectdetail/433491)
3. [Knowledge Distillation](https://aistudio.baidu.com/aistudio/projectdetail/439460)
4. [Ask Ernie](https://aistudio.baidu.com/aistudio/projectdetail/456443)
5. ...

# Setup

##### 1. install dependencies with:

```script
pip install -r requirement.txt
```

##### 2.  put `$PWD`(root directory of this repo) into `$PYTHONPATH` with:

```script
export PYTHONPATH=$PWD:$PYTHONPATH
```

##### 3. download pretrained models

| Model                                              | Description                                                  |
| :------------------------------------------------- | :----------------------------------------------------------- |
| [ERNIE 1.0 Base for Chinese](https://ernie-github.cdn.bcebos.com/model-ernie1.0.1.tar.gz)           | ernie 1.0 base: L12H768A12|
| [ERNIE tiny](https://ernie-github.cdn.bcebos.com/model-ernie_tiny.1.tar.gz)                         | erine tiny: L3H1024A16|
| [ERNIE 2.0 Base for English](https://ernie-github.cdn.bcebos.com/model-ernie2.0-en.1.tar.gz)        | ernie 2.0 base: L12H768A12  |
| [ERNIE 2.0 Large for English](https://ernie-github.cdn.bcebos.com/model-ernie2.0-large-en.1.tar.gz) | ernie 2.0 large: L24H1024A16 |

##### 4. download datasets
 
**English Datasets**

Download the [GLUE datasets](https://gluebenchmark.com/tasks) by running [this script](https://gist.github.com/W4ngatang/60c2bdb54d156a41194446737ce03e2e) 

the `--data_dir` option in the following section assumes a directory tree like this:

```shell
data/xnli
├── dev
│   └── 1
├── test
│   └── 1
└── train
    └── 1
```

see [demo](https://ernie-github.cdn.bcebos.com/data-mnli-m.tar.gz) data for MNLI task.

**Chinese Datasets**

- [XNLI](https://ernie-github.cdn.bcebos.com/data-xnli.tar.gz)
- [ChnSentiCorp](https://ernie-github.cdn.bcebos.com/data-chnsenticorp.tar.gz)
- [MSRA-NER](https://ernie-github.cdn.bcebos.com/data-msra_ner.tar.gz)
- [NLPCC2016-DBQA](https://ernie-github.cdn.bcebos.com/data-dbqa.tar.gz)


# Finetune 

- try eager execution with `dygraph model` :

```script
python3 ./ernie/finetune_classifier_dygraph.py \
    --from_pretrained ernie_1.0 \
    --data_dir ./data/xnli 
```

- Distributed finetune

`paddle.distributed.launch` is a process manager, we use it to launch python processes on each avalible GPU devices:

when in distributed training, `max_steps` is used as stopping criteria rather than `epoch` to prevent dead block.
also notice than we shard the train data according to device id to prevent over fitting.

demo: 

```script
python3 -m paddle.distributed.launch \
./ernie/finetune_classifier_dygraph_distributed.py \
    --data_dir data/mnli \
    --max_steps 10000 \
    --from_pretrained ernie2.0-en
```


other demo python script:

1. [Name Entity Recognition(NER)](./ernie/finetune_ner_dygraph.py)
1. [Machine Reading Comprehension](./ernie/finetune_mrc_dygraph.py)


**recomended hyper parameters:**

|tasks|batch size|learning rate|
|--|--|--|
| CoLA         | 32 / 64 (base)  | 3e-5                     |
| SST-2        | 64 / 256 (base) | 2e-5                     |
| STS-B        | 128             | 5e-5                     |
| QQP          | 256             | 3e-5(base)/5e-5(large)   |
| MNLI         | 256 / 512 (base)| 3e-5                     |
| QNLI         | 256             | 2e-5                     |
| RTE          | 16 / 4 (base)   | 2e-5(base)/3e-5(large)   |
| MRPC         | 16 / 32 (base)  | 3e-5                     |
| WNLI         | 8               | 2e-5                     |
| XNLI         | 512             | 1e-4(base)/4e-5(large)   |
| CMRC2018     | 64              | 3e-5                     |
| DRCD         | 64              | 5e-5(base)/3e-5(large)   |
| MSRA-NER(SIGHAN2006)  | 16     | 5e-5(base)/1e-5(large)   |
| ChnSentiCorp | 24              | 5e-5(base)/1e-5(large)   |
| LCQMC        | 32              | 2e-5(base)/5e-6(large)   |
| NLPCC2016-DBQA| 64             | 2e-5(base)/1e-5(large)   |

# Distributed pretrain

see [here](./ernie/pretrain/README.md)

# Citation

please cite our [Arxiv paper](https://arxiv.org/abs/1907.12412):

```
@article{SunERNIE,
  title={ERNIE 2.0: A Continual Pre-training Framework for Language Understanding},
  author={Sun, Yu and Wang, Shuohuan and Li, Yukun and Feng, Shikun and Tian, Hao and Wu, Hua and Wang, Haifeng},
}
```

