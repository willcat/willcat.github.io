---
title: 'Effective Java 笔记8,避免使用Finalizer和Cleaner'
date: 2018-08-14 13:18:41
tags: [Java,编程语言,读书笔记,Effective Java]
categories: [编程语言]
---
### 1 被遗弃的`finalize()`
`finalize()`是`Object`的`protected`方法，子类可以覆盖该方法以实现资源清理工作，GC在回收对象之前调用该方法。但是在`Java9`中以及被标记为`deprecated`，取而代之的的是`Cleaner`对象。

**`finalizer`通常是不可预测，经常是危险的，降低性能且不必要的**，当然`Cleaner`虽然好一些，但是也是**不可预测地、慢地、通常非必要地**

### 2 避免使用`finalize()`或者`Cleaner`的理由
#### 2.1 执行时间没有保证
在对象变为不可达状态和`finalizer`或者`Cleaner`执行之间的时间有可能任意长。这意味着不要在`finalizer`或者`Cleaner`里执行任何时间敏感的操作，比如关闭文件。

`finalizer`或者`Cleaner`执行的迅速性与垃圾回收算法有关，依其实现而各异。

给一个类提供`finalizer`可能会任意久地延迟其实例的回收。`finalizer`的执行线程的优先级很低，最终可能会造成应用有上千个对象在它的`finalizer`队列里等待被回收。

不仅不能保证`finalizer`或者`Cleaner`及时执行，甚至根本不能保证其执行。因此，你**永远不要依赖一个`finalizer`或者`Cleaner`去更新持久化状态**。比如依赖`finalizer`或者`Cleaner`释放一个共享状态的持久锁(如数据库)是使整个分布式系统停止运行的好方法^-^~~

#### 2.2 `uncaught exception`在`finalization`中会被忽略
`finalizer`的另一个问题是会忽略在`finalization`中抛出的`uncaught exception`，然后终止该对象的`finalization`。未捕获异常可能导致其它对象处于损坏状态。如果另一个线程试图使用这个损坏的对象，可能会导致任意的不确定性行为。

`Cleaner`没有这个问题，因为一个使用cleaner的库能够控制它自己的线程。

####  2.3 使用`finalizer`或者`Cleaner`会严重影响性能
有时会比`try-with-resources`方式慢上几十倍。主要因为他们抑制(inhibit)垃圾回收的效率。当然，在你把`Cleaner`只当做`safety net`使用时它会快很多。

#### 2.4 安全问题
`finalizer`会把你的类置于`finalizer`攻击下。

`finalizer`攻击的原理很简单：如果有异常从构造函数或者它的序列化中抛出-如`readObject`和`readResolve`方法-恶意子类的`finalizer`可以运行在被应该死掉(died on the
vine)的、只完成部分构造的对象上。这个`finalizer`可以把对象的引用记录到一个`static`域上，从而阻止其被GC回收。然后缺陷对象的本不应该存在的方法就会被子类随意调用。

一个final类型的class能够免于出现上述情况，因为不能创建其子类。

##### 解决办法
在对象上写一个`final`的`finalize()`方法，这个方法什么也不做

### 3. `finalizer`或者`Cleaner`的替代方法
在不使用`finalizer`或者`Cleaner`时，一个类的对象封装了需要回收的资源如文件和或者线程时，我们应该怎样做呢?**让这个类实现`AutoCloseable`**,并且要求其的客户端(Client)在不再需时调用每个实例的`close()`方法。通常使用`try-with-resources`来保证终止，即使要面对异常。

有一个需要强调的点：*实例必须跟踪它是否被closed*: 一个对象失效时，`close()`方法必须记录到一个对象的属性中，其他方法必须检查这个属性，当对象关闭之后被调用时，这些方法应该抛出一个`IllegalStateException`


### 4.  `finalize()`和`Cleaner`的几个应用
#### 4.1 充当`safety net`的角色
当自由拥有者忘记调用`close()`方法时，充当保护网(`safety net`)的角色。虽然不能保证其运行时机，但是在client忘记时，迟些释放自由总比不释放要强。

当你要写这种`safety net`时要三思，考虑为这些保护付出的代价是否值得。

一些`Java`类比如`FileInputStream`,`FileOutputStream`,`ThreadPoolExecutor`,`java.sql.Connection`实现了这样的安全网。

#### 4.2 另一个合法应用
- `cleaners`还可用于有本地对等类(`native peer`)的情况。因为一个`native peer`不是一个普通的对象，垃圾回收器意识不到其存在，当它的`Java peer`被回收的时候并不能同时回收它。

- 当性能是可以接受，并且`native peer`持有的不是关键资源时，`finalizer`或者`Cleaner`是一种完成回收任务的一个合适选项。
- 当上述性能不能被接受，或者类的`native peer`持有的是关键资源时需要尽快被回收时，这个类还是应该实现`close()`方法
##### 4.2.1 `native peer`
`native peer`是普通对象通过本地方法委托的本地(非java)对象

### 结论
不要使用`cleaners`，或者在早于`Java9`的版本中使用`finalize()`，除非作为`safety net`或者终止不重要的本地资源。即使这样，依然要注意不确定性和性能影响。

### 参考
[Effective Java 第三版——8. 避免使用Finalizer和Cleaner机制](https://www.cnblogs.com/IcanFixIt/p/8133798.html)
