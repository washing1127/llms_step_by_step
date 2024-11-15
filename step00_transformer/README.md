<!--
 * @Author: washing1127
 * @Date: 2024-11-09 23:46:20
 * @LastEditors: washing1127
 * @LastEditTime: 2024-11-13 11:19:38
 * @FilePath: /llms_step_by_step/step00_transformer/README.md
 * @Description: 
-->
## 介绍
本项目为 LLMs_Step_by_Step 计划的起始第一步，计划分别通过 PyTorch 中自带的 `nn.Transformer` 模型和自己动手写的模型，实现一个翻译项目。因为只是练手项目，只求跑通代码，看到效果。不求特别严格的优化，也暂不追求泛化效果。

为方便新手入门，该项目希望尽量少的使用除 PyTorch 外的其他第三方库。

入口文件为 `main.py`，可通过修改 `which_model` 参数来选择用官方的模型(`official`)，还是自己动手实现的模型(`myself`)。两种情况除模型实现代码外其他如数据、优化器、损失函数等完全一样。理想情况下，在安装过 PyTorch 的前提下，拿到代码后可直接通过 `python main.py` 开始训练。

## 踩坑记录 
### 2024-11-07
做为自己的第一个从零开始的大模型项目，这个项目陆陆续续写了一个多月了，还没有完全成功。中间踩了好多好多坑，到今天才算是有一个比较明确的进步吧。随后就想到后续的工作中可以记录一下自己思路调整，或者对代码改进的日志。

先大致回忆一下到目前为止的思路历程吧：

**step1**

最开始的时候直接上手写了各个模块拆开的 transformer 模型。

模型任务定做了文言文翻译成现代文的翻译器。数据是在 GitHub 上找到的别人开源的数据集，进行了非常简单的清洗就开始跑了。

结果是 loss 几乎不下降，训练过程中输出的结果也是乱七八糟，都是 pad 什么的。

**step2**

想到先找一个别人开源的模型跑起来，就找了[月来客栈](https://github.com/moon-hotel)的[TransformerTranslation](https://github.com/moon-hotel/TransformerTranslation)仓库尝试跑了起来。（他的任务是德语翻译成英语）

配了半天环境终于跑起来了，也可以正常预测，就对照着他的代码改了一些我原来代码中明显不对的地方。

再训练，发现 loss 可以下降了，而且训练过程中的输出结果非常好。但是无法预测，预测结果集中于某一两个 token 上。推测是没有对 decoder 添加 mask 导致的。这个可以计作【实验2】。

这都还比较正常，写的时候也有留意，就是想试一下看看不添加 decoder 的 mask 会怎么样。所以这个问题很容易定位。

加入 mask 后再训练，发现 loss 也可以下降，但无法下降到实验2时的那么低，训练过程中的输出结果吃力看也可以看出来是在做翻译。

**step3**

这个过程才是浪费了最多时间的过程。

一开始，尝试调整了词典大小，尝试根据句子长度对训练数据进行了筛选，尝试了更细致的数据清洗，尝试了训练数据喂给模型的顺序，等等各种方法，都没有什么明显的改善。

然后尝试了更换任务数据集，换成英文翻译中文，也还是不行。我能感觉到是数据的问题，但是一直认为我自己的数据集比上面 TransformerTranslation 的数据多的多，没有理由他行我不行的。

直到最后我尝试了使用他的数据集（他的数据集是通过 spacy 加载的，这个包我没用过，一开始配环境的时候有点费劲，所以没想着往我自己的代码里加），发现模型可以正常表现了。我才意识到是数据集的问题。

最后预测的时候，我从测试集里找了两句话，又自己随便搜了一句英文，用百度翻译成德语做了个 case。预测发现，他原来测试集里的文本能翻译的很好，我自己创建的 case 也是翻译不了一点。终于彻底明白了是数据的问题。

这是知道今天才有的进展，想来可能是对方的数据集数据分布更统一一些？根据这个思路，后续再尝试构建一下文言文翻译现代文的数据试试吧。

### 2024-11-08

尝试爬取了《史记》的原文及白话文翻译。因为考虑到这样，可以保证古文和白话文各自的风格及内容方向尽可能地保持一致。

爬完后看看数据样式，尝试一下清洗规则。

### 2024-11-09

《史记》的文白对照，下载好后非常认真的整理了数据。整理的时候越来越觉得不太有希望，文言文对白话文不是像两种不同的语言那样近似一一对应的关系。文言文太简略了，且爬到的白话解释并不单是对文言文对翻译，而是非常详细的解释。两三个字的文言文，有时甚至对应好几十个字的白话文。

整理了好久才整理了一点点，先跑了一下试试，果然不太行。还是没办法推理。

失去耐心后，决定暂时放弃文言文对白话文的任务。把上面【月来客栈】的项目中的任务做了反转。他的任务是德语翻译成英语的。我做成了英语翻译德语的。这才成功。

古文和现代文之间的转换这个，暂时先不做了。但应该不会一直放弃，等以后有精力了再去完成吧。

## 2024-11-13

终于完成了自己手写的代码。一开始还是 loss 可以下降但无法预测，调了半天也没有好转，想到可以把官方代码抽出来一点点删除不需要的部分调试看。昨天调了一天终于调通了。对比才发现到底还是 attention, add, norm 这一套的排列组合顺序不对。

果然这些细节还是要自己实现一边才能知道啊。

最后对比了一下使用自己实现的 transformer 和官方的 transformer 训练的速度，自己实现的代码一个 epoch 大概需要 61 秒，官方的大概需要 42 秒。差别还是挺大的。