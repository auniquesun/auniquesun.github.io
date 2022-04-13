---
layout: post
title: PyTorch save and load DDP model
tags: [PyTorch, DistributedDataParallel, DDP, Model save & load]
gh-repo: MohamedAfham/CrossPoint
gh-badge: [star, fork, follow]
mathjax: true
comments: true
---

深度学习中，模型权重保存与加载是很普遍的需求，
最简单的情况是用**单个GPU**训练模型，完成后保存权重，重新加载时一般不会出问题。
以PyTorch为例，如下操作即可完成

```python
import torch

import Model
```

```python
# Model should be defined and trained earlier
# save model weights
model = Model(args)
torch.save(model.state_dict(), 'best_model.pth')
```

```python
# load previous `model` weights
model = Model(args)
pretrained_model = torch.load('best_model.pth')
model.load_state_dict(pretrained_model)
```

然而很多时候为了缩短实验周期，往往会用**多个GPU**加速训练过程，此时模型权重保存与加载会有些$\color{red}{注意事项}，
否则很可能得不到正确的预测/调优结果。以PyTorch为例，多卡训练模型后，保存步骤如下

方式一：
```python
import torch
from torch.utils.data import DistributedDataParallel as DDP

import Model
```

```python
# Model should be defined earlier
model = Model(args)
# wrap with DDP for training with multiple GPUs,
# here `rank` is a parameter specifying which GPU to used, and 
# it is passed into by PyTorch multiprocessing function
model_ddp = DDP(model, device_ids=[rank])

# after training, save `model_ddp` weights
torch.save(model_ddp.state_dict(), 'best_model.pth')
```

此时，如果想要加载多卡训练得到的 `best_model.pth`，不同情况加载方法不同，否则某些权重**不能正常加载**
(此时也无异常提示，而当事人以为一切正常，最终模型效果不好而又找不到问题)或报错

情况一：多卡加载（比如进行调优/下游任务）。因为**方式一**中模型是在DDP下模式保存的，所以加载时也要包裹在DDP下

```python
import torch
from torch.utils.data import DistributedDataParallel as DDP

import Model
```

```python
# Model should be defined earlier
model = Model(args)
# wrap with DDP for training with multiple GPUs,
# here `rank` is a parameter specifying which GPU to used, and 
# it is passed into by PyTorch multiprocessing function
model_ddp = DDP(model, device_ids=[rank])

# load `model_ddp` weights trained on multiple GPUs
pretrained_model = torch.load('best_model.pth')
model_ddp.load_state_dict(pretrained_model)
```

情况二：单卡加载（比如进行测试/推理）。因为**方式一**中模型是在DDP模式下模式保存的，而**单卡测试无法使用DDP模式**，此时就要对原来保存的
`best_model.pth` 的key作些更改，以适应单卡需求

```python
import torch

import Model
```

```python
# Model should be defined earlier
model = Model(args)

# load previous `model_ddp` weights
pretrained_model = torch.load('best_model.pth')
# Actually, best_model.pth is a torch.nn.Module, and 
#   each Module is a dict consisting of `key: value` pairs.
# Here we need to change `key` in pretrained_model
#   e.g. features.module.0.weight -> features.0.weight
#        key in DDP mode          -> key in single GPU mode
#        `module` in features.module.0.weight is added by DDP utility
pretrained_model = {key.replace("module.", ""): value for key, value in pretrained_model.items()}

# load model weights on single GPU
model.load_state_dict(pretrained_model)
```

可以看到**单卡加载多卡训练保存的模型**，需要对DDP模型的key作出修改，以匹配单卡加载出来模型的key；若不进行此修改
且 `model.load_state_dict(pretrained_model, strict=False)` 时，代码运行并$\color{red}{不会提示}$未匹配的key及对应的权重未被加载，
在推理/调优时效果会大打折扣而找不到问题所在。

当然，多卡训练完模型，保存时还有另一种方式，如下

方式二：
```python
import torch
from torch.utils.data import DistributedDataParallel as DDP

import Model
```

```python
# Model should be defined earlier
model = Model(args)
# wrap with DDP for training with multiple GPUs,
# here `rank` is a parameter specifying which GPU to used, and 
# it is passed into by PyTorch multiprocessing function
model_ddp = DDP(model, device_ids=[rank])

# after training, save model weights
# here `model_ddp.module.state_dict()` is equivalent to saving model 
# trained on single GPU since the synchronization mechanism in PyTorch DDP
# makes sure all models trained on different GPUs are same after every epoch
torch.save(model_ddp.module.state_dict(), 'best_model.pth')
```

因为保存机制作出了改变，加载机制也应作出改变，分不同情况

情况一：多卡加载（比如进行调优/下游任务）。因为**方式二**中模型相当于在单卡模式下保存，而多卡训练要用到DDP，所以就是
把模型加载出来再用DDP封装

```python
import torch
from torch.utils.data import DistributedDataParallel as DDP

import Model
```

```python
# Model should be defined earlier
model = Model(args)

# load previous `model_ddp` weights
pretrained_model = torch.load('best_model.pth')
model.load_state_dict(pretrained_model)

# wrap with DDP for training with multiple GPUs,
# here `rank` is a parameter specifying which GPU to used, and 
# it is passed into by PyTorch multiprocessing function
model_ddp = DDP(model, device_ids=[rank])
```

情况二：单卡加载（比如进行测试/推理）。因为**方式二**中模型相当于在单卡模式下保存，单卡直接加载即可

```python
import torch

import Model
```

```python
# Model should be defined earlier
model = Model(args)

# load `model` weights
pretrained_model = torch.load('best_model.pth')
model.load_state_dict(pretrained_model)
```

## 总结
很多人有这样的经历，复现模型时经常达不到论文报告的效果，有时差了好几个点又找不到问题，靠调参不可能补上这么大gap，
更离奇这是论文作者公开的代码。
这样的问题我也遇到过，比如复现 [CrossPoint](https://github.com/MohamedAfham/CrossPoint/)，加载预训练模型key不匹配，
导致最终效果里论文结果相差甚远，
主要原因就是上面讲的 `保存-加载` 不匹配（我把代码改成了分布式训练，原代码是单卡训练，但忽略了加载模型也要随之改变）
经过一番研究终于搞明白了这个问题，模型效果显著提升，某些数据集上的指标达到了与论文可比的结果。
总而言之，`单卡保存-单卡加载`，`多卡保存-多卡加载`一般不会有意外，而 `单卡保存-多卡加载`，`多卡保存-单卡加载` 就要
灵活应对了。