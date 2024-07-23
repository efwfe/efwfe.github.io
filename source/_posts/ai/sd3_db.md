
---
title: Stable Diffusion 3 Dreambooth微调
date: 2024-07-03 10:24:20
tags:
  - AI
  - SD
  
---
  

## 训练概况：

  

我们正在进行图像风格迁移到Stable Diffusion 3 (SD3)的实验。由于目前sd3出来不久，对sd3的新的DIT架构的微调技术资料并不是很多，目前比较多的方面是diffusers和sd-scripts的实现，目前【2024-7-22】采用的是基于Dreambooth的实现方法。

  

在使用的过程中，先使用的是diffusers的dreambooth脚本，但是训练了几次，学习率在1e-4, 1e-5左右测试了几组，梯度下降的优化都是使用的AdamW8bit, 发现模型的效果都不是很好，和训练集的风格有点接近，但是差距还是挺明显的。

  

就切换到sd-script仓库的脚本下，使用的sd3分支，这个分支目前依然是开发状态中，目前是不可以微调text encoder部分，加上sd3的架构比较新颖，目前只有dreambooth的实现。

  
  

[sd-scripts的项目地址](https://github.com/kohya-ss/sd-scripts)

  

## 训练的服务器的配置：

使用的服务器型号：g5x12large aws, GPU占用四张A10G卡 基本上都占用完，一张卡22G左右；

训练效果比较好的参数：

1. 分辨率：1024

2. 梯度下降优化算法：adamW8bit

3. Epoch: 16

4. Steps: 4000

5. lr_scheduler 学习率调度: cosine_with_restarts

6. 学习率： 1e-4

  
  

## 训练命令

```shell

accelerate launch ./sd3_train.py \
--pretrained_model_name_or_path="/opt/models/sd3_medium.safetensors"\
--optimizer_type="AdamW8bit" \
--cache_text_encoder_outputs \
--cache_text_encoder_outputs_to_disk \
--vae_batch_size=1 \
--cache_latents --cache_latents_to_disk \
--caption_extension=".txt" \
--learning_rate=1e-4 \
--lr_scheduler="cosine_with_restarts" \
--output_name="last" \
--save_model_as=safetensors\
--save_precision="fp16" \
--train_batch_size=1 \
--resolution="1024,1024" \
--save_every_n_epochs=1 \
--lr_scheduler_num_cycles=1 \
--clip_g="/opt/models/text_encoders/clip_g.safetensors" \
--clip_l="/opt/models/text_encoders/clip_l.safetensors" \
--t5xxl="/opt/models/text_encoders/t5xxl_fp16.safetensors" \
--max_train_epochs=16 \
--mixed_precision=bf16 \
--debiased_estimation_loss \
--lr_warmup_steps=0 \
--keep_tokens=1 \
--output_dir="<OUTPUT DIR>" \
--train_data_dir="<INPUT DIR>"\
--full_bf16

```

  

## 验证代码

  

```python

import torch

from itertools import cycle

from diffusers import AutoPipelineForText2Image, StableDiffusion3Pipeline, AutoencoderKL

from transformers import T5EncoderModel, BitsAndBytesConfig, CLIPTokenizer, AutoTokenizer

from transformers import (

CLIPTextModelWithProjection,

CLIPTokenizer,

T5EncoderModel,

T5TokenizerFast,

)


_pipe = AutoPipelineForText2Image.from_pretrained(

"stabilityai/stable-diffusion-3-medium-diffusers",

torch_dtype=torch.float16

).to('cpu')


pipe = StableDiffusion3Pipeline.from_single_file(

model_path,

text_encoder=_pipe.text_encoder,

text_encoder_2=_pipe.text_encoder_2,

text_encoder_3=_pipe.text_encoder_3,

torch_dtype=torch.float16).to('cuda')

image = pipe(prompt).images[0]

  

```

  
  

## 微调lora的遇到的问题

  

1. keep token

> sd-scripts的参数中有一个--keep-token参数，这个参数表示保留前n个参数，后面的参数可以用乱序打乱来，但是目前在sd3中text都会在训练之前缓存到本地，也就是这个参数并不能配合乱序的text文本进行训练

> 解决：启用或者不启用影响不是很大，建议启用

  

1. 图片发白的问题

> 在测试的steps增加的情况下，有可能会出现发白的情况，特别是白色的图片基本上整体都是白的了，另外就是这种情况并不是都会出现，因为我还有两组参数的结果里面并没有很发白，应该是局部最优解导致的问题，所以可以从学习率调度和学习率上调整，一个epoch的steps也没有必要太多，针对一个epoch反向太多我的理解可能更容易导致过拟合吧，因为1个epoch意味着便利一次训练集，通常会在每一次epoch的时候shuffle数据集，如果太多的重复数据分布可能会更容易导致过拟合或者效果不佳的情况。

> 解决： 调整steps，学习率和学习率调度

  

1. 欠拟合的问题

> 刚开始实验的时候我们使用的是51张图，后续采用了102张图，整体上来说要比前者好很多，因为db本身就需要更多的数据来训练吧。在测试中学习率调整的太高了比较容易会导致这种情况，整个图片的输出是模糊的，就好像是latent的数据，stable diffusion的中间态的图片一样。

> 解决：降低学习率

  

1. 过拟合的问题

> epcoh太多或者steps太多的时候，这种情况的输出结果拓展性就很差，过分拟合原图的效果

> 解决： 调整学习率，适当调整steps和epcoh的批次数，根据每一次的输出结果保存的checkpint中选一组比较可靠的模型。


## 注意事项

1. 测试的cfg的值设置在4-5左右可能比较好，diffusers默认是7。
2. 数据集的构建是比较重要的一部分。