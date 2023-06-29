这段代码是一个用于管理和调度智能合约执行的 Rust 库。整个代码片段的核心功能是实现一个名为 `SchedulerImpl` 的结构体，该结构体负责管理执行环境、调度执行任务、处理执行结果和执行限制等。

**1. 代码片段和功能分析**

首先，代码引入了一系列需要的模块和库。这里不再一一列出。

然后，代码定义了两个常数：

```rust
const SPAMMY_LOG_INTERVAL_ROUNDS: u64 = 10 * 60;
const SUBNET_MESSAGES_LIMIT_FRACTION: u64 = 16;
```

`SPAMMY_LOG_INTERVAL_ROUNDS` 是用于限制日志记录频率的常数，以避免过于频繁的日志输出。`SUBNET_MESSAGES_LIMIT_FRACTION` 是用于限制子网消息的常数，以避免智能合约消息的数量超过限制。

接下来，代码定义了一些模块，用于测试和度量：

```rust
mod scheduler_metrics;
use scheduler_metrics::*;
mod round_schedule;
pub use round_schedule::RoundSchedule;
use round_schedule::*;
```

这部分代码引入了度量模块（`scheduler_metrics`）和调度模块（`round_schedule`）。

紧接着，代码定义了一个宏 `debug_assert_or_critical_error`，用于在调试时验证条件是否满足，同时记录错误日志：

```rust
macro_rules! debug_assert_or_critical_error {
    // ...
}
```

最后，代码定义了核心结构体 `SchedulerImpl`，它包含了一系列调度任务所需的成员变量和方法：

```rust
pub(crate) struct SchedulerImpl {
    config: SchedulerConfig,
    own_subnet_id: SubnetId,
    ingress_history_writer: Arc<dyn IngressHistoryWriter<State = ReplicatedState>>,
    exec_env: Arc<ExecutionEnvironment>,
    cycles_account_manager: Arc<CyclesAccountManager>,
    metrics: Arc<SchedulerMetrics>,
    log: ReplicaLogger,
    thread_pool: RefCell<scoped_threadpool::Pool>,
    rate_limiting_of_heap_delta: FlagStatus,
    rate_limiting_of_instructions: FlagStatus,
    deterministic_time_slicing: FlagStatus,
}
```

这个结构体的成员变量包括：

- config：用于保存调度器的配置。
- own_subnet_id：表示调度器所属的子网 ID。
- ingress_history_writer：用于写入执行历史的组件。
- exec_env：表示执行环境，用于执行智能合约。
- cycles_account_manager：管理服务费用的组件。
- metrics：度量组件，用于收集和报告度量数据。
- log：日志记录器，用于记录调度器的日志。
- thread_pool：线程池，用于执行任务。
- rate_limiting_of_heap_delta：标志，表示是否启用堆增量的速率限制。
- rate_limiting_of_instructions：标志，表示是否启用指令数的速率限制。
- deterministic_time_slicing：标志，表示是否启用确定性时间切片。

**2. 代码详细分析**

由于代码片段较长，这里仅对结构体 `SchedulerImpl` 进行详细分析。其他部分的代码，如宏和模块定义，可以参考上述分析。

结构体 `SchedulerImpl` 的成员变量表示了调度器所需的各种组件和状态。当创建一个新的 `SchedulerImpl` 实例时，需要为这些成员变量赋值。在调度器实际运行时，这些成员变量会被用于管理执行环境、调度执行任务、处理执行结果和执行限制等。





这段代码主要实现了一个名为 `SchedulerImpl` 的结构体，实现了一种基于优先级、计算资源分配以及执行状态的调度器。以下是对代码的拆分与分析：

1. `compute_capacity_percent` 函数：

```rust
pub fn compute_capacity_percent(scheduler_cores: usize) -> usize {
    if scheduler_cores >= 2 {
        (scheduler_cores - 1) * 100
    } else {
        0
    }
}
```

该函数的功能是计算调度器的计算容量百分比。对于 DTS 调度器，计算公式为 `(scheduler_cores - 1) * 100%`。`scheduler_cores` 表示调度器的核心数量。函数会确保至少有两个调度器核心；如果核心数小于 2，计算容量将被设置为 0。

2. `order_canister_round_states` 函数：

```rust
fn order_canister_round_states(&self, round_states: &mut [CanisterRoundState]) {
    round_states.sort_by_key(|rs| {
        (
            Reverse(rs.long_execution_mode),
            Reverse(rs.has_aborted_or_paused_execution),
            Reverse(rs.accumulated_priority),
            rs.canister_id,
        )
    });
}
```

该函数根据调度策略对 Canister 轮次状态进行排序。它接收一个可变引用的 CanisterRoundState 切片，并对其进行排序。排序规则是先按长执行模式，然后按是否有中止或暂停执行，再按累积优先级，最后按 Canister ID 排序。

