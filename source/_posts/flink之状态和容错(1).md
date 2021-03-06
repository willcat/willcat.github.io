---
title: flink之状态和容错(1)Working with `state`
date: 2018-08-21 15:26:40
tags: [flink,流式处理]
categories: [大数据]
---
# 概览
有状态函数和算子(`operator`)将数据存储在各个元素/事件的处理中，使状态成为任何类型的更精细操作的关键构建块。
比如：
- 当应用程序搜索某些事件模式时，状态将存储到目前为止遇到的事件序列
- 在聚合每分钟/小时/天事件时，状态保留待处理的聚合
- 当在数据点流上训练机器学习模型时，状态保持模型参数的当前版本
- 当需要管理历史数据时，状态允许有效访问过去发生的事件

Flink需要了解状态，以便使用检查点使状态容错，并允许流应用程序的保存点。

有关状态的知识还允许重新调整Flink应用程序，这意味着Flink负责跨并行实例重新分发状态。

Flink的可查询状态功能允许你在运行时从Flink外部访问状态。

在使用state时，阅读Flink的state backend（状态后端）可能也很有用。Flink提供了不同的状态后端，用于指定状态的存储方式和位置。State可以位于Java的堆上或堆外。根据您的状态后端，Flink还可以管理应用程序的状态，这意味着Flink处理内存管理（如果需要可能会溢出到磁盘）以允许应用程序保持非常大的状态。你可以在不更改应用程序逻辑的情况下配置状态后端。

# Working with `state`
Flink中有两种基本状态:`Keyed State`和`Operator State`.
## `Keyed State` 和 `operator state`
### `Keyed State`
*`Keyed Steate`* 永远跟`key`有关，只能作用到`keyed stream`上。

你可以把`Keyed Stream`理解为被分区或者共享的，每个key只有一个状态-分区的`Operator State`.每个`keyed-state`逻辑上被绑定到一个唯一的`<parallel-instance,key>`的组合上，并且因为每个`key`属于一个确切的`keyed-operator`的`并行实例(`parallel-instance`)，所以我们可以将其简单的成为`<operator,key>`。

`keyed-state`进一步被组织为所谓的`Key Groups`.`Key Groups`是Flink重新分发`Keyed State`的原子单位(`atomic unit`)；`Key Groups`数量与定义的最大并行度相等。在运行期间，每个`keyed operator`的并行实例(`parallel instance`)处理一个或者多个`key group`的`keys`。

### `operator state`
对`Operator State`(`non-keyed state`)来说，每个`operator state`被绑定到一个并行实例上。`Kafka Connector`是在Flink中使用`Operator State`的一个很好的示例。每个`Kafka`消费者的并行实例维护一个`topic`分区和偏移量(`offset`)的映射(`map`)作为它的`operator state`

当并行度改变时，`Operator State`接口支持在并行的`operator`实例间重新分发状态。可以有不同的方案来进行这种重新分发。

## 原始状态和托管状态(`Raw` and `Managed` `State`)
`Keyed State`和`Operator State`以两种形式存在:`managed`和`raw`。

托管状态(`Managed State`):由Flink运行时控制的数据结构表示，例如内部哈希表或`RocksDB`。托管状态的例子有`ValueState`，`ListState`等。Flink的运行时对这些状态进行编码并将它们写入检查的(`checkpoint`)。

原始状态(`Raw State`): `operator`保存在自己数据结构中的状态。当执行写入检查点操作时(`checkpointed`)，它们只会将一个字节序列写到到检查点(`checkpoint`)。Flink对状态的数据结构一无所知，只能看到原始字节。

所有的数据流都可以使用托管状态，但是原始状态只能在`operator`上使用。我们推荐使用托管状态(而不是原始状态)，因为在使用托管状态时，Flink可以在并行度改变后自动重新分发状态，并且能够更好的进行内存管理。

**注意** 如果你需要对托管状态的序列化逻辑进行自定义，务必参考[这里
](https://ci.apache.org/projects/flink/flink-docs-release-1.6/dev/stream/state/custom_serialization.html)来保证对未来的兼容性。Flink默认的序列化器不需要特殊处理。

## 使用托管键控状态(`managed keyed state`)
托管键控状态(`managed keyed state`)接口提供了对不同状态类型的访问，这些类型的作为方位都被限定在当前输入元素的键(`key`)上。这意味着这个类型的状态只能被用在键流(`keyed stream`)上，这些流可以由`stream.keyBy(...)`来创建。
支持的状态有:
- `ValueState<T>`
- `ListState<T>`
- `ReducingState<T>`
- `AggregatingState<IN, OUT>`
- `MapState<UK, UV>`

**注意**
1. 这些状态对象仅用于对接状态。状态不一定驻留在内部但是有可能存储在磁盘或其他位置。
2. 你从状态中获取的值取决于输入元素所属的`key`

### 如何使用这些状态
如果想获取状态的句柄，需要创建一个`StateDesciptor`。它保存了状态的名称（我们稍后会看到，您可以创建多个状态，并且它们必须具有唯一的名称以便您可以引用它们），以及状态所持有的值的类型，还可能有一个用户自定义的函数，如`ReduceFunction`。根据想要检索的状态类型，创建`ValueStateDescriptor`,`ListStateDescriptor`,`ReducingStateDescriptor`或者`MapStateDescriptor`.

因为`State`通过`RuntimeContext`来访问，所以只可能在`rich function`里使用。可以查看[这里](https://ci.apache.org/projects/flink/flink-docs-release-1.6/dev/api_concepts.html#rich-functions)(简而言之就是`rich function`可以为我们提供除了普通function以外，还有:`open`, `close`, `getRuntimeContext`,和 `setRuntimeContext`四个函数，赋予我们做更多事情的能力)。在`Rich fucntion`里的`RuntimeContext`有以下访问对象的方法:
- `ValueState<T> getState(ValueStateDescriptor<T>)`
- `ReducingState<T> getReducingState(ReducingStateDescriptor<T>)`
- `ListState<T> getListState(ListStateDescriptor<T>)`
- `AggregatingState<IN, OUT> getAggregatingState(AggregatingState<IN, OUT>)`
- `FoldingState<T, ACC> getFoldingState(FoldingStateDescriptor<T, ACC>)`
- `MapState<UK, UV> getMapState(MapStateDescriptor<UK, UV>)`

下面是`FlatMapFunction`的一个例子
```java
public class CountWindowAverage extends RichFlatMapFunction<Tuple2<Long, Long>, Tuple2<Long, Long>> {

