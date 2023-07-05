如果你没学过 Rust 的话，别担心，这里会用最易懂的方式拆解 IC 源码。由于 IC 系统复杂，项目代码很多，这里只挑出一部分代码逻辑分析，以帮助大家接触学习 Rust ，了解 IC 。

https://github.com/dfinity/ic/tree/master/rs/messaging/src



首先，这个 Rust 代码片段的核心功能是实现一个消息路由模块，负责处理跨子网的消息传递。它主要涉及到的概念有：批次（Batch）、消息时间（MessageTime）、流时间线（StreamTimeline）、延迟度量（LatencyMetrics）等。



引入依赖

在代码开始部分，我们可以看到许多 `use` 语句，它们引入了本模块需要使用的其他模块的类型和函数。这有点像高中生学习物理时需要掌握的各种公式和概念。

```rust
use crate::{
    routing, scheduling,
    state_machine::{StateMachine, StateMachineImpl},
};
// ... 省略其他引入 ...
use std::sync::{Arc, Mutex, RwLock};
use std::thread::sleep;
use std::{
    collections::{BTreeMap, BTreeSet, VecDeque},
    time::Instant,
};
```



常量定义

接着是一些常量的定义，这些常量在后面的代码中会被用到。`BATCH_QUEUE_BUFFER_SIZE` 是批次队列的缓冲大小，表示允许的最大批次数。其他常量主要用于指标（Metrics）的名称和标签。

```rust
const BATCH_QUEUE_BUFFER_SIZE: usize = 16;

const METRIC_DELIVER_BATCH_COUNT: &str = "mr_deliver_batch_count";
const METRIC_EXPECTED_BATCH_HEIGHT: &str = "mr_expected_batch_height";
const METRIC_REGISTRY_VERSION: &str = "mr_registry_version";
pub(crate) const METRIC_TIME_IN_BACKLOG: &str = "mr_time_in_backlog";
pub(crate) const METRIC_TIME_IN_STREAM: &str = "mr_time_in_stream";

const LABEL_STATUS: &str = "status";
pub(crate) const LABEL_REMOTE: &str = "remote";

const STATUS_IGNORED: &str = "ignored";
const STATUS_QUEUE_FULL: &str = "queue_full";
const STATUS_SUCCESS: &str = "success";

const PHASE_LOAD_STATE: &str = "load_state";
const PHASE_COMMIT: &str = "commit";

const METRIC_PROCESS_BATCH_DURATION: &str = "mr_process_batch_duration_seconds";
const METRIC_PROCESS_BATCH_PHASE_DURATION: &str = "mr_process_batch_phase_duration_seconds";
const METRIC_TIMED_OUT_REQUESTS_TOTAL: &str = "mr_timed_out_requests_total";

const CRITICAL_ERROR_MISSING_SUBNET_SIZE: &str = "cycles_account_manager_missing_subnet_size_error";
const CRITICAL_ERROR_NO_CANISTER_ALLOCATION_RANGE: &str = "mr_empty_canister_allocation_range";
```



`MessageTime` 结构体

`MessageTime` 结构体记录了一个消息在流中的时间戳。它包含两个字段：`index` 是流索引，`time` 是一个计时器（`Timer`）记录了消息进入流中的时间。

```rust
struct MessageTime {
    index: StreamIndex,
    time: Timer,
}

impl MessageTime {
    fn new(index: StreamIndex) -> Self {
        MessageTime {
            index,
            time: Timer::start(),
        }
    }
}
```



`StreamTimeline` 结构体

`StreamTimeline` 是一个时间线结构，用于记录流中的消息时间戳。它包含一个 `entries` 字段，用 `VecDeque` 保存消息时间（`MessageTime`）对象。`histogram` 字段是一个直方图（`Histogram`），用于记录消息在流中的持续时间。

```rust
struct StreamTimeline {
    entries: VecDeque<MessageTime>,
    histogram: Histogram,
}

impl StreamTimeline {
    fn new(histogram: Histogram) -> Self {
        StreamTimeline {
            entries: VecDeque::new(),
            histogram,
        }
    }

    fn add_entry(&mut self, index: StreamIndex) {
        // ...
    }

    fn observe(&mut self, index_range: Range<StreamIndex>) {
        // ...
    }
}
```