3. `apply_scheduling_strategy` 函数：

这是一个较长的函数，负责实现调度策略。以下是函数的主要功能和关键部分的分析：

- 计算总的可分配计算资源百分比
- 计算所有 Canister 的计算资源分配百分比之和，确保它小于总的可分配计算资源百分比
- 计算每个 Canister 的累积优先级和执行状态
- 根据累积优先级、执行状态和计算资源分配，为每个 Canister 分配实际计算资源
- 对 Canister 进行排序，以确定调度顺序
- 创建一个新的 RoundSchedule，包含实际调度顺序和分配给每个 Canister 的计算资源

这个函数实现了一个基于累积优先级、计算资源分配和执行状态的调度策略。它会周期性地重置累积优先级，以便支持 Canister 集合和计算资源分配的更改。在每个调度轮次中，它会为 Canister 分配实际的计算资源，并根据其累积优先级、执行状态和计算资源分配对 Canister 进行排序。最后，它会创建一个新的 RoundSchedule，包含实际调度顺序和分配给每个 Canister 的计算资源。

优点：
- 考虑了累积优先级、计算资源分配和执行状态，使调度策略更加合理
- 周期性地重置累积优先级，支持 Canister 集合和计算资源分配的更改
- 通过使用 BTreeMap 存储 Canister 状态，提高了查找效率





这段代码主要实现了一个名为 `SchedulerImpl` 的结构体，该结构体负责执行 Canister 内的计算任务（如安装代码、执行子网消息等），并具有处理长时间运行的安装代码、排空子网队列等功能。以下是对代码的拆分与分析：

1. `new` 函数：

```rust
pub(crate) fn new(
    ...
) -> Self {
    ...
}
```

这是 `SchedulerImpl` 结构体的构造函数，用于创建一个新的 `SchedulerImpl` 实例。函数接收一系列参数，如调度器配置、子网 ID、执行环境等，然后将其存储在创建的 `SchedulerImpl` 实例中。

2. `advance_long_running_install_code` 函数：

```rust
fn advance_long_running_install_code(
    &self,
    ...
) -> (ReplicatedState, bool) {
    ...
}
```

这个函数的主要功能是处理长时间运行的安装代码。它接收一个可变的 `ReplicatedState` 对象，一个 `RoundLimits` 对象，一个包含长时间运行 Canister 的 ID 集合等。函数遍历长时间运行的 Canister，逐个执行它们的安装代码。当遇到一个已经暂停安装代码的 Canister 时，函数将返回新的 `ReplicatedState` 和一个布尔值，表示是否还有正在进行的长时间运行的安装代码。

3. `drain_subnet_queues` 函数：

```rust
fn drain_subnet_queues(
    &self,
    ...
) -> ReplicatedState {
    ...
}
```

这个函数的主要功能是排空子网队列，执行所有没有被长时间运行的任务阻塞的消息。函数接收一个可变的 `ReplicatedState` 对象，一个伪随机数生成器，一个 `RoundLimits` 对象等。函数循环处理子网队列中的消息，只要有可执行的消息且没有达到执行轮次的限制，就将其从队列中取出并执行。执行完所有可执行消息后，函数返回新的 `ReplicatedState`。





整个代码片段的作用是在一个执行周期内，模拟执行一组副本状态中的Canister，并返回执行后的副本状态和活动Canister的ID集合。该功能主要包含以下步骤：

1. 初始化一些变量和数据结构。
2. 如果需要，为Canister添加`Heartbeat`和`GlobalTimer`任务。
3. 进入一个迭代循环，直到达到每一轮的指令限制或所有Canister变为空闲。
4. 在迭代循环中，执行活动的Canister。
5. 在迭代结束后，移除所有剩余的`Heartbeat`和`GlobalTimer`任务，并应用优先级信用。
6. 导出Canister的相关度量。

接下来，我们将逐一分析这些代码片段。

**初始化一些变量和数据结构**

```rust
let measurement_scope =
    MeasurementScope::nested(&self.metrics.round_inner, measurement_scope);
let mut ingress_execution_results = Vec::new();
let mut is_first_iteration = true;
let mut round_filtered_canisters = FilteredCanisters::new();
let mut total_heap_delta = NumBytes::from(0);
```

这部分代码主要用于初始化一些变量和数据结构，如`ingress_execution_results`用于存储在执行过程中产生的入口消息执行结果，`is_first_iteration`标志表示是否为第一次迭代，`round_filtered_canisters`用于存储过滤后的Canister，`total_heap_delta`用于计算所有执行过程中的堆内存增量。

**为Canister添加`Heartbeat`和`GlobalTimer`任务**