    /**
     * The ValueState handle. The first field is the count, the second field a running sum.
     */
    private transient ValueState<Tuple2<Long, Long>> sum;

    @Override
    public void flatMap(Tuple2<Long, Long> input, Collector<Tuple2<Long, Long>> out) throws Exception {

        // access the state value
        Tuple2<Long, Long> currentSum = sum.value();

        // update the count
        currentSum.f0 += 1;

        // add the second field of the input value
        currentSum.f1 += input.f1;

        // update the state
        sum.update(currentSum);

        // if the count reaches 2, emit the average and clear the state
        if (currentSum.f0 >= 2) {
            out.collect(new Tuple2<>(input.f0, currentSum.f1 / currentSum.f0));
            sum.clear();
        }
    }

    @Override
    public void open(Configuration config) {
        ValueStateDescriptor<Tuple2<Long, Long>> descriptor =
                new ValueStateDescriptor<>(
                        "average", // the state name
                        TypeInformation.of(new TypeHint<Tuple2<Long, Long>>() {}), // type information
                        Tuple2.of(0L, 0L)); // default value of the state, if nothing was set
        sum = getRuntimeContext().getState(descriptor);
    }
}

// this can be used in a streaming program like this (assuming we have a StreamExecutionEnvironment env)
env.fromElements(Tuple2.of(1L, 3L), Tuple2.of(1L, 5L), Tuple2.of(1L, 7L), Tuple2.of(1L, 4L), Tuple2.of(1L, 2L))
        .keyBy(0)
        .flatMap(new CountWindowAverage())
        .print();

// the printed output will be (1,4) and (1,5)
```

### 状态的过期时间(`Time-to-Live TTL`)
`time-to-live`可以在任意类型的`keyed state`上设置。如果一个状态设置了`TTL`，并且已经过期，那么被保存的值就会被清理。

所有的状态集合类型都支持`per-entry` TTL,也就是说`list`元素和`map` `entries`各自独立过期。

想要使用TTL,你需要首先构建一个`StateTtlConfig`配置对象。然后，可以通过传递配置在任何状态描述符中启用TTL功能。
例子：
```java
import org.apache.flink.api.common.state.StateTtlConfig;
import org.apache.flink.api.common.state.ValueStateDescriptor;
import org.apache.flink.api.common.time.Time;

StateTtlConfig ttlConfig = StateTtlConfig
    .newBuilder(Time.seconds(1))
    .setUpdateType(StateTtlConfig.UpdateType.OnCreateAndWrite)
    .setStateVisibility(StateTtlConfig.StateVisibility.NeverReturnExpired)
    .build();

