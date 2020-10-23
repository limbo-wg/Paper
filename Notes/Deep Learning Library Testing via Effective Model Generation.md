## **Deep Learning Library Testing via Effective Model Generation**

提出了一种针对**Deep Learning library**的测试方法**LEMON**

###### 当前对深度学习library的测试存在两个挑战：

 	1. 缺少能够触发DL library bug的model
 	2. DL model很难暴露DL library中的bug

###### 目前已有方法CRADLE的局限性：

1. 依赖于现有的model去触发library bug，这样仅能探索少量代码空间
2. 真实的bug与不确定因素之间的inconsistency degree较小，难以设置阈值区分

###### 针对CRADLE遇到的局限性提出的解决方案：

1. 提出model的变异规则，以弥补model数量不足的情况
2. 提出一种启发式的策略LEMON来引导模型生成，以扩大真实bug与不确定因素之间的inconsistency

两种DL library的测试指标：D_CLASS和D_MAD

**D_CLASS**用于分类模型，根据预测值与真实值的排名计算每个预测结果的score，然后计算两个score之间的差异

**D_MAD**用于分类与回归模型，首先分别计算两个输出向量与真实值向量之间的差异：$\delta_{O,G}=\frac{1}{m}\sum_{r=1}^{n}|o_i-g_i|$，然后计算两个输出向量之间的差异$D\_MAD_{G,O_j,O_k}=\frac{|\delta_{O_j,G}-\delta_{O_k,G}|}{\delta_{O_j,G}+\delta_{O_k,G}}$。

###### 模型的变异规则：整层变异和层内变异

**整层变异**：即对整个隐藏层进行变异，包括Layer Removal、Layer Switch、Layer Copy、Layer Addition、Multi-Layers Addition、Activation Function Removal、Activation Function Replace

**层内变异**：即对神经元的变异，包括Gaussian Fuzzing、Weights Shuffling、Neuron Activavtion Inverse、Neuron Effect Block、Neuron Switch

###### 启发式方法LEMON：

使用D_MAD作为测试指标计算inconsistency，主要步骤分为两步：

1. 选择需要变异的种子模型
2. 选择变异规则

***1.种子模型的选择***

定义$ACC(M)$来计算对于模型M，在所有输入下DL library pair之间的inconsistency之和：

$ACC(M)=\sum_{i=1}^{n}\sum_{j,k=1}^{m}D\_MAD_{G,O_{ji},O_{ki}}(k>j)$

如果变异后的模型的$ACC(M)$值增大了，则认为模型的inconsistent-degree扩大了

对于每个模型，其被选中的次数越多，那么之后被选中的可能性越小，用$c_i$表示模型$i$被选中的次数，然后定义$score_i=\frac{1}{c_i+1}$，然后使用随机选择的方法，模型的$score_i$值越大，则其被选中的可能性越大，可以计算模型$i$被选中的概率为：$p_i=\frac{score_i}{\sum_{k=1}^{r}score_k}$

***2.变异规则的选择***

定义$Ratio(MU)$为变异规则$MU$的priority score，$MU$使得$ACC(M)$增大的次数越多，$Ratio(MU)$越大，然后将所有的变异规则按照$Ratio(MU)$排序

由于变异规则的选择只首最近一次选择的变异规则的影响，因此可以将其看做是一个马尔科夫链，则排名为$k$的$MU$被选中的概率为$Ps(X=k)=(1-p)^{k-1}p$，定义$k_a$和$k_b$分别为$MU_a$和$MU_b$的排名，则在上一个变异规则为$MU_a$的情况下，选择$MU_b$作为下一个变异规则的概率为：

$Pa(MU_b|MU_a)=\frac{Ps(MU_b)}{Ps(MU_a)}=(1-p)^{k_b-k_a}$，然后同样使用随机选择，$Pa(MU_b|MU_a)$越大，$MU_b$被选中的概率越大



