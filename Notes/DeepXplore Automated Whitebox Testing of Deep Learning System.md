## DeepXplore: Automated Whitebox Testing of Deep Learning System

#### 背景

深度学习在无人驾驶汽车、恶意软件检测等对安全性要求比较高的领域中的应用越来越广泛。在这些领域中，深度学习系统对极端输入的正确性和可预测性是十分重要的。然而现有的深度学习测试依赖于手动标记的数据，导致很难在罕见的输入上暴露错误行为

当前大型深度学习系统的自动化系统测试的挑战有两个方面：

1. 如何生成”能触发深度学习系统逻辑的不同部分，并能发现不同类型错误行为的“输入
2. 如何不通过人工标记或检查，来识别深度学习系统的错误行为

#### 目前$DNN$测试的局限性

1. 需要花费巨大的人力才能为一个目标任务提供正确的标记$(label)$
2. 现有的测试方法没有覆盖$DNN$的不同规则，因此测试输入通常无法发现$DNN$的不同错误行为
3. 低覆盖率的$DNN$测试输入会导致$DNN$的很多行为无法被探索到

#### 主要贡献

1. 使用神经元覆盖作为度量标准，以估计一组测试输入探索的深度学习逻辑$(DL\ logic)$的数量
2. 证明了“寻找能够触发许多不同行为，并且能实现高神经元覆盖率的深度学习测试输入”的问题可以被转化为一个联合优化问题，并提出了一种基于梯度的算法来解决这个问题
3. 实现了第一个白盒深度学习测试框架$DeepXplore$
4. 由$DeepXplore$生成的测试输入可以被用来重训练模型，并能提高$3\%$的分类准确性

#### $DeepXplore$简介

因此设计并实现了一种用于系统性测试深度学习系统的白盒框架$DeepXplore$，主要步骤如下：

1. 介绍神经元覆盖，用来系统地测量由测试输入执行的深度学习系统的各个部分
2. 使用功能相似的多个深度学习系统作为交叉引用的$oracle$，以避免手动检查
3. 说明为什么寻找深度学习的（能够触发许多不同行为，并且能实现高神经元覆盖率的）测试输入，可以被转化为联合优化问题，并且使用基于梯度的搜索算法来解决

$DeepXplore$生成的测试输入比同等数量输入的随机选择输入和对抗性输入分别高$34.4\%$和$33.2\%$

使用$DeepXplore$生成的输入来重训练模型，比使用随机选择或对抗性输入来重训练模型的准确性高$3\%$

![DeepXplore-workflow](..\Images\DeepXplore-workflow.png)

如图所示，$DeepXplore$首先将未被标记的输入作为种子输入，然后生成“覆盖大量神经元并能触发不同行为”的测试输入，将其转化为一个联合优化问题，最终生成满足条件的输入

* **定义**

神经元覆盖率$(Neuron\ Coverage)$：$DNN$所有测试输入中，被唯一激活的神经元数量与所有神经元数量的比值。可以公式化表示为：$NCov(T,x)=\frac{|\{n|\forall x\in T,out(n,x)>t\}|}{|N|}$，其中$N=\{n_1,n_2,...\}$表示$DNN$中的所有神经元，$T=\{x_1,x_2,...\}$表示所有测试输入，$out(n,x)$表示输入为$x$的神经元$n$对应的输出值，$t$是神经元的激活阈值

梯度$(Gradient)$：定义梯度$G$的计算公式如下：$G=\nabla_xf(\theta,x)=\partial y/\partial x$，其中$\theta$和$x$分别表示$DNN$的参数和输入，$y=f(\theta,x)$表示一个神经元的输出，因此根据微积分的链式求导法则，梯度就是输出$y$到输入$x$的偏导数

* $\mathbf{DeepXplore}$**算法**

与传统软件相比，$DNN$测试输入生成过程的主要优势在于，测试生成过程一旦定义为优化问题，就可以使用梯度上升方法来高效解决