ValueStateDescriptor<String> stateDescriptor = new ValueStateDescriptor<>("text state", String.class);
stateDescriptor.enableTimeToLive(ttlConfig);
```
这里的配置有基础需要注意的:
- `newBuilder`是强制的(mandatory)，设置的是TTL值
- 更新类型配置状态TTL什么时候更新(默认是`OnCreateAndWrite`)
  - `StateTtlConfig.UpdateType.OnCreateAndWrite` 只在创建和写的时候更新
  - `StateTtlConfig.UpdateType.OnReadAndWrite` 读的时候也会更新
- 状态可见性配置过期但仍未被清除(clean up)的值在被访问时是否返回(默认是`NeverReturnExpired`)
  - `StateTtlConfig.StateVisibility.NeverReturnExpired` 过期值永远不会被返回
  - `StateTtlConfig.StateVisibility.ReturnExpiredIfNotCleanedUp` 如果还可用就返回

**注意**
  -  `state backend`会存储用户值最后一次修改的时间戳和值，这意味着开启这个特性会增加状态存储的消耗。`Heap state backend`在内存中存储一个额外的Java对象，其中包括一个用户状态对象的引用和一个初始(`primitive`)长整型。`RocksDB state backend`为每个存储值，列表条目(list entry)或(map entry)添加8个字节
  - 目前仅支持`processing time`的TTL。
  - 尝试使用开启了TTL的descriptor恢复状态或者相反，会触发兼容性错误和`StateMigrationException`
  - TTL不是检查点(`checkpoint`)或者保存点(`savepoint`)的一部分，而是Flink如何在当前运行的作业中对待它的一种方式。

#### 清理过期状态(Clieanup of Expired State)
目前，只有在状态被显式读取的时候才会被清除，比如调用`ValueState.getValue()`
**注意** 这意味着默认情况下，未被读取的过期状态是不会被清除的，可能会导致状态不断增长。这可能在将来的版本中改变。

另外，你可以在获取完整状态快照的时候激活清理，这会减小状态的大小。当前实现下不会清除本地状态，但是从上一个快照恢复的情况下，它不会包括被清理的过期状态。它可以在`StateTtlConfig`中配置：
```java
import org.apache.flink.api.common.state.StateTtlConfig;
import org.apache.flink.api.common.time.Time;

StateTtlConfig ttlConfig = StateTtlConfig
    .newBuilder(Time.seconds(1))
    .cleanupFullSnapshot()
    .build();
```
*这个选项不适合在RocksDB state backend下的增量checkpointing*
*未来会添加更多策略，以便在后台自动清理过期状态*

#### State in the Scala DataStream API
[here](https://ci.apache.org/projects/flink/flink-docs-release-1.6/dev/stream/state/state.html#state-in-the-scala-datastream-api)

## 使用托管算子状态(`Managed Operator State`)
要想使用`managed operator state`，有状态函数要么实现`CheckpointedFunction`接口，要么实现`ListCheckpointed<T extends Serializable>`接口。

### `CheckpointedFunction`接口
`CheckpointedFunction`函数提供对具有不同重新分发方案的非键控(non-keyed)状态的访问。它需要实现线面两个方法:
```java
void snapshotState(FunctionSnapshotContext context) throws Exception;

