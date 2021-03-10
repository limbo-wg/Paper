## A Systematic Evaluation of Static API-Misuse Detectors

#### API Misuse Classification

![MUC_MUBENCH](..\Images\MUC_MUBENCH.png)

1. Method Call (方法调用)

    * missing method call: API调用约束要求调用但是没有调用
    * redundant method call: 调用了API限制的方法
2. Condition (条件检查，包括null check, value or state, synchronization, context)
    * missing conditions: 约束条件没有确保满足
        * 调用方法或参数前没有检查是否为null
        * 使用map前没有检查Map中是否有key
        * 修改被其他线程获取的HashMap前没有lock
        * 没有确保在Event Dispatching Thread上更新SWING上的GUI组件
    * redundant conditions: a condition prevents a necessary part of a usage
        * 方法调用完之后才检查是否为null
        * 对一定不是空的集合进行isEmpty检查
        * 对已经持有的资源上锁，导致死锁
        * JUNIT断言运行在了其他线程，导致结果不能被JUNIT获取
3. Iteration (主要出现在集合与IO流中的循环和递归中)
    * missing iterations: 程序执行一段时间后没有重复检查指定的约束条件，例如循环等待条件中没有一直调用wait()
    * redundant iterations: 在只会执行一遍的代码中重复迭代，例如重复执行init()
4. Exception Handling (异常处理)
    * missing exception handling: 对可能出现异常的代码没有进行处理
    * redundant exception handling: 不应该执行catch的地方进行了catch处理