1. **最大化差异行为**$(Maximizing\ Differential\ Behaviors)$

    在被测$DNN$中生成能触发不同行为的测试输入，即不同的$DNN$会将同一个输入分为不同的类。定义$n$个$DNN$模型的函数为$F_{k\in 1...n}:x\rightarrow y$，给定任意的$x$，其在每个$DNN$的输出结果相同，算法的目的是找到$x'$使得至少有一个$DNN$将其划分为不同的类

    ![DeepXplore-Algorithm1](..\Images\DeepXplore-Algorithm1.png)

    定义$F_k(x)[c]$为$F_k$将输入$x$分类为结果$c$的概率，$\lambda_1$是用来平衡$F_{k\ne j}$（分类结果不变）和$f_j$（分类结果变化）之间的客观项的参数

    首先随意选择一个神经网络$F_j$并最大化其目标函数： $obj_1(x)=\sum_{k\ne j}F_k(x)[c]-\lambda_1\cdot F_j(x)[c]$（23-29行），因为每个$F_{k\in 1...n}$都是不同观点，所以目标函数的最大值可以通过使用公式$\frac{\partial obj_1(x)}{\partial x}$迭代地计算梯度来获得（8-14行）

2. **最大化神经元覆盖率**$(Maximizing\ Neuron\ Converage)$

    通过迭代地选择未被激活的神经元并修改其输入值使神经元被激活，来最大化神经元覆盖率。对于神经元$n$，要使其输出值最大，即使得$obj_2(x)=f_n(x)>t$，其中$t$是神经元的激活阈值，$f_n$表示神经元$n$以$x$为输入产生的输出（30-35行），由于$f_n(x)$可微，其梯度为$\frac{\partial f_n(x)}{\partial x}$，因此可以用梯度上升来计算其最大值。可以一次选择多个神经元进行激活

3. **联合优化**$(Joint\ Optimization)$

    将对$obj_1$和$f_n$的优化联合，并最大化联合优化目标函数：$obj_{joint}=(\sum_{i\ne j}F_i(x)[c]-\lambda_1F_j(x)[c])+\lambda_2\cdot f_n(x)$，其中$\lambda_2$是用来平衡两个目标函数的参数，$n$是被随机选择用来激活的神经元。因为$obj_{joint}$的所有项都是可微的，所以可以使用梯度上升修改$x$来联合使其最大化（1-14行）

4. **特定域的约束**$(Domain$-$Specific\ Constraints)$

    生成的测试输入需要满足特定域的约束，比如$x$的像素值要在$0\sim 255$之间，因此要让每次梯度上升的迭代满足特定域的约束条件。由于初始输入$x_{seed}$一定是满足约束条件的，因此只要在每次迭代的过程中，保证梯度上升迭代后的$x$仍然满足约束即可。算法通过修改$grad$值来满足要求（13行），使得$x_{i+1}=x_i+s\cdot grad$仍然满足约束条件

5. **算法中的超参数**$(Hyperparameters\ in\ Algorithm)$

    * $\lambda_1$（29行）：$\lambda_1$的值越大，表明在降低指定$DNN$预测值的优先级越高，反之表明在维持其他$DNN$的预测结果上有更大的权重
    * $\lambda_2$（10行）：$\lambda_2$越大表明越侧重于覆盖不同的神经元，反之则侧重于产生更多的差异诱导测试输入
    * $s$（14行）：$s$表示梯度上升迭代过程中的步长，较大的$s$可能导致在局部最优值振荡，而较小的$s$可能需要迭代更多次才能达到最优值
    * $t$（33行）：t是确定每个神经元是否被激活的阈值。 随着t的增加，寻找激活神经元的输入变得越来越困难

$p.s.$ 对$\lambda_1$和$\lambda_2$的理解为，使得需要平衡的两个参数，不能因为其中一个的值比较大而导致影响比较大，因此相当于一个系数，用来平衡两个参数的值之间的影响

#### 实验设置

* **测试数据集和**$\mathbf{DNN}$

    选取了五个最流行的数据集：$MNIST$，$ImageNet$，$Driving$，$Contagio/VirusTotal$，$Drebin$，并对每个数据集使用三个$DNN$进行评估，所有$DNN$都是预训练的，或是被作者训练为在对应数据集上与最新模型性能近似的

    * $MNIST$：手写数字数据集，每张图片是$28 \times 28$像素的$0 \sim 9$的数字，包含60000个训练样本和10000个测试样本。基于$LeNet\ family$在$MNIST$上构建了$LeNet-1$、$LeNet-4$、$LeNet-5$三个$DNN$
    * $ImageNet$：大型图像数据集，包括超过1000万张手动标注的图片。在$ImageNet$上构建了$VGG-16$、$VGG-19$、$ResNet50$三个$DNN$
    * $Driving$：是$Udacity$自动驾驶汽车数据集，包括101396个训练样本和5614个测试样本。基于$Nvidia$的$DAVE-2$自动驾驶汽车结构，并略微修改了配置，构建了三个$DNN$：$DAVE-orig$、$DAVE-norminit$和$DAVE-dropout$
    * $Contagio/VirusTotal$：包含不同良性和恶意$PDF$文档的数据集，从$Contagio$中选择5000个良性和12205个恶性$PDF$文档作为训练集，从$VirusTotal$中选择5000个恶性$PDF$，并从$Google$爬取5000个良性$PDF$作为测试集。由于没有开源的$DNN$，因此作者构建了性能与$SVM$相似的三个$DNN$
    * $Drebin$：包含129013个安卓应用程序，其中123453个是良性的，5560个是恶意的。选取了3个$DNN$，并使用$66\%$随机选取的安卓应用程序作为训练集，剩余的作为测试集