void initializeState(FunctionInitializationContext context) throws Exception;
```
当一个检查点需要被执行的时，`snapshotState()`会被调用。 每次用户自定义函数初始时会调用对应的`initializeState()`,即首次初始化函数时，或者当函数实际从早期检查点恢复时。鉴于此，`initializeState（）`不仅是初始化不同类型状态的地方，而且还是包括状态恢复逻辑的地方。

当前，Flink支持`list-style`类型的`managed operator state`，状态应该是可序列化对象的列表(`List`)，彼此独立，因此可以在调整大小时重新分发。换句话说，这些对象是可以重新分发非键控状态的最细粒度。根据状态访问方法，定义了以下重新分发方案:
- 均匀分裂再分配(`Even-split redistribution`) 每个`operator`返回一个状态元素的List,整个状态在逻辑上是所有列表的串联(concatenation)。在恢复(restore)/重新分发时，列表被分为与并行算子(`parallel operator`)数一样多的子列表。每个算子获得一个列表，这个列表可能为空，或者包含一个或更多列表。
- 联合再分配(`Union redistribution`)。 每个`operator`返回一个状态元素的List,整个状态在逻辑上是所有列表的串联(concatenation)。在恢复(restore)/重新分发时，每个算子会获得一个完整的状态元素的列表。

下面是一个例子，有状态函数`SinkFunction`使用`CheckpointedFunction`来缓冲在最终被发送到外界的元素。它演示了基本的均匀分布状态列表
```java
public class BufferingSink
        implements SinkFunction<Tuple2<String, Integer>>,
                   CheckpointedFunction {

    private final int threshold;

    private transient ListState<Tuple2<String, Integer>> checkpointedState;

    private List<Tuple2<String, Integer>> bufferedElements;

    public BufferingSink(int threshold) {
        this.threshold = threshold;
        this.bufferedElements = new ArrayList<>();
    }

    @Override
    public void invoke(Tuple2<String, Integer> value) throws Exception {
        bufferedElements.add(value);
        if (bufferedElements.size() == threshold) {
            for (Tuple2<String, Integer> element: bufferedElements) {
                // send it to the sink
            }
            bufferedElements.clear();
        }
    }

    @Override
    public void snapshotState(FunctionSnapshotContext context) throws Exception {
        checkpointedState.clear();
        for (Tuple2<String, Integer> element : bufferedElements) {
            checkpointedState.add(element);
        }
    }

    @Override
    public void initializeState(FunctionInitializationContext context) throws Exception {
        ListStateDescriptor<Tuple2<String, Integer>> descriptor =
            new ListStateDescriptor<>(
                "buffered-elements",
                TypeInformation.of(new TypeHint<Tuple2<String, Integer>>() {}));

        checkpointedState = context.getOperatorStateStore().getListState(descriptor);

        if (context.isRestored()) {
            for (Tuple2<String, Integer> element : checkpointedState.get()) {
                bufferedElements.add(element);
            }
        }
    }
}
```
`initializeState`方法将`FunctionInitializationContext`作为参数。这用于初始化非键控状态“容器”。这些是`ListState`类型的容器，其中非键控状态对象将在检查点存储。


状态访问方法的命名约定包含其重新分发模式，后跟其状态结构。例如，要在还原时使用联合重新分发方案的列表状态，请使用`getUnionListState(descriptor)`访问该状态。如果方法名称不包含重新分发模式，例如`getListState(descriptor)`，这意味着将使用基本的均匀分裂再分配(`Even-split redistribution`)方案。

在初始化容器之后，我们使用上下文的`isRestored()`方法来检查我们是否在失败后恢复。如果是，即我们正在恢复，则应用恢复逻辑。

如修改后的BufferingSink的代码所示，在状态初始化期间恢复的`ListState`保存在类变量`bufferedElements`中以供将来在`snapshotState()'中使用。在那里，ListState被清除了前一个检查点包含的所有对象，然后填充了我们想要检查点的新对象.

作为旁注，键控状态也可以在`initializeState()'方法中初始化。这可以使用提供的`FunctionInitializationContext`来完成

### `ListCheckpointed`接口
`ListCheckpointed`接口是`CheckpointedFunction`的一个更受限的变体，它仅支持在恢复时具有均匀分裂再分配(`Even-split redistribution`)方案的`list-style`的状态。它也要实现以下两个方法:
```java
List<T> snapshotState(long checkpointId, long timestamp) throws Exception;

void restoreState(List<T> state) throws Exception;
```
在`snapshotState()`里，算子(`operator`)应该将对象列表返回到检查点，并且`restoreState()`必须在恢复时处理这样的列表.如果状态不可重新分区，则始终可以在`snapshotState()`中返回`Collections.singletonList(MY_STATE)`.

### 有状态资源函数(stateful source function)

与其他`operator`相比，有状态的`source`需要更多的关注。为了使状态和输出集合的更新成为原子性的（在故障/恢复时`Exactly-once`语义所需），用户需要从`source`的上下文中获取锁(`lock`)
```java
public static class CounterSource
        extends RichParallelSourceFunction<Long>
        implements ListCheckpointed<Long> {

    /**  current offset for exactly once semantics */
    private Long offset;

    /** flag for job cancellation */
    private volatile boolean isRunning = true;

    @Override
    public void run(SourceContext<Long> ctx) {
        final Object lock = ctx.getCheckpointLock();

        while (isRunning) {
            // output and state update are atomic
            synchronized (lock) {
                ctx.collect(offset);
                offset += 1;
            }
        }
    }

    @Override
    public void cancel() {
        isRunning = false;
    }

    @Override
    public List<Long> snapshotState(long checkpointId, long checkpointTimestamp) {
        return Collections.singletonList(offset);
    }

    @Override
    public void restoreState(List<Long> state) {
        for (Long s : state)
            offset = s;
    }
}
```
当Flink完全确认检查点时，某些`operator`可能需要这些信息，以与外界通信。在这种情况下，请参阅`org.apache.flink.runtime.state.CheckpointListener`接口