```rust
{
    let _timer = self
        .metrics
        .round_inner_heartbeat_overhead_duration
        .start_timer();
    let now = state.time();
    for canister in state.canisters_iter_mut() {
        // Add `Heartbeat` or `GlobalTimer` for running canisters only.
        match canister.system_state.status {
            CanisterStatus::Running { .. } => {}
            CanisterStatus::Stopping { .. } | CanisterStatus::Stopped => {
                continue;
            }
        }
        // ...
    }
}
```

这部分代码负责检查每个Canister的状态，如果Canister的状态为`Running`，则为其添加`Heartbeat`和`GlobalTimer`任务。

**迭代循环**

在迭代循环中，执行以下操作：

1. 更新子网可用内存。
2. 将Canister分为活动和非活动Canister。
3. 执行活动的Canister。
4. 计算指令消耗和堆内存增量。
5. 将执行过的Canister放回状态中。
6. 更新入口消息执行结果。
7. 检查迭代终止条件。

该循环将持续进行，直到满足以下条件之一：

- 执行的指令数量为0。
- 达到每轮的指令限制。
- 堆内存增量达到每次迭代的最大值。

**移除剩余的`Heartbeat`和`GlobalTimer`任务，并应用优先级信用**

```rust
{
    let _timer = self
        .metrics
        .round_inner_heartbeat_overhead_duration
        .start_timer();
    // Remove all remaining `Heartbeat` and `GlobalTimer` tasks
    // because they will be added again in the next round.
    for canister in state.canisters_iter_mut() {
        canister.system_state.task_queue.retain(|task| match task {
            ExecutionTask::Heartbeat | ExecutionTask::GlobalTimer => false,
            _ => true,
        });
        // Also, apply priority credit for all the finished executions
        match canister.next_execution() {
            NextExecution::None
            | NextExecution::StartNew
            | NextExecution::ContinueInstallCode => canister.apply_priority_credit(),
            NextExecution::ContinueLong => {}
        }
    }
}
```

在迭代结束后，移除所有剩余的`Heartbeat`和`GlobalTimer`任务，因为它们将在下一轮中再次添加。同时，为所有完成的执行应用优先级信用。

**导出Canister的相关度量**

这部分代码负责导出与Canister相关的度量数据，例如：每轮执行的指令数、堆内存增量等。这些度量数据可用于监控和分析Canister的性能。

```rust
let executed_canister_ids = round_filtered_canisters.executed_canisters();
let executed_canister_ids_len = executed_canister_ids.len();
self.metrics
    .executed_canister_ids_per_round
    .observe(executed_canister_ids_len as f64);
```

这部分代码主要用于记录每轮执行的Canister数量，并将其作为度量值进行观察。这有助于理解在每个执行周期中，有多少Canister实际执行了指令。这有助于监控和分析整个系统的性能。

总结一下，这段代码模拟了在一个执行周期内执行一组副本状态中的Canister，并返回执行后的副本状态和活动Canister的ID集合。在整个过程中，会执行活动的Canister，处理相关任务，并更新相关度量数据。、



这是一个用于执行 canister 的 Rust 代码片段，并行使用线程池进行处理。核心功能是在给定的执行轮次中执行 canister，返回新的 canister 状态、ingress 结果、每个线程上执行的最大指令数和总堆增量。

我们将代码拆分为以下几个部分：

1. `execute_canisters_in_inner_round` 函数
2. `process_stopping_canisters` 函数
3. `purge_expired_ingress_messages` 函数
4. `observe_canister_metrics` 函数

### 1. `execute_canisters_in_inner_round` 函数

这个函数是整个代码片段的核心部分，它负责在给定的执行轮次中执行 canister。函数的输入参数包括按线程划分的 canister，执行轮次 ID，时间，网络拓扑等，输出参数为新的 canister 状态、ingress 结果和总堆增量。

#### 1.1 检查剩余指令数

```rust
if round_limits.reached() {
    return (
        canisters_by_thread.into_iter().flatten().collect(),
        vec![],
        NumBytes::from(0),
    );
}
```

如果没有剩余可执行的指令，函数将返回，不执行任何操作。

#### 1.2 准备执行线程的结果容器

```rust
let mut results_by_thread: Vec<ExecutionThreadResult> = canisters_by_thread
    .iter()
    .map(|_| Default::default())
    .collect();
```

为每个执行线程准备一个 `ExecutionThreadResult` 结构体，用于存储执行结果。

#### 1.3 分发内存并运行 canister

```rust
thread_pool.scoped(|scope| {
    // ...
});
```

使用线程池并行执行 canister。每个线程上的 canister 使用 `execute_canisters_on_thread` 函数执行。

#### 1.4 聚合执行结果

```rust
for mut result in results_by_thread.into_iter() {
    // ...
}
```

所有线程完成执行后，将各个线程的执行结果聚合起来，得到整个函数的执行结果。

### 2. `process_stopping_canisters` 函数

该函数负责将停止状态的 canister 从 canister 列表中移除。

