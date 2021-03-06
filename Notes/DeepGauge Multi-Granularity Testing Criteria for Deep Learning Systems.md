## DeepGauge: Multi-Granularity Testing Criteria for Deep Learning Systems

提出了一系列旨在呈现测试平台多方面描述的多粒度深度学习系统测试标准

#### 背景

目前没有一套系统的方法来衡量深度学习系统的测试充分性，当前的研究注重于将深度学习系统的高准确性作为测试标准，但是存在以下问题：

* 仅从$DL$的输出来度量其质量是不可靠的，因为没有涉及$DL$内部的逻辑
* 仅基于$DL$输出的标准很大程度上取决于测试数据的代表性，高性能的$DL$输出并不意味着该系统是最通用的（比如对抗性样本）
* 通过系统测试的$DL$系统应该能够在某种程度上抵御所有类型的对抗攻击，但是即使目前最全面的衡量标准也无法消除对抗性攻击带来的风险

因此需要一套测试标准，在各种粒度级别上监视和评估神经元活动以及内在网络连接性。如果没有一套全面的标准，很难设计出覆盖$DNN$的不同学习逻辑和规则的测试，从而遗漏错误行为，对测试质量的评估也会有偏差，测试结果的置信度可能会偏高

#### 主要贡献

* 提出了一套标准，从不同级别和角度了解$DNN$与测试数据质量，衡量一组输入在多大程度上涵盖了可能导致深度学习缺陷的神经元
* 提出的标准可以高效地检测$DNN$中的缺陷，并且覆盖率越高表明检测出缺陷的概率越大
* 提出的不同标准在具有不同网络复杂度和数据集的$DNN$上表现不同，这些标准可以潜在地帮助对测试$DNN$的理解

#### 前提条件

将编程语言逻辑执行到传统软件的过程模拟为$DNN$的连接强度，即权重，并研究之间的差异

* **传统软件中的覆盖标准**

    给定一组测试数据，将测试数据输入到程序中，并比较程序运行结果与预期结果来验证程序的正确性，具有较高测试覆盖率的程序通常表明其包含缺陷的概率较小

    软件测试覆盖标准包括：代码级$(code\ level)$：语句覆盖、分支覆盖、数据流覆盖、变异测试和模型级$(model\ level)$：状态覆盖、过渡覆盖，其中：

    * 语句覆盖：度量是否每一条指令都被执行过
    * 分支覆盖：度量是否控制结构的每一条分支都被覆盖
    * 数据流覆盖：度量是否每一个变量定义都被覆盖，以检测数据流异常
    * 基于模型的覆盖：通过抽象的行为模型覆盖更多程序行为

* $\mathbf{DNN}$**的架构**

    $DNN$由训练数据驱动，包括输入层、输出层、隐含层，每个神经元都是经过激活函数计算的计算单元

#### $\mathbf{DeepGauge}$介绍

目前的$DNN$测试主要依赖于预测准确性，缺乏系统性的测试覆盖标准，而由于传统软件与$DNN$之间的差别，传统软件的覆盖标准都不能直接应用于$DNN$的测试

$DNN$的行为可以被分为两类：$major\ function\ behavior$和$corner-case\ behavior$

**定义**：$N=\{n_1,n_2,...\}$为$DNN$的神经元集合；$T=\{x_1,x_2,...\}$为测试集；$\phi(x,n)$表示神经元$n$在输入为$x$时的输出；$L_i(1\le i\le l)$表示第$i$层神经元的集合

* **神经元级的覆盖标准**$(\mathbf{Neuron-Level\ Coverage\ Criteria})$

    令$high_n$和$low_n$分别为神经元$n$输出值的上下边界，对于输入$x$，如果满足条件$\forall n\in N：\phi(x,n)\in[low_n,high_n]$，则称$DNN$位于其主要功能区$major\ function\ region$，如果神经元$n$的值在$[low_n,high_n]$范围外，即$(-\infin,low_n)\cup(high_n,+\infin)$，则称之为极端情况区$corner-case\ region$

    从训练数据获得的神经元的输出值不会出现在极端情况区中，而如果测试输入与训练数据的统计分布类似，那么测试输入也很少会出现在极端情况区中，然而$DNN$中的缺陷可能他会出现在极端情况区中

    * $\mathbf{k-multisection\ Neuron\ Coverage}$

        $k-multisection$神经元覆盖度量了测试输入$T$对主要功能区$[low_n,high_n]$的覆盖程度，将$[low_n,high_n]$划分为$k(k>0)$个部分，用$S_i^n$表示第$i(1\le i\le k)$部分的神经元输出值，如果$\phi(x,n)\in S_i^n$，则称第$i$部分被输入$x$覆盖，所以对于输入$T$和神经元$n$：

        每个神经元的覆盖定义为：$\frac{|\{S_i^n|\exist x\in T:\phi(x,n)\in S_i^n\}|}{k}$，即神经元$n$覆盖的$k$个部分的比例

        $DNN$的$k-multisection$神经元覆盖定义为：$KMNCov(T,K)=\frac{\sum_{n\in N}|\{S_i^n|\exist x\in T:\phi(x,n)\in S_i^n\}}{k\times|N|}$

    * $\mathbf{Neuron\ Boundary\ Coverage}$

        如果$\phi(x,n)$在$(-\infin,low_n)$或$(high_n,+\infin)$中，则称对应的极端情况区被覆盖，并且：

        * $UpperCornerNeuron=\{n\in N|\exist x\in T:\phi(x,n)\in(high_n,+\infin)\}$
        * $LowerCornerNeuron=\{n\in N|\exist x\in T:\phi(x,n)\in(-\infin,low_n)\}$

        神经元边界覆盖定义为：$NBCov(T)=\frac{|UpperCornerNeuron|+|LowerConerNeuron|}{2\times|N|}$

    * $\mathbf{Strong\ Neuron\ Activation\ Coverage}$

        强神经元激活覆盖只考虑与$upper-corner\ case$有关的覆盖，定义为：$SNACov(T)=\frac{|UpperCornerNeuron|}{|N|}$

        $p.s.$ <u>思考：强神经元激活覆盖存在的意义是什么？为什么在神经元边界覆盖的基础上再加上它？</u>