* **特定领域的约束**

    为了确保生成的测试输入能实际有用，需要通过应用特定域的约束来确保生成测试是的有效性

    * 图像约束$(Image\ Constraints)$：$MNIST$、$ImageNet$、$Driving$

        使用三种不同类型的约束来模拟图像的不同环境条件：

        * 模拟不同强度的光线的照明效果
        * 遮挡图片的以个小矩形区域（模拟遮挡摄像头的某些部分）
        * 遮挡图片的多个黑色小矩形区域（模拟灰尘对相机镜头的影响）

    ![DeepXplore-Different_lighting_condition](..\Images\DeepXplore-Different_lighting_condition.png)

    ![DeepXplore-Occlusion_with_a_single_small_rectangle](..\Images\DeepXplore-Occlusion_with_a_single_small_rectangle.png)

    ![DeepXplore-Occlusion_with_multiple_tiny_black_trctangles](..\Images\DeepXplore-Occlusion_with_multiple_tiny_black_trctangles.png)

    * 其他约束$(Other\ Constraints)$：$Drebin$、$Contagio/VirusTotal$
        * 对于$Drebin$，仅允许修改与$Android\ manifest$文件相关的功能，仅允许添加功能，不允许删除功能，以确保不会由于权限不足而影响应用程序功能，因此在计算梯度之后，$DeepXplore$仅修改梯度大于零的$manifest$特征
        * 对于$Contagio/VirusTotal$，$VirusTotal$数据集中的四个文件已存在于$Contagio$中，因此把它们删除

#### 结果

$DeepXplore$从所有被测试的$DNN$中发现了上万个错误行为，具体数据如下：

![DeepXplore-table2](..\Images\DeepXplore-table_2.png)

根据经验选择超参数，以使得联合优化目标函数的值最大化，选用的参数也如上图所示

* 使用神经元覆盖的好处$(\mathbf{Benefits\ of\ Neuron\ Coverage})$

    有研究表明，每一个神经元都能单独地提取输入的一个特征，也就是说，每个神经元都能与其他的神经元学习不同的规则，并且不同分类的输入比相同分类的输入更容易激活不同的神经元

    * 神经元覆盖和代码覆盖的对比

        * 使用同样的测试样本计算代码覆盖率和神经元覆盖率，发现代码覆盖率很容易就达到了$100\%$，而神经元覆盖率一直没有超过$34\%$
        * 神经元覆盖率受$DNN$和测试输入的影响显著

    * 神经元覆盖对$DeepXplore$发现差异诱导输入的影响

        最大化神经元覆盖的目的是生成多样性的差异诱导输入

        * 使用同一个种子生成的所有测试输入与原始输入的$L1$距离来度量生成的差异诱导输入的多样性，$L1$距离定义为生成图片与原始图片每个像素的绝对距离之和，结果显示神经元覆盖的提高能增加生成的输入的多样性

        ![DeepXplore-table5](..\Images\DeepXplore-table_5.png)

        * 神经元覆盖率越高就越难提升，而很小的提升就能显著地提高测试的多样性
        * 仅凭差异输入的数量无法衡量针对视觉任务生成的测试的质量，因为进行微小的更改就可以创建很多有相同根本原因的图像

    * 不同类别的输入对神经元激活的影响

        不同分类的输入能激活更多不同的神经元，而不同类别的输入是通过匹配不同的$DNN$规则来检测的，所以神经元覆盖可以有效地估计激活的不同规则的数量