`LatencyMetrics` 结构体

`LatencyMetrics` 结构体用于记录跨子网的消息传递中的延迟度量。它包含一个 `timelines` 字段，是一个 `BTreeMap` 类型，存储远程子网 ID 和对应的 `StreamTimeline` 对象。`backlog_histogram` 字段是一个直方图（`Histogram`），用于记录消息在后台队列中的停留时间。

```rust
struct LatencyMetrics {
    timelines: BTreeMap<SubnetId, StreamTimeline>,
    backlog_histogram: Histogram,
}

impl LatencyMetrics {
    fn new(metrics_registry: &MetricsRegistry) -> Self {
        // ...
    }

    fn add_entry(&mut self, remote: SubnetId, index: StreamIndex) {
        // ...
    }

    fn observe(&mut self, remote: SubnetId, index_range: Range<StreamIndex>) {
        // ...
    }

    fn observe_backlog_latency(&mut self, index_range: Range<StreamIndex>, backlog_start: Instant) {
        // ...
    }
}
```



`MessageRoutingImpl` 结构体

`MessageRoutingImpl` 结构体是整个模块的核心，负责实现 `MessageRouting` trait。它包含了一个状态机（`StateMachine`）对象、一个批处理队列（`batch_queue`）、一个共享的延迟度量对象（`LatencyMetrics`）以及一些其他成员。

```rust
pub(crate) struct MessageRoutingImpl {
    state_machine: Arc<StateMachine>,
    batch_queue: Arc<Mutex<VecDeque<Batch>>>,
    latency_metrics: Arc<RwLock<LatencyMetrics>>,
    registry_version: Arc<AtomicU64>,
    expected_batch_height: Arc<AtomicU64>,
    deliver_batch_count: IntCounterVec,
    process_batch_duration: HistogramVec,
    process_batch_phase_duration: HistogramVec,
    timed_out_requests_total: IntCounterVec,
}
```



`MessageRouting` trait 的实现

在 `MessageRoutingImpl` 结构体的基础上，实现了 `MessageRouting` trait。主要方法包括：`enqueue_batch`、`process_batch` 和 `commit`。这些方法负责处理批次、更新状态和提交消息。

```rust
impl MessageRouting for MessageRoutingImpl {
    fn enqueue_batch(&self, batch: Batch) -> Result<(), BatchEnqueueError> {
        // ...
    }

    fn process_batch(&self, batch: &Batch) -> Result<(), BatchProcessingError> {
        // ...
    }

    fn commit(&self, height: BatchHeight) -> Result<(), BatchCommitError> {
        // ...
    }
}
```

通过以上分析，我们可以了解到这段 Rust 代码实现了一个跨子网消息路由模块。它包括批次处理、消息时间线、延迟度量等核心功能。这些功能通过一系列的结构体和方法实现，为外部提供消息处理和状态更新的接口。







这段代码的核心功能是记录流中消息的延迟。

在进行详细分析之前，让我们先看看代码中的关键部分。代码中有三个结构体：`MessageTime`、`StreamTimeline` 和 `LatencyMetrics`。结构体就像是一种蓝图，用来描述我们关心的事物。此外，代码还定义了一些常量，我们可以把它们看作是标签，用来帮助我们理解和分类数据。

1. **常量定义：**

   这部分定义了一些用于批处理、度量和状态的常量。我们可以把这些常量看作是标签，用来帮助我们理解和分类数据。

2. **MessageTime 结构体：**

   这个结构体就像是一个记录仪，用来记录一条消息在流中的位置和时间。它有两个部分：`index`（位置）和 `time`（时间）。它还有一个功能，就是创建一个新的 `MessageTime` 实例，就像制造一个新的记录仪。

3. **StreamTimeline 结构体：**

   这个结构体就像是一张地图，记录了流中消息的时间戳。它有两个部分：一个用来存储 `MessageTime` 记录仪的列表（`entries`）和一个用来记录消息在流中停留时间的直方图（`histogram`）。

   这个地图有三个功能：

   - 创建一张新的地图（`new` 方法）；
   - 添加一个记录仪到地图上（`add_entry` 方法）；
   - 观察地图上某个范围内的记录仪（`observe` 方法）。