* **层级覆盖标准**$(\mathbf{Layer-Level\ Coverage\ Criteria})$

    活跃$(hyperactive)$的神经元可能会潜在地提供更多的$DNN$信息，因此层级覆盖标准主要是用活跃神经元，以及活跃神经元之间的组合来描述$DNN$的行为

    **定义**：对于输入$x$以及神经元$n_1$和$n_2$，如果$\phi(x,n_1)>\phi(x,n_2)$，则称$n_1$比$n_2$更活跃；用$top_k(x,i)$表示第$i$层输出值前$k$大的神经元

    * $\mathbf{Top-k\ Neuron\ Coverage}$

        度量有多少神经元至少一次是当前层的前$k$活跃的神经元，定义为此类神经元的数量与神经元总数的比值：$TKNCov(T,k)=\frac{|\cup_{x\in T}(\cup_{1\le i\le l}top_k(x,i))|}{|N|}$

        $DNN$同一层上的神经元的功能往往比较相似，不同层的活跃神经元是表征$DNN$主要功能的重要指标，因此，为了更全面地测试$DNN$，测试数据集需要发现更多的活跃神经元

    * $\mathbf{Top-k\ Neuron\ Patterns}$

        对于指定的输入$x$，得到的$top$-$k$神经元可以看成一组$pattern$，比如在下图中，绿色神经元即一组$top$-$2\ pattern$，$pattern$定义为一组元素$2^{L_1}\times2^{L_2}\times...\times2^{L_l}$，其中$2^{L_i}$表示第$i(1\le i\le l)$层神经元的子集的集合，对于指定的测试输入集$T$，$top$-$k\ neuron pattern$的数量为：$TKNPat(T,k)=|\{(top_k(x,1),...,top_k(x,l))|x\in T\}|$

        $top$-$k\ neuron\ pattern$表示每层活跃神经元的不同激活情况

<img src="..\Images\DeepGauge-figure_1.png" alt="DeepGauge-figure1" style="zoom: 25%;" />

#### 实验评估

* **评估对象**

    * 数据集：$MNIST$、$ImageNet$
    * $DNN$模型：对$MNIST$使用$LeNet-1$、$LeNet-4$、$LeNet-5$；对$ImageNet$使用$VGG-19$、$ResNet-50$
    * 对抗样本生成：使用对抗样本，通过给定输入的微小扰动来检测$DNN$的潜在缺陷，使用了$FGSM$、$BIM$、$JSMA$、$Carlini/Wagner(CW)$四种对抗样本生成方法生成对抗样本

* **评估设置**

    * $MNIST$：首先使用$60000$个训练数据对每个$DNN$进行运行时分析，获得$DNN$神经元输出信息，对每个$DNN$使用$10000$个测试数据获取对应的测试覆盖率，然后对于四个对抗样本集，分别与原始测试集合并，变成四个$20000$大小的测试数据集，来测试对抗性测试数据如何通过提供的覆盖标准来增强缺陷检测能力

        下表中，$u$和$l$分别表示神经元输出上界和下界，$\sigma$表示神经元输出的标准差，在原有标准的基础上，还增加了同时将神经元输出上下界同时扩展$0.5*\sigma$和$\sigma$的标准，因此一共有14种标准设置，评估配置一共有 3($DNN$模型) × 5(数据集) × 14(标准配置) = 210 种

    ![DeepGauge-table2](..\Images\DeepGauge-table_2.png)

    * $ImageNet$：由于$ImageNet$数据集太大，并且两个$DNN$模型也相对复杂，因此从原始$ImageNet$测试数据集的每个标记类中随机抽取图像作为评估的测试数据，并使用$FGSM$、$BIM$和$CW$生成对抗样本（$JSMA$由于数据集的大小与复杂度构建对抗样本失败），一共使用 2($DNN$模型) × 4(数据集) × 14(标准配置) = 112 种测试评估的实验配置

