## Testing File System Implementations on Layered Models

##### 通过构建分层模型和抽象工作负载的优化，提出了一种用于解决工作负载生成问题的新方法，此方法实例化为三层文件系统模型，用于生成文件系统工作负载。

#### 背景介绍

为了验证复杂的文件系统，需要生成高质量的测试输入（即工作负载workloads）以驱使系统达到潜在的错误状态。生成高质量的文件系统调用序列(file system call sequences)对测试文件系统很重要，但是也由于输入空间过大而极具挑战性。

现有的模糊测试方法类似系统调用的随机生成器，因为基于覆盖信息不能达到文件系统的不同状态。而由于复杂系统的复杂性，为其构建一个精准而完善的模型是不现实的。因此，模型检查(model checking)，基于模型的测试(model-based testing)或格式验证(format verification)仅限于核心功能的一小部分。