4. **LatencyMetrics 结构体：**

   这个结构体是一个大本营，它包含了一组用于记录消息延迟的地图和直方图。它有两个部分：一个用来存储子网 ID 到地图的映射的字典（`timelines`）和一个用来存储每个子网的消息持续时间直方图的列表（`histograms`）。

   这个大本营有很多功能：

   - 创建一个新的大本营（`new` 方法）；
   - 创建一个用于记录特定观察值的大本营（`new_time_in_stream` 和 `new_time_in_backlog` 方法）；
   - 在给定子网的地图上执行某个功能（`with_timeline` 方法）；
   - 在地图上为给定子网的消息添加一个记录仪（`record_header` 方法）；
   - 观察给定子网中某个范围内的所有消息的持续时间（`observe_message_durations` 方法）。

它包含了一些结构体（蓝图）和方法（功能），用来描述我们关心的数据并执行一些操作。希望这个通俗易懂的解释能帮助你更好地理解这段代码。对于 Rust 初学者来说，了解这些代码可以帮助你学习如何使用 Rust 中的数据结构和功能。





---





这段代码是用 Rust 编写的，它主要包括两个结构体（`MessageRoutingMetrics` 和 `MessageRoutingImpl`）以及一个 trait（`BatchProcessor`）。整个代码片段的核心功能是对消息路由的度量和批处理进行监控和处理。现在我将为 Rust 初学者逐一分析这些代码片段。

首先，我们来看 `MessageRoutingMetrics` 结构体：

```rust
pub(crate) struct MessageRoutingMetrics {
    // ...
}
```

`MessageRoutingMetrics` 结构体主要用于存储一些度量指标，例如批处理的调用次数、预期批处理高度、处理批处理所需的时间等。这些指标可以用于监控和优化系统性能。

接下来，我们看 `MessageRoutingMetrics` 的实现：

```rust
impl MessageRoutingMetrics {
    pub(crate) fn new(metrics_registry: &MetricsRegistry) -> Self {
        // ...
    }

    pub fn observe_no_canister_allocation_range(&self, log: &ReplicaLogger, message: String) {
        // ...
    }
}
```

`MessageRoutingMetrics` 有两个方法。`new` 方法用于创建一个新的 `MessageRoutingMetrics` 实例，它需要一个 `MetricsRegistry` 参数来初始化各种度量指标。`observe_no_canister_allocation_range` 方法则用于记录一个关键错误，当没有可用的 canister 分配范围时调用。

现在我们来看 `MessageRoutingImpl` 结构体：

```rust
pub struct MessageRoutingImpl {
    // ...
}
```

`MessageRoutingImpl` 结构体主要用于实现消息路由功能。它包含一些字段，例如用于发送批处理的 `batch_sender`，状态管理器 `state_manager`，度量指标 `metrics`，日志记录器 `log` 等。这些字段协同工作，实现消息的路由和处理。

接下来，我们来看 `BatchProcessor` trait：

```rust
trait BatchProcessor: Send {
    fn process_batch(&self, batch: Batch);
}
```

`BatchProcessor` trait 是一个接口，表示可以处理批次的组件。它有一个方法 `process_batch`，用于处理给定的批次。

最后，我们来看 `BatchProcessorImpl` 结构体：

```rust
struct BatchProcessorImpl {
    // ...
}
```

`BatchProcessorImpl` 结构体是 `BatchProcessor` trait 的一个具体实现。它包含一些字段，例如状态管理器 `state_manager`，状态机 `state_machine`，注册表客户端 `registry` 等。这些字段协同工作，实现批处理的执行和提交。

总之，这段代码主要用于处理和监控消息路由的批处理。它包括两个结构体（`MessageRoutingMetrics` 和 `MessageRoutingImpl`）以及一个 trait（`BatchProcessor`）。每个组件都有其自己的职责和功能，通过协作来实现代码的核心功能。





这段代码是一个名为 `BatchProcessorImpl` 的结构体的实现（implementation）。`BatchProcessorImpl` 的主要功能是与注册表（Registry）进行交互，从中获取并处理网络拓扑信息、各类配置和记录。下面我们将逐个分析这个实现中的函数。

