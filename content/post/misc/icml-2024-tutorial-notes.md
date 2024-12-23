---
date: "2024-11-20T21:43:22+08:00"
draft: false
title: "ICML 2024 tutorial: 语言模型物理学"
slug: "icml-2024-tutorial-notes"
description: "我应该怎么训练大模型?"
image:
categories:
    - 杂志
tags:
    - ICML
    - 大模型
    - 微调
license: GNU General Public License v3.0
---

这是关于 ICML 2024 tutorial: 语言模型物理学的阅读笔记, 从 tutorial 中摘取一些对我有用的结论, 用于训练模型的参考.

## 知识

### 提取知识 Knowledge Extraction

传统的预训练语料 + 指令微调 + 可选的偏好学习训练流程可以给大模型输入知识, 但并不能让大模型正确地将存储的知识提取出来. 在 tutorial 中, 以人物传记为例, 传统的训练流程训练出的大模型无法就针对人物传记相关知识的提问给出正确答案. 要提升大模型的知识提取能力, 有两种办法: 混合预训练(mixed pretrain)和知识增强(knowledge augmentation).

#### 混合预训练 Mixed Pretrain

混合预训练指的是在预训练阶段, 不止使用传统语料, 还将指令微调语料也加入到预训练中. 例如将按照指令模板格式化的问答对加入到预训练语料中. 这样可以提升大模型对指令的理解能力

> 根据 tutorial, 在他们的测试中, 问答准确率是 86.6%

#### 知识增强 Knowledge Augmentation

除了在预训练语料中加入指令微调语料, 还可以对原有的语料进行增强. 例如在人物传记中, 使得每个人的每条信息不止被描述一次, 而是用各种不同的方式--更换表述方法, 更换句子结构, 更换语言等--描述多次. 这样也可以提升大模型对知识的提取能力

> 根据 tutorial, 在他们的测试中, 问答准确率是 96%

### 使用知识 Knowledge Manipulation

使用知识要求大模型基于特定的知识完成某些任务, 例如知识分类. 一个简单的例子是, 询问某个人物的出生月份是奇数还是偶数. 这个任务需要大模型先从人物传记中提取出出生月份, 然后判断这个数字是奇数还是偶数. 这个任务需要大模型使用 CoT(Chain of Thought) 来完成

> 作者给出了一个非常强的陈述: 大模型在用任何方式使用知识前, 必须先显式地将知识写出来

同样地, 想要让大模型可以通过 CoT 来使用知识, 也必须在预训练中加入相关的 CoT 语料. 仅仅在指令微调阶段进行 CoT 训练是没用的

### 知识的规模定律 Knowledge Scaling Law

这里作者引入了*知识位数*的概念, 但我并不打算详细地记录. 仅需要了解, 知识位数是每个参数能存储的*知识量*. 一般地, 知识位数越高, 模型存储的知识越多, 上限为 2 bit. 当大模型被*充分训练*时, 可以达到这个上限

要充分训练一个大模型, 需要将知识充分地**暴露(exposure)**给它. 作者给出的结论是: 如果每个知识都**暴露**了 1000 次, 则可以达到 2 bit 的知识位数. 这里的**暴露**不同于 epoch 数, 对一条知识训练 1000 epochs 并不是 1000 次**暴露**. tutorial 中作者并没有对**暴露**给出明确的定义. 我的理解是, 向知识增强那样, 一条知识在不同的地方用不同的方式展示给模型, 才算做不同的**暴露**

这带来一个问题: 充分暴露需要大量的数据, 而网络数据良莠不齐. 高质量数据和低质量数据混合起来进行训练, 会极大地(20 倍的差距)损害大模型的知识存储能力

其解决方法也很简单: 给每条数据加上一个来源字段, 例如在所有来自 wikipedia 的文本前面加上 `wikipedia.org`, 大模型将会自己学会区分不同来源的数据, 知识存储能力也会得到极大的恢复

## 推理

推理能力(使用或不使用 CoT)也是在预训练阶段习得的. 而且模型的思考并不是从 CoT 开始, 而是在 CoT 之前, 模型就已经*胸有成竹*. 推理能力和模型的深度(层数)正相关, 无论是否采用 CoT

> 在 tutorial 中, 为了解决数学问题, 模型在 CoT 之前就已经画出了变量求解依赖图, 并决定了哪些变量是必须要求解的

这里模型已经发展出了被称为*二级推理*的能力, 即求解依赖图. 这中能力可以被用于其他任务微调, 例如询问某个变量依赖于什么之类的

在推理过程中, 大模型可能会犯错. probing 显示, 当大模型犯错后, 它实际上也很*后悔*, 但无法修正那个错误. 为了让大模型能修正错误, 需要在预训练阶段加入错误修正的语料. 例如对于求解简单数学问题, 可以加入诸如试图提前求解还无法解得的变量等错误并修正之的语料. 语料中犯错误的频率高低并不会影响模型犯错误的概率, 事实上, 犯错频率越高, 模型正确率越高

> 所以 reflection 的想法是挺好的

同样地, 微调或者波束搜索(beam search)无法让大模型学会纠错或者提高推理正确率

## 模型

tutorial 的 Part 1 还讲解了关于大模型学习*语言格式*的相关内容, 但和我个人进行训练的关系不大, 所以我就不做笔记了

## 总结

整个 tutorial 看下来, 在以上任务中, 大模型的预训练阶段的作用远比微调阶段重要得多, 微调能做到的事情比我想象中的要少. 开源社区中诸多的大模型微调其实都暗含了一个前提: 它们的原始模型是经过了*良好的预训练*的. 相应地, 想要让一个模型完成原本它根本不能做到的事情(例如破限), 那就要从预训练阶段入手

> 是的, 说的就是 Qwen 系列! 在预训练中就加入了超强道德对齐的模型, 我的前几个微调的效果都不显著. 真不知道加这么多对齐干什么(恼)

然而进行预训练对我的硬件提出了更高的要求. 一般来说, 预训练任务应该使用全量训练来完成, 但对于个人玩家来说, 全量训练一个基本可用的小模型(7B 量级)是几乎不可能的

根据这篇论文: [LoRA Learns Less and Forgets Less](https://arxiv.org/abs/2405.09673), 使用 LoRA 并且将 rank 开到非常高的程度, 例如 256, 也能取得相对可以接受的效果, 且不会导致过多的遗忘. 不过缺点是无法让模型显著地摆脱原本数据的影响