* **实验结果**

    * $MNIST$：

        与原始$MNIST$测试数据集相比，对抗样本在不同标准上的覆盖率更高，表明对抗样本探索了$DNN$中未被原始测试覆盖的内部状态，因此也表明生成能提高覆盖率的测试样本可以触发$DNN$的更多状态，从而发现$DNN$中的缺陷

    ![DeepGauge-table3](..\Images\DeepGauge-table_3.png)

    * $ImageNet$：

        * $VGG-19$和$ResNet-50$的模型更大更复杂度，潜在地导致覆盖率比$LeNet$低，但这不意味着复杂的模型获得的覆盖率就会比较低

        * 与$MNIST$相比，$ImageNet$在神经元边界覆盖和强神经元激活覆盖这两个标准上的覆盖率更高
        * 从$top$-$k$神经元覆盖的实验结果来看$DNN$的许多神经元已被触发为前$k$活跃的神经元，虽然$top$-$k$标准的提升没有其他标准明显，但是对抗样本也触发了更多能检测出隐藏缺陷的神经元
        * 往往每层最活跃的神经元都是固定的几个，说明每层最活跃的神经元可以描述神经网络的主要特征，它们的组合也可以发现输入数据的结构差异，而当$k$被正确选定时，$top$-$k$ $neuron$ $pattern$也可以最大程度地区分数据数据，说明涵盖更多的前$k$活跃的神经元的$pattern$将更有可能发现$DNN$的缺陷

    ![DeepGauge-table4](..\Images\DeepGauge-table_4.png)

* **调查结果**$(\mathbf{Findings})$

    实验结果证明提出的$DNN$测试标准的有效性，也能从多方面解释对抗技术间的差异：

    * $MNIST$和$ImageNet$的原始测试数据以及对抗样本都在主要功能区和极端情况区发现了$DNN$的缺陷，说明$DNN$的缺陷在两个区域都可能出现，因此两个区域都需要被广泛测试
    * 生成的对抗样本都能提高提出的标准的覆盖率，由于对抗样本能够揭示深度学习系统的缺陷，所以提升测试标准的覆盖率可以增强缺陷检测能力，而提出的测试标准可以发现$DNN$在原始样本和对抗样本之间的内部行为差异
    * 测试数据在$k-multisection$神经元覆盖标准上获得的覆盖率比在神经元边界覆盖标准和强神经元激活覆盖标准上的覆盖率更高，说明测试数据比之极端情况区覆盖了更多的主要功能区
    * 强神经元激活覆盖比神经元边界覆盖的覆盖率更高，这可能是由激活函数导致的，$RELU$激活函数可能会使神经元的低值区域比高值区域小很多

* **评论**$(\mathbf{Remarks})$
    * 对于神经元边界覆盖和强神经元激活覆盖，神经元输出值的上界越高（下界越低），覆盖率的增量就越小
    * 对于$top$-$k$神经元覆盖，$k$越大，覆盖率的增量越低
    * 对于$top$-$k\ neuron\ pattern$，$k$越大，$pattern$数量的增量越多
    * 四种对抗样本生成技术在神经元边界覆盖和强神经元激活覆盖两个标准上的多样性较好，因此能够覆盖$DNN$的不同的潜在缺陷

* **与**$\mathbf{DeepXplore}$**的神经元覆盖标准$\mathbf{DNC}$的比较**

    $DNC$的一个关键参数是用户自定义的阈值，即如果神经元的输出值大于阈值，则认为神经元是被覆盖的，将$DNC$与上述提出的标准设置相同的实验，并分别将阈值设置为0、0.2、0.5、0.75，发现：$DNC$在所有配置实验下，在原始测试样本和对抗样本上的实验结果几乎是一样的，说明$DNC$并不能区分原始测试样本和对抗样本，然而为了更细粒度地检测$DNN$的缺陷，这样的差异是必须被检测到的

    在对$DNC$进行深度探索后发现$DNC$存在以下局限性：

    * $DNC$对不同的神经元设置了相同的激活阈值，但是不同神经元的输出分布不同，因此对所有神经元使用相同的阈值而不考虑神经元功能分布的差异，会大大降低其准确性 $e.g.$ 如果给定的神经元的均值和标准差非常小，即使给定的阈值很小，但只要稍微大一点，就会导致神经元无法覆盖
    * $DNC$根据要分析的每个输入图像在每一层上神经元的最大和最小值来标准化神经元输出，而因为对于每个输入，每一层的最大和最小输出都可能会变化，就会导致相同的归一化后的激活阈值对于不同的输入有不同的含义

    $DNC$消除了不同输入之间激活幅度的相关性，而这是神经元激活覆盖范围的重要属性，而上述提出的标准则指定了神经元输出的上下界，即依赖于训练集数据的统计信息

* **有效性威胁及讨论**
    * 数据集与$DNN$选择的威胁，因此选择了被广泛研究的$MNIST$和$ImageNet$数据集，以及知名的预训练模型
    * 覆盖标准定义中的可配置超参数是另一个威胁，因此使用不同的设置来评估每个标准，并分析参数对覆盖标准的准确性的影响，但可能还是不能涵盖最佳参数
    * 训练数据的质量也是一个威胁，因此使用了公开可用的且经过良好训练的$DNN$模型及训练数据