首先是 `new` 函数，它是一个构造函数，用于生成一个新的 `BatchProcessorImpl` 结构体实例。

```rust
fn new(
    state_manager: Arc<dyn StateManager<State = ReplicatedState>>,
    state_machine: Box<dyn StateMachine>,
    registry: Arc<dyn RegistryClient>,
    bitcoin_config: BitcoinConfig,
    metrics: Arc<MessageRoutingMetrics>,
    log: ReplicaLogger,
    malicious_flags: MaliciousFlags,
) -> Self {
    Self {
        state_manager,
        state_machine,
        registry,
        bitcoin_config,
        metrics,
        log,
        malicious_flags,
    }
}
```

`new` 函数接收一系列必要的参数，如状态管理器、状态机、注册表客户端等，并将它们存储在结构体的字段中。

接下来是 `observe_phase_duration` 函数，用于记录给定阶段的持续时间。

```rust
fn observe_phase_duration(&self, phase: &str, timer: &Timer) {
    self.metrics
        .process_batch_phase_duration
        .with_label_values(&[phase])
        .observe(timer.elapsed());
}
```

这个函数接收阶段名称和计时器作为参数，使用它们更新度量（metrics）中的 `process_batch_phase_duration` 数据。

接下来是 `observe_canisters_memory_usage` 函数，用于观察所有 Canister 的内存使用情况。

```rust
fn observe_canisters_memory_usage(&self, state: &ReplicatedState) {
    let mut memory_usage = NumBytes::from(0);
    for canister in state.canister_states.values() {
        memory_usage += canister.memory_usage(state.metadata.own_subnet_type);
    }
    self.metrics
        .canisters_memory_usage_bytes
        .set(memory_usage.get() as i64);
}
```

这个函数计算所有 Canister 的内存使用量并更新度量（metrics）。

接下来是 `get_provisional_whitelist` 函数，用于从注册表获取临时白名单。

```rust
fn get_provisional_whitelist(&self, registry_version: RegistryVersion) -> ProvisionalWhitelist {
    let provisional_whitelist = loop {
        match self.registry.get_provisional_whitelist(registry_version) {
            Ok(record) => break record,
            Err(err) => {
                warn!(
                    self.log,
                    "Unable to read the provisional whitelist: {}. Trying again...",
                    err.to_string(),
                );
            }
        }
        sleep(std::time::Duration::from_millis(100));
    };
    provisional_whitelist.unwrap_or_else(|| ProvisionalWhitelist::Set(BTreeSet::new()))
}
```

这个函数会一直尝试从注册表获取临时白名单，直到成功为止。如果获取失败，它会等待一段时间（100 毫秒）后重试。

接下来是 `populate_network_topology` 函数，用于在指定的注册表版本中填充网络拓扑信息。

```rust
fn populate_network_topology(&self, registry_version: RegistryVersion) -> NetworkTopology {
    loop {
        match self.try_to_populate_network_topology(registry_version) {
            Ok(network_topology) => break network_topology,
            Err(err) => {
                warn!(
                    self.log,
                    "Unable to populate network topology: {}. Trying again...",
                    err.to_string(),
                );
            }
        }
        sleep(std::time::Duration::from_millis(100));
    }
}
```

这个函数与 `get_provisional_whitelist` 类似，会一直尝试填充网络拓扑，直到成功为止。

接下来是 `try_to_populate_network_topology` 函数，它尝试从注册表填充网络拓扑信息。

```rust
fn try_to_populate_network_topology(
    &self,
    registry_version: RegistryVersion,
) -> Result<NetworkTopology, RegistryClientError> {
    // ...
}
```

这个函数包含很多逻辑，它会获取子网列表、节点列表等，然后逐个处理这些信息，构建一个完整的 `NetworkTopology` 结构体。如果发生错误，它会返回一个 `RegistryClientError` 类型的错误。

最后是 `process_batch` 函数，这是 `BatchProcessorImpl` 结构体的核心函数，用于处理一个批次（batch）。

```rust
fn process_batch(
    &self,
    mut batch: Batch,
    subnet_call_context_manager: &SubnetCallContextManager,
) -> Result<(), ProcessBatchError> {
    // ...
}
```

