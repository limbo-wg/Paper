#### API Misuse简介

在现代软件的开发过程中API被开发人员广泛使用，因为API提供了一种信息隐藏的机制，开发人员不需要知道具体的实现细节，通过调用API直接可以完成相应的功能，从而节约了软件的开发时间，提高了软件的开发效率。但是由于API本身使用的复杂性、文档资料的缺失或者使用者本身的疏漏等原因，开发人员有时会错误的使用API，它会在一定条件下引发运行错误，产生异常的结果或行为。

API通常有使用约束，例如调用顺序或者调用条件上的限制，API误用是指违反了使用约束的行为，最终可能会导致软件崩溃或缺陷。API误用有多种情况，比如多余的API调用、缺少了API调用或者错误的API调用等。例如在java中打开一个文件进行写入，如果写入完成后没有关闭文件就会导致文件写入失败。

----

#### PR-Miner: Automatically extracting implicit programming rules and detecting violations in large software code

提出一种通用且高效的隐式规则自动提取与反例检测方法，使用频繁闭合项集挖掘技术**从大量的软件代码库中**挖掘包含多种程序元素的编程模式，然后由编程模式产生编程规则；引入正序规则的概念，以避免从同一个编程模式中产生多个冗余规则。在此基础上提出一种高效的反例检测算法检测违反规则的程序片段。

一个简单的编程规则就是lock和unlock方法的调用，在调用lock方法后必须跟上一个unlock的方法来解锁。

在大型软件中还有很多隐式的规则，例如在数据库管理系统PostgreSQL中包含着如下一条隐式规则，调用SearchSysCache方法后需要调用ReleaseSysCache方法，如果违反了该条规则就会导致内存泄漏。

PR-Miner的主要功能是：**自动提取隐式的编程规则**和**自动检测违反这些编程规则的情况**

包含三个步骤：

* 从源文件中提取编程模式
* 从编程模式中生成编程规则
* 利用编程规则来检测缺陷

**PR-Miner主要思想**：从源代码中找出经常一起使用的元素，包括函数、变量和数据类型。为了发现程序元素之间的相关性，PR-Miner把问题转变成频繁项集挖掘的问题。每一个程序元素都被映射成一个数字，一个函数定义中的元素都被映射到一个项集中，这样整个程序就会产生许多项集。通过使用频繁项集挖掘算法得到频繁项集，再从频繁项集中推出编程规则。

通过挖掘算法得到频繁项集后，其中每个项集中程序元素的集合称为**编程模式**，表明这些语法成分相互关联或者经常一起被使用。例如：{$spin\_lock\_irqsave$, $ spin\_unlock\_irqrestore$}，集合中的两个元素都是函数的调用。

编程模式不同于编程规则，比如上面的编程模式可能推出两条**编程规则**：

1. 调用了$spin\_lock\_irqsave$，必须也调用$spin\_lock\_irqrestore$
2. 调用了$spin\_lock\_irqrestore$，必须也调用$spin\_lock\_irqsave$

这两条规则不一定都是成立的，所以接下来需要从编程模式中生成编程规则。生成编程规则的主要思想就是统计左边元素出现但是右边元素没有出现的次数，反之也是。

**结果**：他们的实验是在最新版本的Linux，Apache HTTP Server和PostgreSQL源代码上进行的，共计超过300万行代码。PR-Miner只需花费1~42秒就可以提取超过32000条编程规则，23个报告的缺陷被证实，并被开发人员所修复。

----

#### Doc2Spec: Inferring Resource Specifications from Natural Language API Documentation

上一篇文章利用数据挖掘算法从大量的代码中挖掘出编程规则，当不能获取大量的代码时，这种方法就受到了限制。这篇文章是从API文档出发的，作者认为软件库提供的API文档中提供了如何正确使用API的说明，而许多开发人员不愿意仔细阅读API文档，导致写出了和API文档中使用说明不一致的代码，从而引入了缺陷。

这篇文章提出了方法Doc2Spec，该方法使用了自然语言处理的技术来分析API文档，可以从API文档中推断出和资源使用有关的规约。在这篇文章中使用到的API文档的信息包括方法描述，以及类和接口的继承关系。

![API_Misuse_pic1](C:\Study\Paper\Images\API_Misuse_pic1.png)

**方法步骤**：

* 首先从API文档中提取出方法描述和继承关系

例如对javax.resource.cci.Connection中三个方法的描述：

1. createInteraction():“**Creates** an interaction associated with this **connection**.”

2. getMetaData():“**Gets** the information on the underlyingEIS instance represented through an active **connection**.”

3. close():“Initiates **close** of the **connection** handle at the application level.”

* 第二步是从方法描述中建立action-resource pair，对于每一个方法它的动作资源对表示着该方法对什么资源采取了什么动作。例如从上面三个方法的描述中可以提取出如下的动作资源对：

1. createInteraction():⟨create, connection⟩.
2. getMetaData():⟨get, connection⟩.
3. close():⟨close, connection⟩.

* 第三步是从得到的动作资源对和继承关系中推断出资源使用的规约，作者预先定义了一个规约模板，包括5种方法类型：creation, lock, manipulation, unlock和closure。它们分别代表着创建资源、给资源上锁、操纵资源、给资源解锁和释放资源。

![API_Misuse_pic2](C:\Study\Paper\Images\API_Misuse_pic2.png)

首先对于每一个类或者接口，**根据资源的名字和继承关系把方法聚到不同的类别中**，在这个例子中这三个方法属于同一个类，因为它们资源的名字相同并且在同一个接口中被申明。然后在同一个类别中，**根据动作把方法映射到不同的类型**，这三个方法的类别分别是:

