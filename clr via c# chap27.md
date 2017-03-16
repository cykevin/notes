# CH27 计算限制的异步操作


##　CLR线程池基础

每个CLR一个线程池，这个线程池由CLR包含的所有AppDomain共享

在内部，线程池维护了一个请求队列，当应用程序执行异步操作请求时，实际就是将一个记录项添加到队列。线程池会提取记录项，派发给选取一个可用的线程去执行操作，当没有可用的线程时就创建一个。

线程执行完毕后，并不立即销毁，而是进入线程池等待响应下一个请求。

线程池会尝试用一个线程去完成工作，如果异步请求的速度超过了线程处理的速度，线程池会创建线程，如果在较短的时间发送了大量的请求那么线程池会创建大量的线程。当这些线程完成操作后处于闲置状态时，他们会唤醒然后结束自己。

## 执行简单的计算限制操作

    public static boolean QueueUserWorkItem(WaitCallback callBack);
	public static boolean QueueUserWorkItem(WaitCallback callBack,Object state);

## 执行上下文

每个线程都包含了一个与之关联的数据结构（执行上下文）里面包含了一些信息：

* 安全设置
* 宿主设置
* 逻辑调用上下文数据

默认情况下，当执行线程开启一个辅助线程执行操作时，执行线程的执行上下文信息会流向辅助线程。这会对性能造成一定的影响。我们可以使用以下方法来阻止这种默认行为：

    ExecuteContext.SupressFlow();

使用以下方法恢复：

    ExecuteContext.RestoreFlow();

## 协作式取消和超时

.net framework提供了一种协作式取消模式，为长时计算限制操作提供取消功能

需要借助

    System.Threading.CancellationTokenSource

类来完成

先创建该类型的实例，将实例的Token属性传递到长时的计算限制操作中，在操作内部定时调用Token属性对象的IsCancellationRequested属性来查看是否发出了取消请求，从而终止循环操作。

还可以调用Register方法登记若干个回调方法，这些回调方法在操作被取消时执行。

## 任务

ThreadPool并没有提供内建的机制来通知什么时候操作完成或是获取计算的返回值。
为此Microsoft新提出“任务”的概念以解决上述两个问题（以及其他问题）

### 等待任务完成

可调用Task对象的Wait方法等待任务完成

可调用Task对象的Result属性获取任务结果（Result属性内部会调用Wait方法）

如果任务抛出了未处理的异常，调用Wait或者Result属性时会抛出异常

### 取消任务

可用CancellationTokenSource来取消Task，向任务传递CancellationToken，在任务中调用Token对象的ThrownIfCancellationRequested方法，在任务被取消时抛出异常。

### 任务完成时启动新任务

可以通过调用Task对象的ContinueWith方法在任务完成时开始一个新的任务。甚至还可以传递一些位标识指定新任务的运行条件：比如说在前置任务完成、取消或者出错后执行。

### 任务可以启动子任务

我们可以在Task内部再创建若干个Task。
> 理论上一个任务创建的Task默认是顶级任务，他们与创建他们的Task无关，但是我们可以通过在创建时传递AttachToParent标识来将他们关联到创建他们的Task

这样，他们就成了子任务，除非所有的子任务完成，否则父任务的状态都不是完成。

### 任务内部揭秘

每个任务都包含了一组字段：

* Task的Int32Id，表示执行状态
* 对父任务的引用
* 对TaskScheduler的引用
* 对回调方法的引用
* 对ExecutionContext的引用
* 对MaualResetEventSlim的引用
* 对CancellationToken的引用
* 对ContinueTask集合的引用

这些都会造成资源占用，必须为所有的这些分配内存，如果我们不需要Task带来的新功能，可以使用ThreadPool。

Task的状态会自动变化：

1. 创建后：Created
2. 任务启动：WaitingToRun
3. 在线程上运行时：Running
4. 任务停止等待子任务：WaitingForChildrenToComplete
5. 任务完成进入以下三种中的一种：
	1. RanToCompletion（运行完成）
	2. Canceled（任务被取消）
	3. Faulted（任务出错）

通过Continue*、FromAsync方法创建的Task处于WaitingForActivation状态，表明该Task的调度由任务基础结构控制，不可显式启动。

### 任务工厂

TaskFactory用于创建返回void的任务，TaskFactory<TResult>用于创建返回TResult类型的任务。

同一个任务工厂创建的任务具有相同的配置：

* CancellationToken
* TaskScheduler
* TaskCreationOptions
* TaskContinuationOptions

### 任务调度器

TaskScheduler是任务调度的核心，负责执行被调度任务。

FCL提供了两个派生的调度类：

* 线程池调度器（默认）

可通过TaskScheduler.Default访问，该调度器将任务调度到线程池线程。

* 同步上下文调度器

可通过TaskScheduler.FromCurrentSynchronizationContext()方法获取，该调度器将所有的任务都调度给应用程序的GUI线程。使得任务的代码能访问并更新UI

## Parallel的静态For、Foreach和Invoke方法

Parallel为一些常见的情景提供了封装。

* For
* Foreach

Parallel内部会启用Task完成循环中的任务，这些任务都是并行运行的。

## 并行语言集成查询

Linq提供了便捷的方法对集合进行排序、筛选、映射的操作，它们在一个线程内顺序执行

PLinq在内部使用任务将循环操作分配到多个CPU上以提高性能。

System.Linq.ParallelEnumerable类实现了PLinq的所有功能

要使用并行查询功能（以前基于IEnumerable或IEnumerbale<T>的查询称为顺序查询），必须将顺序查询转换为并行查询（基于ParallelEnumerable或ParallelEnuerable<T>）。

使用ParallelEnumerable提供的扩展方法AsParallel实现该功能。

## 执行定时的	计算限制操作

可以使用System.Threading.Timer来执行定时操作。

为了避免计算限制操作时间过长而导致在下一次timer运行时仍然没有返回，可以在回调方法使用Change方法以确保在回调方法完成后再执行下一次回调。

## 线程池如何管理线程

线程池使用全局队列，工作线程从队列中以FIFO的方式，使用同步锁获取任务。

获取后加载到工作线程的本地队列。

我们应该完全相信ThreadPool后台对工作线程的设定算法。而不应该显式去修改这些默认的设定。