`process_batch` 函数接收一个 `Batch` 变量和一个 `SubnetCallContextManager` 变量作为参数。函数的主要逻辑如下：

1. 检查批次中的消息是否有效。
2. 检查批次中的消息是否满足运行时策略。
3. 按照预定义的策略处理批次中的消息。
4. 使用状态机处理批次中的消息。
5. 更新度量（metrics）和日志。
6. 返回处理结果。

整个实现中，`BatchProcessorImpl` 结构体主要负责与注册表交互，获取并处理网络拓扑信息、各类配置和记录。通过这些功能，它可以处理一个批次的消息，并更新相关度量和日志。





这段代码主要包含两个 Rust 结构体的实现：`BatchProcessorImpl` 和 `FakeBatchProcessorImpl`。它们都实现了 `BatchProcessor` trait，负责处理一批交易，并更新整个系统的状态。这两个结构体的实现有些许不同，`BatchProcessorImpl` 是正常的处理逻辑，而 `FakeBatchProcessorImpl` 主要用于测试和模拟场景。

我们将分几部分对代码进行分析：

**1. get_subnet_public_key 函数**

```rust
fn get_subnet_public_key(
    registry: Arc<dyn RegistryClient>,
    subnet_id: SubnetId,
    registry_version: RegistryVersion,
) -> Result<Vec<u8>, RegistryClientError> { ... }
```

这个函数的作用是从注册表中获取子网的公钥。它接收三个参数：注册表客户端、子网 ID 和注册表版本。函数返回一个 `Result` 类型，包含公钥的字节表示或一个错误。

**2. BatchProcessorImpl 结构体**

`BatchProcessorImpl` 结构体实现了 `BatchProcessor` trait，主要包含一个方法：`process_batch`。

```rust
impl BatchProcessor for BatchProcessorImpl {
    fn process_batch(&self, batch: Batch) { ... }
}
```

`process_batch` 方法负责处理一批交易，包括获取状态、执行交易、更新状态并提交。这个方法的流程如下：

1. 从 `StateManager` 获取上一个批次的状态。
2. 根据批次是否需要完整状态哈希来设置认证范围。
3. 根据注册表版本获取网络拓扑、临时白名单、子网特性等信息。
4. 使用 `state_machine` 执行交易。
5. 如果需要完整状态哈希，则在提交前进行垃圾回收。
6. 观察处理后的内存使用情况。
7. 将状态提交给 `StateManager`。

**3. FakeBatchProcessorImpl 结构体**

`FakeBatchProcessorImpl` 结构体同样实现了 `BatchProcessor` trait，主要用于测试和模拟场景。它的 `process_batch` 方法与 `BatchProcessorImpl` 类似，但有一些简化。

```rust
impl BatchProcessor for FakeBatchProcessorImpl {
    fn process_batch(&self, batch: Batch) { ... }
}
```

`process_batch` 方法的流程如下：

1. 从 `StateManager` 获取上一个批次的状态。
2. 设置批次时间。
3. 获取批次中的入口消息，并将它们视为已执行。
4. 更新入口消息的状态。
5. 修剪入口消息历史。
6. 使用 `stream_builder` 合并数据流。
7. 根据批次是否需要完整状态哈希来设置认证范围。
8. 将状态提交给 `StateManager`。

总之，这段代码主要处理批次交易的逻辑。`BatchProcessorImpl` 是正常的处理逻辑，而 `FakeBatchProcessorImpl` 主要用于测试和模拟场景。这两个结构体都实现了 `BatchProcessor` trait，负责执行交易并更新整个系统的状态。







这个代码片段实现了一个名为 `MessageRoutingImpl` 的结构体及其相关方法。整个代码片段的核心功能是实现消息路由，它负责处理、调度和分发消息批次。让我们逐步分析每个部分。

首先，我们先来看 `MessageRoutingImpl` 结构体的实现：

```rust
impl MessageRoutingImpl {
    ...
}
```

这里实现了一些方法，包括 `from_batch_processor`、`new`、`new_fake` 和 `inc_deliver_batch`。我们将分别解析这些方法。

1. `from_batch_processor` 方法：