1. createInteraction() → **creation** method.

2. getMetaData() → **manipulation** method.

3. close() → **closure** method.

然后推断出这三个方法的使用规约：

![API_Misuse_pic3](C:\Study\Paper\Images\API_Misuse_pic3.png)

上述方法得到的规约可以用来检测缺陷，可以**检查close函数是否在所有可能的执行路径上都被调用了**。

**结果**：作者在5个广泛使用的库的文档上进行了实验，结果表明该方法能以相当高的准确率推断出不同的规约，并且在开源项目中找到了已经被发现和还没被发现的和资源使用有关的缺陷。

----

#### Bugram: Bug Detection with N-gram Language Models

前面的两篇文章分别是从**代码库和API文档**中学出使用规约，这两种都是基于规则的方法。其中提到基于规则的方法经常**依赖于某个模式在项目中出现的频率**，当出现的**不够频繁时，就会忽略**了该规则，从而**导致检测时错过很多缺陷**。作者提出了一个新的方法Bugram，它使用了n-gram语言模型来检测缺陷，用学习得到的模型来预测程序中token序列出现的概率，概率低的序列可能是缺陷或者不常见的用法。

Bugram的工作流程主要分为三部分，首先是**从源文件中提取出token序列**，然后**利用序列建立n-gram模型**，最后**用模型来检测项目中的bug**

![API_Misuse_pic4](C:\Study\Paper\Images\API_Misuse_pic4.png)

从语法层面提取token的话只能检测到语法错误，因此作者从语义层面上提取token。为了从语义层面上检测bug，需要在语义层面上建立n-gram模型，因此作者关注了代码中的方法调用和控制流信息。通过使用JDT来解析源文件，从构造的抽象语法树中提取出token序列。利用得到的序列，作者使用n-gram模型来学习序列的概率分布。

for (int i=0; i<n; i++) { foo(i); } 将会被表示为token：[<FOR>, foo(), <END_FOR>]

例如：序列ABC，用3-gram模型来计算它的概率是P(ABC) = P(A) · P(B|A) · P(C|AB)。在缺陷检测阶段，Bugram计算出所有序列的概率，并且报告概率最低的序列作为疑似的缺陷。

**结果**：作者通过两方面的实验对Bugram进行了评估，一方面作者用Bugram检测了16个最新版本的java开源项目，共检测出59个bug，42个被人工证实为正确的，其中25个是真的bug，17个是需要被重构的代码片段。另一方面，作者把Bugram和另三个基于图或者规则的工具进行了比较，在这三个方法的评估中用到的14个java项目，Bugram共检测出21个真的bug，其中至少有10个不能被这三个工具检测出来。

----

#### MUTAPI: Exposing Library API Misuses via Mutation Analysis

第一篇和第二篇文章都是使用基于规则的办法来检测API误用，它们首先**通过大量的代码或者API文档**挖掘出正确的使用模式，然后用来检测API误用缺陷。这篇文章不同于前者的是，它**利用的是错误的模式来检测API误用缺陷**。作者认为从大量**正确的API**用法中挖掘出频繁模式的方法，在实际中可能会受到限制，因为有的API可能发布的时间比较短或者使用的人比较少，从而**没有足够的正确样例**；其次偏离频繁使用模式的用法**不一定是错误的，只是使用的比较少**。

针对上述问题，作者的观点：

* API误用可以看作是**正确用法的变异**，通过少量的正确用法可以mutate出大量API用法的变种。
* 其次，这些mutate出来的用法可以通过执行测试用例、分析执行信息来验证是不是API误用。从经过验证的API误用中，我们可以学习出该API是如何被误用的，然后用错误模式检测出API误用。

![API_Misuse_pic5](C:\Study\Paper\Images\API_Misuse_pic5.png)

这是项目Apache Commons Lang中的一段代码，它包含了java.lang.Float.parseFloat的正确用法。作者给出了两种变异方法，其中Mutant#2代表着一个API误用，因为这个API可能会抛出异常，该mutant删除了try-catch模块，从而违反了该API的正确用法，就是缺少了异常处理部分。

![API_Misuse_pic6](C:\Study\Paper\Images\API_Misuse_pic6.png)

根据API误用模式的特征，作者设计了8种mutate操作，包括：调换API调用序列，增加API调用、删除API调用等。在得到mutants后，使用项目的测试用例对mutants进行测试，并设计了metric对执行信息进行分析，按照分数对可能是API误用的mutant进行排序。

由于mutation操作是人工设计出来的，所以可以从mutant中推出是怎么产生该API误用的，也就是API使用的错误模式。作者给出了一些排名较高的API误用模式，利用这些API误用模式就可以检测API的使用是否存在这些情况。

![API_Misuse_pic7](C:\Study\Paper\Images\API_Misuse_pic7.png)

**结果**：作者使用该方法在16个项目的73种流行的java API上进行了实验，结果表明MUTAPI发现API误用模式的准确率高达0.78，在MUBENCH数据集上取得了0.49的召回率，超过了最先进的技术。后续作者计划在更少使用的API上使用该方法，调查MUTAPI是否可以检测出未被发现的API误用模式。



(1)(2)(4)使用**基于规则的方法**来检测API误用缺陷的工作

(3)使用**概率模型**来检测API误用缺陷的工作



(1)(3)(4)基于**开源项目**进行研究

(2)从**API文档**中推断正确用法



























