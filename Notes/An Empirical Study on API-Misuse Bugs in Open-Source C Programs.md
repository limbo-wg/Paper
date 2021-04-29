## An Empirical Study on API-Misuse Bugs in Open-Source C Programs

对API Misuse的原因、特征，以及API Misuse Detector的性能进行总结的一篇实证研究

#### 背景

隐性约束（如调用条件、调用顺序）的存在对正确使用API提出了挑战。尽管近20年已经提出了很多API Misuse的检测方法，近期的研究显示，API Misuse的问题仍然存在。检测方法主要分为**对源代码的静态分析**以及**对运行日志和trace的动态分析**。有些检测方法用来推荐使用正确的API参数和调用位置，有些使用代码复查、运行时检验、静态分析。这些检测方法的主要思想是，将**与最常用的情况偏差最大的使用方式**作为滥用。

#### 研究问题

**RQ1**: API Misuse bug的特征。现有研究给出了软件缺陷的分类，但是并没有给出API Misuse bug的产生原因以及解决方式。

**RQ2**: 现有检测方法的性能。当前最新的检测方法使用于什么类型的API Misuse bug。

#### 贡献

1. 进行了第一个对C程序的API Misuse bug的实证研究，对六个大规模不同领域的程序中的830个API Misuse bug进行研究
2. 提供了APIMU4C，一个C程序的API Misuse benchmark
3. 对APIMU4C上三个具有不同分析策略的现有开源静态检测器进行了定性和定量比较，分析了这些检测器的优缺点，为API Misuse bug检测提供新思路

#### 结果

* **API Misuse Characteristics**



* **Static Detector Performance**





