```rust
fn process_stopping_canisters(&self, state: ReplicatedState) -> ReplicatedState {
    util::process_stopping_canisters(
        state,
        self.ingress_history_writer.as_ref(),
        self.own_subnet_id,
    )
}
```

### 3. `purge_expired_ingress_messages` 函数

该函数负责清除过期的 ingress 消息。

```rust
fn purge_expired_ingress_messages(&self, state: &mut ReplicatedState) {
    // ...
}
```

### 4. `observe_canister_metrics` 函数

这个函数用于观察 canister 的各种度量指标，如余额、二进制大小、内存使用情况等。

```rust
fn observe_canister_metrics(&self, canister: &CanisterState) {
    // ...
}
```

以上是对给定 Rust 代码片段的详细分析。该代码段实现了在一个执行轮次中并行执行 canister 的功能，示例代码包含了处理停止状态的 canister、清除过期 ingress 消息和观察 canister 指标的功能。整体上，这个代码片段设计合理、易于理解，但在执行效率和可扩展性方面可能还有优化空间。例如，在聚合执行结果时可以考虑使用并行算法进一步提升性能。





整个代码片段主要完成了两个核心功能：

1. 为 Canister 分配资源并收取费用。如果某个 Canister 未能支付费用，则将其卸载。
2. 在同一个子网中，将消息从一个 Canister 转移到另一个 Canister。

我们将逐个分析这两个核心功能的实现。

### 1. 为 Canister 分配资源并收取费用

```rust
fn charge_canisters_for_resource_allocation_and_usage(
    &self,
    state: &mut ReplicatedState,
    subnet_size: usize,
) {
    // ...
}
```

此函数的主要作用是遍历子网中的所有 Canister，为它们分配资源并收取费用。如果 Canister 无法支付费用，会将其卸载。

1. 遍历所有 Canister，并检查它们是否需要支付费用。
2. 如果需要支付费用，则调用 `cycles_account_manager.charge_canister_for_resource_allocation_and_usage()` 方法来收取费用。
3. 如果 Canister 无法支付费用，则将其卸载。

优点：这个函数实现了 Canister 的资源分配和费用收取功能，确保了系统的正常运行。

缺点：如果 Canister 数量较多，遍历所有 Canister 可能会影响性能。

### 2. 在同一个子网中，将消息从一个 Canister 转移到另一个 Canister

```rust
pub fn induct_messages_on_same_subnet(&self, state: &mut ReplicatedState) {
    // ...
}
```

此函数的主要作用是在同一个子网中，将消息从一个 Canister 转移到另一个 Canister。

1. 计算子网可用内存。
2. 遍历所有 Canister，将它们的输出消息转移到同一子网中的其他 Canister。
3. 更新相关指标。

优点：这个函数实现了在同一个子网中传递消息的功能，提高了消息传递的效率。

缺点：如果 Canister 数量较多，遍历所有 Canister 可能会影响性能。同时，如果输出队列中的消息较多，将消息从一个 Canister 转移到另一个 Canister 也可能会影响性能。

总结：整个代码片段实现了两个核心功能：为 Canister 分配资源并收取费用，以及在同一个子网中将消息从一个 Canister 转移到另一个 Canister。这两个功能都是为了维护整个系统的正常运行。但在实现过程中，可能会因为 Canister 数量较多而影响性能。





整个代码片段的主要作用是检查和维护 Canister（计算单元）的执行状态。其中包含四个主要函数，它们分别是：

1. `check_canister_invariants()`: 验证 Canister 的不变式是否成立，确保 Canister 内部状态有效。
2. `abort_paused_executions_above_limit()`: 中止超出限制的暂停执行。
3. `finish_round()`: 在一轮执行结束后，执行一些必要的维护操作。
4. `check_dts_invariants()`: 在一轮执行结束后，检查确定性时间切片（deterministic time slicing）的不变式。

接下来，我们将逐个分析这几个函数。

### 1. `check_canister_invariants()`

```rust
fn check_canister_invariants(
    &self,
    round_log: &ReplicaLogger,
    current_round: &ExecutionRound,
    state: &ReplicatedState,
    canister_ids: &BTreeSet<CanisterId>,
) -> bool {
    // ...
}
```

此函数的主要作用是遍历给定的 Canister 列表，检查每个 Canister 是否满足其不变式。如果所有 Canister 都满足不变式，则返回 `true`；否则返回 `false`。如果某个 Canister 不满足不变式，函数会记录一个警告日志。

### 2. `abort_paused_executions_above_limit()`

```rust
fn abort_paused_executions_above_limit(&self, state: &mut ReplicatedState) {
    // ...
}
```

此函数的主要作用是根据调度优先级，中止超过 `max_paused_executions` 限制的暂停执行。具体步骤如下：