* $\mathbf{DeepXplore}$**的性能**

    使用<u>神经元覆盖率</u>和<u>运行时间</u>两个指标来评估$DeepXplore$的性能

    * **神经元覆盖率**$(\mathbf{Neuron\ Converage})$

        比较了三种方法生成相同数量的测试输入时的神经元覆盖率：

        * $DeepXplore$
        * $Adversarial\ Testing$
        * $Random\ Select$

        结果显示：

        * $DeepXplore$平均比随机测试和对抗性测试多覆盖了$34.4\%$和$33.2\%$的神经元
        * 神经元激活阈值$t$会极大地影响神经元覆盖率（阈值$t$较高会导致神经元很难被激活）

    ![DeepXplore-figure9](..\Images\DeepXplore-figure_9.png)

    * **运行时间与种子输入数量**$(\mathbf{Excution\ Time\ and\ Number\ of\ Seed\ Inputs})$

        考虑除全连接层以外所有神经元被激活时的运行时间（**<u>*为什么全连接层的部分神经元很难被激活？*</u>**）

        结果显示$DeepXplore$在寻找差异诱导输入和增加神经元覆盖率范围方面非常有效

    ![DeepXplore-table8](..\Images\DeepXplore-table_8.png)

    * **超参数的选择**$(\mathbf{Different\ Choice\ of\ Hyperparameters})$

        结果表明$s$和$\lambda_1$的最优值由$DNN$和数据集决定，而对于所有数据集，$\lambda_2$的最优值是$\lambda_2=0.5$，不同阈值$t$的影响在$Figure\ 9$中已经给出

        使用找到第一个差异诱导输入的时间作为比较不同超参数选择的度量，因为找到第一个差异诱导输入比增加输入数量要难很多

    ![DeepXplore-table9](..\Images\DeepXplore-table_9.png)

    ![DeepXplore-table10.png](..\Images\DeepXplore-table_10.png)

    ![DeepXplore-table11](..\Images\DeepXplore-table_11.png)

    * **使用**$\mathbf{DeepXplore}$**测试相似模型**

        对于决策边界非常相似的$DNN$，$DeepXplore$可能无法在合理时间内找到差异诱导输入，为了估计使得$DeepXplore$失败时的$DNN$相似度，使用控制变量法测量每一个指标生成第一个差异诱导输入时所需的变化

        * 训练样本数量
        * 每个卷积层的$filter$数量
        * 训练的$epoch$数量

        结果显示，随着$DNN$之间差异数量的减少，寻找引起差异的输入所需的的迭代次数增加，$DNN$之间的差异越小，找到差异引起的测试输入越困难。而$DeepXplore$在寻找相似$DNN$之间的差异时的效果很好

![DeepXplore-table12](..\Images\DeepXplore-table_12.png)

* **使用**$\mathbf{DeepXplore}$**优化**$\mathbf{DNN}$

    $DeepXplore$生成的测试输入的作用：

    * 扩大训练集，以提高$DNN$的准确性

        用生成的测试输入来提高$DNN$的准确性在对抗性输入上也有应用，区别在于对抗性输入需要手动标记标签，而$DeepXplore$可以采用投票的方式自动生成标签（假设大多数$DNN$分类正确）

        将$DeepXplore$生成的测试输入与随机选择和多样性测试作比较，结果显示$DeepXplore$生成的测试输入多提升了$1\%\sim 3\%$的准确率

    ![DeepXplore-figure10](..\Images\DeepXplore-figure_10.png)

    * 检测潜在的损坏的训练数据

        使用一个$DNN$训练$MNIST$数据集的60000个手写数字，另一个$DNN$训练同一个数据集的污染版本，其中有$30\%$的数字9被标记为1。使用$DeepXplore$生成的测试输入中，被未被污染版本和被污染版本的$DNN$识别为9和1的部分，然后从结构相似性方面搜索训练集中最接近$DeepXplore$生成的输入的样本，并将其识别为污染数据，准确率达到$95.6\%$

#### 局限性

$DeepXplore$使用了差异测试，因此会存在一定的局限性：

* 差异测试需要功能相似的两个不同的$DNN$，如果两个$DNN$之间的差异较小的话，$DeepXplore$需要花费更多的时间去寻找差异诱导的输入
* 只有至少一个$DNN$与其他$DNN$的结果不同时，差异测试才能检测到错误行为。实际上大多数$DNN$都是独立构造训练的，因此犯相同错误的概率很低









