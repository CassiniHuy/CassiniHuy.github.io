---
title: 每周分享：Wav2Vec2语音识别模型微调代码实现、加语言模型推理
date: 2023-09-09 16:07:03
categories: [随笔]
tags: [神经网络, 深度学习, Wav2Vec2, 语音识别, 语言模型]
---

GitHub仓库：https://github.com/CassiniHuy/wav2vec2_finetune

<!--more-->

# 要干什么

实现的目标在于快捷地进行模型微调、模型测试这两种操作，有下面的需求：

- 给定有标签的数据集，给定一个huggingface上的wav2vec2本地或远程仓库id，能够方便地进行微调
- 数据集可能是自己收集的、也可能是huggingface上的
- 测试时候，能使用语言模型提高性能

# 具体实现

小细节没啥好说的，按部就班实现就完了，下面是可能可以提一下的问题：

## 如何使用语言模型进行推理？

huggingface的wav2vec2模型是可以使用带有LM的processor的，但是我们会发现，这需要我们使用的仓库本身带有相关实现，才能使用。

但是，很多常见的wav2vec2仓库本身并不带有可以直接调用的语言模型，即没有Wav2Vec2ProcessorWithLM类的processor可以使用，那么如何才能使其接上语言模型增强其性能呢？

理论上，我们完全可以使用其他带有语言模型仓库的processor来解码model的输出，但是其中会涉及一个问题：id和vocab的对应关系，即processor的decoder/encoder需要和原本仓库的匹配。

例如，我们有model1和对应的processor1，processor1不支持语言模型；同时我们有model2和对应的processor2，processor2支持语言模型，那么只需要进行下面几步，可以得到一个processor来给model1解码：

- 首先需要保证processor1的所有vocab，在processor2中都有与其对应的vocab，否则失败
- 初始化一个decoder为processor2的decoder，然后遍历可能的id，将其对应的vocab改为processor1的vocab，注意大小写依然与processor2的相同
- 对应修改encoder
- feature_extractor使用processor1的，tokenizer和decoder使用processor2的，得到新的processor可以用以解码model1

## 微调数据集准备

代码：https://github.com/CassiniHuy/wav2vec2_finetune/blob/main/utils/datasets.py

如果是huggingface上的数据集，直接载入即可，但是如果是自己的数据集，则需要先转为huggingface的datasets库里的dataset类型。

这个可以通过Dataset类的from_dict来完成，将数据以字典形式准备好，并且准备好类型信息，可转化为Dataset类。

如何划分数据集为训练集和测试集呢？例如，Dataset自己有一个select方法，可以随机选择一些index的数据出来。

## DataCollatorCTCWithPadding类

代码：https://github.com/CassiniHuy/wav2vec2_finetune/blob/main/wav2vec2/train_utils.py

这个类训练时给每个batch做padding，传入trainer。

当时我参考的wav2vec2微调代码需要自己实现，但是貌似目前huggingface已经有官方实现。

## TorchServe

另外，还尝试使用了一下torchserve来部署wav2vec2模型，需要将所有用到的模型相关文件(通过save_pretrained得到)都通过torch-model-archiver打包，然后其余和网上常见的操作一样。

有个例外是如果使用语言模型，会有路径问题：由于打包后，部署时会释放到一个临时目录，所有文件都是默认当前路径，而lm需要在一个名为language_model的子目录，所以需要在initialize里将语言模型相关的文件移动到这个目录中。


完。