1. 遍历所有 Canister，收集具有暂停执行的 Canister。
2. 根据调度优先级对收集到的 Canister 进行排序。
3. 中止超过 `max_paused_executions` 限制的暂停执行。

### 3. `finish_round()`

```rust
fn finish_round(&self, state: &mut ReplicatedState, current_round_type: ExecutionRoundType) {
    // ...
}
```

此函数在每轮执行结束后执行一些必要的维护操作。根据当前执行轮次的类型（检查点轮次或普通轮次），执行不同的操作。这些操作包括：

- 如果是检查点轮次：重置堆增量估计、清除预期的已编译 Wasm，以及在启用确定性时间切片时，中止所有暂停执行。
- 如果是普通轮次：在启用确定性时间切片时，中止超过限制的暂停执行。

最后调用 `check_dts_invariants()` 函数检查确定性时间切片的不变式。

### 4. `check_dts_invariants()`

```rust
fn check_dts_invariants(
    &self,
    state: &ReplicatedState,
    current_round_type: ExecutionRoundType,
) {
    // ...
}
```

此函数在每轮执行结束后，检查确定性时间切片（deterministic time slicing）的不变式。具体检查内容如下：

1. 心跳任务（Heartbeat）和全局定时器任务（GlobalTimer）只能在执行轮次中存在，执行结束后不应存在。
2. 暂停执行（PausedExecution）只能在普通轮次（OrdinaryRound）中存在，不能在检查点轮次（CheckpointRound）中存在。
3. 如果确定性时间切片被禁用，则不应存在暂停任务。但如果禁用是在最近的检查点发生的，仍可能存在已中止的任务。

总结：整个代码片段主要用于检查和维护 Canister 的执行状态。通过四个主要函数，分别对 Canister 的不变式进行验证、对暂停执行进行中止、在执行结束后进行必要的维护操作，以及对确定性时间切片的不变式进行检查。这些操作有助于确保 Canister 的正确执行，以及在执行过程中维护系统的稳定性和可靠性。