```rust
fn from_batch_processor(
    state_manager: Arc<dyn StateManager<State = ReplicatedState>>,
    batch_processor: Box<dyn BatchProcessor>,
    metrics: Arc<MessageRoutingMetrics>,
    log: ReplicaLogger,
) -> Self {
    ...
}
```

这个方法是一个构造函数，它接受一个状态管理器、一个批处理器、一些度量（指标）和一个日志记录器作为参数。它会创建一个新的 `MessageRoutingImpl` 实例，然后启动一个批处理线程来处理批次。当该线程收到新的批次时，它会调用批处理器的 `process_batch` 方法来处理批次。

2. `new` 方法：

```rust
pub fn new(
    state_manager: Arc<dyn StateManager<State = ReplicatedState>>,
    certified_stream_store: Arc<dyn CertifiedStreamStore>,
    ingress_history_writer: Arc<dyn IngressHistoryWriter<State = ReplicatedState> + 'static>,
    scheduler: Box<dyn Scheduler<State = ReplicatedState>>,
    hypervisor_config: HypervisorConfig,
    cycles_account_manager: Arc<CyclesAccountManager>,
    subnet_id: SubnetId,
    metrics_registry: &MetricsRegistry,
    log: ReplicaLogger,
    registry: Arc<dyn RegistryClient>,
    malicious_flags: MaliciousFlags,
) -> Self {
    ...
}
```

`new` 是另一个构造函数，它创建一个新的 `MessageRoutingImpl` 实例。这个方法接收许多参数，如状态管理器、认证流存储、Ingress 历史记录写入器、调度器等。这些参数用于初始化各种组件，如流处理器、验证集规则实现、解复用器、流构建器等。最后，它调用 `from_batch_processor` 方法创建一个新的 `MessageRoutingImpl` 实例。

3. `new_fake` 方法：

```rust
pub fn new_fake(
    subnet_id: SubnetId,
    state_manager: Arc<dyn StateManager<State = ReplicatedState>>,
    ingress_history_writer: Arc<dyn IngressHistoryWriter<State = ReplicatedState> + 'static>,
    metrics_registry: &MetricsRegistry,
    log: ReplicaLogger,
) -> Self {
    ...
}
```

`new_fake` 方法是一个创建假的（用于测试目的）`MessageRoutingImpl` 实例的构造函数。它创建一个带有假批处理器的 `MessageRoutingImpl` 实例，并调用 `from_batch_processor` 方法。

4. `inc_deliver_batch` 方法：

```rust
fn inc_deliver_batch(&self, status: &str) {
    self.metrics
        .deliver_batch_count
        .with_label_values(&[status])
        .inc();
}
```

这个方法是一个辅助方法，用于增加具有给定状态的 `deliver_batch_count` 指标。

接下来，我们来看 `MessageRouting` trait 的实现：

```rust
impl MessageRouting for MessageRoutingImpl {
    ...
}
```

这里实现了两个方法：`deliver_batch` 和 `expected_batch_height`。我们将分别解析这些方法。

1. `deliver_batch` 方法：

```rust
fn deliver_batch(&self, batch: Batch) -> Result<(), MessageRoutingError> {
    ...
}
```

这个方法用于处理传入的批次。它首先检查批次号是否与预期的批次号相等。如果批次号不匹配，它将忽略该批次并返回一个错误。然后，它尝试将批次发送到批处理器。如果发送成功，它会更新 `last_seen_batch` 的值。如果队列已满，它将返回一个错误，表示队列容量已达到上限。

2. `expected_batch_height` 方法：

```rust
fn expected_batch_height(&self) -> Height {
    ...
}
```

这个方法返回下一个预期批次的高度。这个高度由 `last_seen_batch` 变量加上 1 计算得出。

总之，这个 Rust 代码片段实现了一个名为 `MessageRoutingImpl` 的结构体，它负责处理、调度和分发消息批次。它通过运行一个批处理线程，使用批处理器处理传入的批次，同时使用度量（指标）和日志记录器来跟踪处理过程。`MessageRoutingImpl` 结构体还实现了 `MessageRouting` trait，它定义了两个方法 `deliver_batch` 和 `expected_batch_height`，分别用于处理批次和获取下一个预期批次的高度。