```rust


////////////////////////////////////////////////////////////////////////
/// Filtered Canisters

/// This struct represents a collection of canister IDs.
struct FilteredCanisters {
    /// Active canisters during the execution of the inner round.
    active_canister_ids: BTreeSet<CanisterId>,

    /// Canisters that were heap delta rate-limited during the execution of the inner round.
    rate_limited_canister_ids: BTreeSet<CanisterId>,
}

impl FilteredCanisters {
    fn new() -> Self {
        Self {
            active_canister_ids: BTreeSet::new(),
            rate_limited_canister_ids: BTreeSet::new(),
        }
    }

    fn add_canisters(
        &mut self,
        active_round_schedule: &RoundSchedule,
        rate_limited_ids: &[CanisterId],
    ) {
        self.active_canister_ids
            .extend(active_round_schedule.iter());
        self.rate_limited_canister_ids
            .extend(rate_limited_ids.iter());
    }
}

fn observe_instructions_consumed_per_message(
    logger: &ReplicaLogger,
    metrics: &SchedulerMetrics,
    canister: &CanisterState,
    consumed_instructions: NumInstructions,
    instruction_limit_per_message: NumInstructions,
) {
    // If the message consumed any non-zero instructions, count it towards the
    // messages we are going to charge cycles for.
    if consumed_instructions.get() > 0 {
        metrics.canister_messages_where_cycles_were_charged.inc();
    }
    metrics
        .instructions_consumed_per_message
        .observe(consumed_instructions.get() as f64);
    if consumed_instructions > instruction_limit_per_message {
        warn!(
            logger,
            "Execution consumed too many instructions: limit={} consumed={}, canister_id={}",
            instruction_limit_per_message,
            consumed_instructions,
            canister.canister_id()
        );
    }
}

/// This struct holds the result of a single execution thread.
#[derive(Default)]
struct ExecutionThreadResult {
    canisters: Vec<CanisterState>,
    ingress_results: Vec<(MessageId, IngressStatus)>,
    slices_executed: NumSlices,
    messages_executed: NumMessages,
    heap_delta: NumBytes,
    round_limits: RoundLimits,
}

/// Executes the given canisters one by one. For each canister it
/// - runs the heartbeat or timer handlers of the canister if needed,
/// - executes all messages of the canister.
/// The execution stops if `total_instruction_limit` is reached
/// or all canisters are processed.
#[allow(clippy::too_many_arguments)]
fn execute_canisters_on_thread(
    canisters_to_execute: Vec<CanisterState>,
    exec_env: &ExecutionEnvironment,
    config: &SchedulerConfig,
    metrics: Arc<SchedulerMetrics>,
    round_id: ExecutionRound,
    time: Time,
    network_topology: Arc<NetworkTopology>,
    logger: ReplicaLogger,
    rate_limiting_of_heap_delta: FlagStatus,
    deterministic_time_slicing: FlagStatus,
    mut round_limits: RoundLimits,
    subnet_size: usize,
) -> ExecutionThreadResult {
    // Since this function runs on a helper thread, we cannot use a nested scope
    // here. Instead, we propagate metrics to the outer scope manually via
    // `ExecutionThreadResult`.
    let measurement_scope =
        MeasurementScope::root(&metrics.round_inner_iteration_thread).dont_record_zeros();
    // These variables accumulate the results and will be returned at the end.
    let mut canisters = vec![];
    let mut ingress_results = vec![];
    let mut total_slices_executed = NumSlices::from(0);
    let mut total_messages_executed = NumMessages::from(0);
    let mut total_heap_delta = NumBytes::from(0);

    let instruction_limits = InstructionLimits::new(
        deterministic_time_slicing,
        config.max_instructions_per_message,
        config.max_instructions_per_slice,
    );

    for (rank, mut canister) in canisters_to_execute.into_iter().enumerate() {
        // If no more instructions are left or if heap delta is already too
        // large, then skip execution of the canister and keep its old state.
        if round_limits.reached() || total_heap_delta >= config.max_heap_delta_per_iteration {
            canisters.push(canister);
            continue;
        }

        // Process all messages of the canister until
        // - it has not tasks and input messages to execute
        // - or the canister is blocked by a long-running install code.
        // - or the instruction limit is reached.
        // - or the canister finishes a long execution
        loop {
            match canister.next_execution() {
                NextExecution::None | NextExecution::ContinueInstallCode => {
                    break;
                }
                NextExecution::StartNew | NextExecution::ContinueLong => {}
            }

            if round_limits.reached() {
                canister
                    .system_state
                    .canister_metrics
                    .interruped_during_execution += 1;
                break;
            }
            let measurement_scope = MeasurementScope::nested(
                &metrics.round_inner_iteration_thread_message,
                &measurement_scope,
            );
            let timer = metrics.msg_execution_duration.start_timer();

            let instructions_before = round_limits.instructions;
            let canister_had_paused_execution = canister.has_paused_execution();
            let ExecuteCanisterResult {
                canister: new_canister,
                instructions_used,
                heap_delta,
                ingress_status,
                description,
            } = execute_canister(
                exec_env,
                canister,
                instruction_limits.clone(),
                config.max_instructions_per_message_without_dts,
                Arc::clone(&network_topology),
                time,
                &mut round_limits,
                subnet_size,
            );
            ingress_results.extend(ingress_status);
            let round_instructions_executed =
                as_num_instructions(instructions_before - round_limits.instructions);
            let messages = NumMessages::from(instructions_used.map(|_| 1).unwrap_or(0));
            measurement_scope.add(round_instructions_executed, NumSlices::from(1), messages);
            if let Some(instructions_used) = instructions_used {
                total_messages_executed.inc_assign();
                observe_instructions_consumed_per_message(
                    &logger,
                    &metrics,
                    &new_canister,
                    instructions_used,
                    instruction_limits.message(),
                );
            }
            total_slices_executed.inc_assign();
            canister = new_canister;
            round_limits.instructions -=
                as_round_instructions(config.instruction_overhead_per_message);
            total_heap_delta += heap_delta;
            if rate_limiting_of_heap_delta == FlagStatus::Enabled {
                canister.scheduler_state.heap_delta_debit += heap_delta;
            }
            let msg_execution_duration = timer.stop_and_record();
            if msg_execution_duration > config.max_message_duration_before_warn_in_seconds {
                warn!(
                    logger,
                    "Finished executing message type {:?} on canister {:?} after {:?} seconds",
                    description.unwrap_or_default(),
                    canister.canister_id(),
                    msg_execution_duration;
                    messaging.canister_id => canister.canister_id().to_string(),
                );
            }
            if total_heap_delta >= config.max_heap_delta_per_iteration {
                break;
            }
            if canister_had_paused_execution && !canister.has_paused_execution() {
                // Break the loop, as the canister just finished its long execution
                break;
            }
        }
        if let Some(es) = &mut canister.execution_state {
            es.last_executed_round = round_id;
        }
        if !canister.has_input() || rank == 0 {
            // The very first canister is considered to have a full execution round for
            // scheduling purposes even if it did not complete within the round.
            canister.scheduler_state.last_full_execution_round = round_id;
        }
        canister.system_state.canister_metrics.executed += 1;
        canisters.push(canister);
        round_limits.instructions -=
            as_round_instructions(config.instruction_overhead_per_canister);
    }

    ExecutionThreadResult {
        canisters,
        ingress_results,
        slices_executed: total_slices_executed,
        messages_executed: total_messages_executed,
        heap_delta: total_heap_delta,
        round_limits,
    }
}

/// Updates end-of-round replicated state metrics (canisters, queues, cycles,
/// etc.).
fn observe_replicated_state_metrics(
    own_subnet_id: SubnetId,
    state: &ReplicatedState,
    current_round: ExecutionRound,
    metrics: &SchedulerMetrics,
    logger: &ReplicaLogger,
) {
    // Observe the number of registered canisters keyed by their status.
    let mut num_running_canisters = 0;
    let mut num_stopping_canisters = 0;
    let mut num_stopped_canisters = 0;

    let mut num_paused_exec = 0;
    let mut num_aborted_exec = 0;
    let mut num_paused_install = 0;
    let mut num_aborted_install = 0;

    let mut consumed_cycles_total = NominalCycles::new(0);

    let mut ingress_queue_message_count = 0;
    let mut ingress_queue_size_bytes = 0;
    let mut input_queues_message_count = 0;
    let mut input_queues_size_bytes = 0;
    let mut queues_response_bytes = 0;
    let mut queues_reservations = 0;
    let mut queues_oversized_requests_extra_bytes = 0;
    let mut canisters_not_in_routing_table = 0;
    let mut canisters_with_old_open_call_contexts = 0;
    let mut old_call_contexts_count = 0;

    state.canisters_iter().for_each(|canister| {
        match canister.status() {
            CanisterStatusType::Running => num_running_canisters += 1,
            CanisterStatusType::Stopping { .. } => num_stopping_canisters += 1,
            CanisterStatusType::Stopped => num_stopped_canisters += 1,
        }
        match canister.next_task() {
            Some(&ExecutionTask::PausedExecution(_)) => {
                num_paused_exec += 1;
            }
            Some(&ExecutionTask::PausedInstallCode(_)) => {
                num_paused_install += 1;
            }
            Some(&ExecutionTask::AbortedExecution { .. }) => {
                num_aborted_exec += 1;
            }
            Some(&ExecutionTask::AbortedInstallCode { .. }) => {
                num_aborted_install += 1;
            }
            Some(&ExecutionTask::Heartbeat) | Some(&ExecutionTask::GlobalTimer) | None => {}
        }
        consumed_cycles_total += canister
            .system_state
            .canister_metrics
            .consumed_cycles_since_replica_started;
        let queues = canister.system_state.queues();
        ingress_queue_message_count += queues.ingress_queue_message_count();
        ingress_queue_size_bytes += queues.ingress_queue_size_bytes();
        input_queues_message_count += queues.input_queues_message_count();
        input_queues_size_bytes += queues.input_queues_size_bytes();
        queues_response_bytes += queues.responses_size_bytes();
        queues_reservations += queues.reserved_slots();
        queues_oversized_requests_extra_bytes += queues.oversized_requests_extra_bytes();
        if state.routing_table().route(canister.canister_id().into()) != Some(own_subnet_id) {
            canisters_not_in_routing_table += 1;
        }
        if let Some(manager) = canister.system_state.call_context_manager() {
            let old_call_contexts =
                manager.call_contexts_older_than(state.time(), OLD_CALL_CONTEXT_CUTOFF_ONE_DAY);
            // Log all old call contexts, but not (nearly) every round.
            if current_round.get() % SPAMMY_LOG_INTERVAL_ROUNDS == 0 {
                for (origin, origin_time) in &old_call_contexts {
                    error!(
                        logger,
                        "Call context has been open for {:?}: origin: {:?}, respondent: {}",
                        state.time() - *origin_time,
                        origin,
                        canister.canister_id()
                    );
                }
            }
            if !old_call_contexts.is_empty() {
                old_call_contexts_count += old_call_contexts.len();
                canisters_with_old_open_call_contexts += 1;
            }
        }
    });
    metrics
        .old_open_call_contexts
        .with_label_values(&[OLD_CALL_CONTEXT_LABEL_ONE_DAY])
        .set(old_call_contexts_count as i64);
    metrics
        .canisters_with_old_open_call_contexts
        .with_label_values(&[OLD_CALL_CONTEXT_LABEL_ONE_DAY])
        .set(canisters_with_old_open_call_contexts as i64);
    let streams_response_bytes = state
        .metadata
        .streams()
        .responses_size_bytes()
        .values()
        .sum();

    metrics
        .current_heap_delta
        .set(state.metadata.heap_delta_estimate.get() as i64);

    // Add the consumed cycles by canisters that were deleted.
    consumed_cycles_total += state
        .metadata
        .subnet_metrics
        .consumed_cycles_by_deleted_canisters;

    // Add the consumed cycles in ecdsa outcalls.
    consumed_cycles_total += state.metadata.subnet_metrics.consumed_cycles_ecdsa_outcalls;

    // Add the consumed cycles in http outcalls.
    consumed_cycles_total += state.metadata.subnet_metrics.consumed_cycles_http_outcalls;

    metrics.observe_consumed_cycles(consumed_cycles_total);

    metrics
        .ecdsa_signature_agreements
        .set(state.metadata.subnet_metrics.ecdsa_signature_agreements as i64);

    let observe_reading = |status: CanisterStatusType, num: i64| {
        metrics
            .registered_canisters
            .with_label_values(&[&status.to_string()])
            .set(num);
    };
    observe_reading(CanisterStatusType::Running, num_running_canisters);
    observe_reading(CanisterStatusType::Stopping, num_stopping_canisters);
    observe_reading(CanisterStatusType::Stopped, num_stopped_canisters);

    metrics
        .canister_paused_execution
        .observe(num_paused_exec as f64);
    metrics
        .canister_aborted_execution
        .observe(num_aborted_exec as f64);
    metrics
        .canister_paused_install_code
        .observe(num_paused_install as f64);
    metrics
        .canister_aborted_install_code
        .observe(num_aborted_install as f64);

    metrics
        .available_canister_ids
        .set(state.metadata.available_canister_ids() as i64);

    metrics.observe_input_messages(MESSAGE_KIND_INGRESS, ingress_queue_message_count);
    metrics.observe_input_queues_size_bytes(MESSAGE_KIND_INGRESS, ingress_queue_size_bytes);
    metrics.observe_input_messages(MESSAGE_KIND_CANISTER, input_queues_message_count);
    metrics.observe_input_queues_size_bytes(MESSAGE_KIND_CANISTER, input_queues_size_bytes);

    metrics.observe_queues_response_bytes(queues_response_bytes);
    metrics.observe_queues_reservations(queues_reservations);
    metrics.observe_oversized_requests_extra_bytes(queues_oversized_requests_extra_bytes);
    metrics.observe_streams_response_bytes(streams_response_bytes);

    metrics
        .ingress_history_length
        .set(state.metadata.ingress_history.len() as i64);
    metrics
        .canisters_not_in_routing_table
        .set(canisters_not_in_routing_table);
}

/// Helper function that checks if a message can be executed:
///     1. A message cannot be executed if it is directed to a canister
///     with another long-running execution in progress.
///     2. Install code messages can only be executed sequentially.
fn can_execute_msg(
    msg: &CanisterMessage,
    ongoing_long_install_code: bool,
    long_running_canister_ids: &BTreeSet<CanisterId>,
) -> bool {
    if let Some(effective_canister_id) = msg.effective_canister_id() {
        if long_running_canister_ids.contains(&effective_canister_id) {
            return false;
        }
    }

    if ongoing_long_install_code {
        let maybe_instal_code_method = match msg {
            CanisterMessage::Ingress(ingress) => {
                Ic00Method::from_str(ingress.method_name.as_str()).ok()
            }
            CanisterMessage::Request(request) => {
                Ic00Method::from_str(request.method_name.as_str()).ok()
            }
            CanisterMessage::Response(_) => None,
        };

        // Only one install code message allowed at a time.
        if let Some(Ic00Method::InstallCode) = maybe_instal_code_method {
            return false;
        }
    }

    true
}

/// Based on the type of the subnet message to execute, figure out its
/// instruction limits.
///
/// This is primarily done because upgrading a canister might need to
/// (de)-serialize a large state and thus consume a lot of instructions.
fn get_instructions_limits_for_subnet_message(
    dts: FlagStatus,
    config: &SchedulerConfig,
    msg: &CanisterMessage,
) -> InstructionLimits {
    let default_limits = InstructionLimits::new(
        FlagStatus::Disabled,
        config.max_instructions_per_message_without_dts,
        config.max_instructions_per_message_without_dts,
    );
    let method_name = match &msg {
        CanisterMessage::Response(_) => {
            return default_limits;
        }
        CanisterMessage::Ingress(ingress) => &ingress.method_name,
        CanisterMessage::Request(request) => &request.method_name,
    };

    use Ic00Method::*;
    match Ic00Method::from_str(method_name) {
        Ok(method) => match method {
            CanisterStatus
            | CreateCanister
            | DeleteCanister
            | DepositCycles
            | ECDSAPublicKey
            | RawRand
            | SetController
            | HttpRequest
            | SetupInitialDKG
            | SignWithECDSA
            | ComputeInitialEcdsaDealings
            | StartCanister
            | StopCanister
            | UninstallCode
            | UpdateSettings
            | BitcoinGetBalance
            | BitcoinGetUtxos
            | BitcoinSendTransaction
            | BitcoinSendTransactionInternal
            | BitcoinGetCurrentFeePercentiles
            | BitcoinGetSuccessors
            | ProvisionalCreateCanisterWithCycles
            | ProvisionalTopUpCanister => default_limits,
            InstallCode => InstructionLimits::new(
                dts,
                config.max_instructions_per_install_code,
                config.max_instructions_per_install_code_slice,
            ),
        },
        Err(_) => default_limits,
    }
}

```



