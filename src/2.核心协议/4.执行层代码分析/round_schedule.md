在这段代码中，我们可以看到有3个主要的结构体：`CanisterRoundState`，`SchedulingOrder` 和 `RoundSchedule`。

它们分别表示：

一个小巧的结构体，用于表示在调度过程中有关 Canister 的信息；

一个表示 Canister 的调度顺序的结构体；

以及一个表示整个调度过程中 Canister 的状态的结构体。

接下来，我们将逐个分析这些结构体及其关键方法。



### CanisterRoundState 结构体

```
#[derive(Clone, Debug)]
pub(super) struct CanisterRoundState {
    pub(super) canister_id: CanisterId,
    pub(super) accumulated_priority: AccumulatedPriority,
    pub(super) compute_allocation: ComputeAllocation,
    pub(super) long_execution_mode: LongExecutionMode,
    pub(super) has_aborted_or_paused_execution: bool,
}
```

`CanisterRoundState` 结构体包含了一些与 Canister 相关的信息，如 Canister ID、累积优先级、计算分配、长时间执行模式以及是否有中止或暂停的执行。这个结构体在调度过程中用于存储 Canister 的各种状态信息。

### SchedulingOrder 结构体

```
#[derive(Debug, Default)]
pub(super) struct SchedulingOrder<P, N, R> {
    pub prioritized_long_canister_ids: P,
    pub new_canister_ids: N,
    pub opportunistic_long_canister_ids: R,
}
```

`SchedulingOrder` 结构体包含了三个有序的 Canister ID 集合，分别为：优先级较高的长时间执行 Canister；新的执行 Canister；以及空闲时执行的长时间执行 Canister。这个结构体在调度过程中用于表示 Canister 的调度顺序。

### RoundSchedule 结构体

```
#[derive(Debug, Default)]
pub struct RoundSchedule {
    pub scheduler_cores: usize,
    pub long_execution_cores: usize,
    pub ordered_new_execution_canister_ids: Vec<CanisterId>,
    pub ordered_long_execution_canister_ids: Vec<CanisterId>,
}
```

`RoundSchedule` 结构体包含了整个调度过程中 Canister 的状态信息，如调度器的核心数量、用于长时间执行的核心数量、按顺序排列的新执行 Canister ID 以及按顺序排列的长时间执行 Canister ID。这个结构体在调度过程中用于存储整个调度过程的状态信息。

现在我们来看一下 `RoundSchedule` 结构体的一些关键方法：

#### new 方法

```
pub fn new(
    scheduler_cores: usize,
    long_execution_cores: usize,
    ordered_new_execution_canister_ids: Vec<CanisterId>,
    ordered_long_execution_canister_ids: Vec<CanisterId>,
) -> Self {
    RoundSchedule {
        scheduler_cores,
        long_execution_cores: long_execution_cores
            .min(ordered_long_execution_canister_ids.len()),
        ordered_new_execution_canister_ids,
        ordered_long_execution_canister_ids,
    }
}
```

`new` 方法用于创建一个新的 `RoundSchedule` 实例。它接收调度器核心数、用于长时间执行的核心数、按顺序排列的新执行 Canister ID 以及按顺序排列的长时间执行 Canister ID 作为参数。

#### iter 方法

```
pub(super) fn iter(&self) -> impl Iterator<Item = &CanisterId> {
    self.ordered_long_execution_canister_ids
        .iter()
        .chain(self.ordered_new_execution_canister_ids.iter())
}
```

`iter` 方法用于返回一个迭代器，用于遍历 `RoundSchedule` 中的所有 Canister ID。这个方法将长时间执行的 Canister ID 和新执行的 Canister ID 链接在一起，以便可以轻松地遍历它们。

#### scheduling_order 方法

```
pub(super) fn scheduling_order(
    &self,
) -> SchedulingOrder<
    impl Iterator<Item = &CanisterId>,
    impl Iterator<Item = &CanisterId>,
    impl Iterator<Item = &CanisterId>,
> {
    SchedulingOrder {
        prioritized_long_canister_ids: self
            .ordered_long_execution_canister_ids
            .iter()
            .take(self.long_execution_cores),
        new_canister_ids: self.ordered_new_execution_canister_ids.iter(),
        opportunistic_long_canister_ids: self
            .ordered_long_execution_canister_ids
            .iter()
            .skip(self.long_execution_cores),
    }
}
```

`scheduling_order` 方法返回一个 `SchedulingOrder` 实例，其中包含了按优先级排列的长时间执行 Canister ID、新执行 Canister ID 以及空闲时执行的长时间执行 Canister ID。这个方法通过对 `ordered_long_execution_canister_ids` 进行切片操作，将其分为两个部分：优先级较高的长时间执行 Canister 和空闲时执行的长时间执行 Canister。

结合以上分析，我们可以得出这段代码的主要作用是定义了一个调度系统，用于管理不同类型的 Canister 的执行顺序和优先级。`CanisterRoundState` 结构体用于存储有关 Canister 的信息；`SchedulingOrder` 结构体用于表示 Canister 的调度顺序；`RoundSchedule` 结构体用于存储整个调度过程的状态信息，并提供了一些关键方法，以便在调度过程中对 Canister 进行管理。









这段代码的核心功能是实现一个轮询调度器，用于在 IC 上调度 Canister 的执行。重要的数据结构有 `CanisterRoundState`，`SchedulingOrder` 和 `RoundSchedule`。接下来，我们将逐个分析这些代码。

### CanisterRoundState 结构体

```rust
/// Round metrics required to prioritize a canister.
#[derive(Clone, Debug)]
pub(super) struct CanisterRoundState {
    /// Copy of Canister ID
    pub(super) canister_id: CanisterId,
    /// Copy of Canister SchedulerState::accumulated_priority
    pub(super) accumulated_priority: AccumulatedPriority,
    /// Copy of Canister SchedulerState::compute_allocation
    pub(super) compute_allocation: ComputeAllocation,
    /// Copy of Canister SchedulerState::long_execution_mode
    pub(super) long_execution_mode: LongExecutionMode,
    /// True when there is an aborted or paused long update execution.
    /// Note: this doesn't include paused or aborted install codes.
    pub(super) has_aborted_or_paused_execution: bool,
}
```

这个结构体包含了一些与 Canister 有关的信息，用于调度器优先级的计算。它包含以下字段：

1. `canister_id`：Canister 的唯一标识。
2. `accumulated_priority`：累积的优先级，用于调度决策。
3. `compute_allocation`：计算分配，表示 Canister 被分配的计算资源量。
4. `long_execution_mode`：长执行模式，表示 Canister 是否执行长时间运行的任务。
5. `has_aborted_or_paused_execution`：表示 Canister 是否有中断或暂停的长时间任务。



### SchedulingOrder 结构体

```rust
/// Represents three ordered active Canister ID groups to schedule.
#[derive(Debug, Default)]
pub(super) struct SchedulingOrder<P, N, R> {
    /// Prioritized long executions.
    pub prioritized_long_canister_ids: P,
    /// New executions.
    pub new_canister_ids: N,
    /// To be executed when the Canisters from previous two groups are idle.
    pub opportunistic_long_canister_ids: R,
}
```

这个结构体表示了三个有序的 Canister ID 分组，用于执行调度。它包含以下字段：

1. `prioritized_long_canister_ids`：具有优先级的长时间执行任务的 Canister ID。
2. `new_canister_ids`：具有新任务的 Canister ID。
3. `opportunistic_long_canister_ids`：当前两组 Canister 空闲时，可执行长时间任务的 Canister ID。



### RoundSchedule 结构体

```rust
/// Represents the order in which the Canister IDs are be scheduled during the whole current round.
#[derive(Debug, Default)]
pub struct RoundSchedule {
    /// Total number of scheduler cores.
    pub scheduler_cores: usize,
    /// Number of cores dedicated for long executions.
    pub long_execution_cores: usize,
    /// Ordered Canister IDs with new executions.
    pub ordered_new_execution_canister_ids: Vec<CanisterId>,
    /// Ordered Canister IDs with long executions.
    pub ordered_long_execution_canister_ids: Vec<CanisterId>,
}
```

这个结构体表示整个调度周期内 Canister ID 的执行顺序。它包含以下字段：

1. `scheduler_cores`：调度器的总核心数。
2. `long_execution_cores`：用于长时间执行任务的核心数。
3. `ordered_new_execution_canister_ids`：具有新任务的 Canister ID 的有序列表。
4. `ordered_long_execution_canister_ids`：具有长时间执行任务的 Canister ID 的有序列表。













这是一个名为 `RoundSchedule` 的结构体定义及其实现。`RoundSchedule` 用于存储执行调度过程中的 Canister ID 顺序。代码主要包括结构体定义、方法定义以及一些核心逻辑。

让我们逐步分析代码。

1. 结构体定义：

   ```rust
   #[derive(Debug, Default)]
   pub struct RoundSchedule {
       pub scheduler_cores: usize,
       pub long_execution_cores: usize,
       pub ordered_new_execution_canister_ids: Vec<CanisterId>,
       pub ordered_long_execution_canister_ids: Vec<CanisterId>,
   }
   ```

   `RoundSchedule` 结构体具有四个公共字段：

   - `scheduler_cores`：调度器总共的核心数量。
   - `long_execution_cores`：用于执行长时间任务的核心数量。
   - `ordered_new_execution_canister_ids`：一个 CanisterId 类型的向量，表示新执行任务的 Canister ID 顺序。
   - `ordered_long_execution_canister_ids`：一个 CanisterId 类型的向量，表示长执行任务的 Canister ID 顺序。

2. 结构体方法实现：

   `impl RoundSchedule` 块包含了 `RoundSchedule` 结构体的方法实现。

   - `new` 方法：创建并返回一个新的 `RoundSchedule` 实例。

     ```rust
     pub fn new(
         scheduler_cores: usize,
         long_execution_cores: usize,
         ordered_new_execution_canister_ids: Vec<CanisterId>,
         ordered_long_execution_canister_ids: Vec<CanisterId>,
     ) -> Self {
         RoundSchedule {
             scheduler_cores,
             long_execution_cores: long_execution_cores
                 .min(ordered_long_execution_canister_ids.len()),
             ordered_new_execution_canister_ids,
             ordered_long_execution_canister_ids,
         }
     }
     ```

   - `iter` 方法：返回一个遍历 `ordered_long_execution_canister_ids` 和 `ordered_new_execution_canister_ids` 的迭代器。

     ```rust
     pub(super) fn iter(&self) -> impl Iterator<Item = &CanisterId> {
         self.ordered_long_execution_canister_ids
             .iter()
             .chain(self.ordered_new_execution_canister_ids.iter())
     }
     ```

   - `scheduling_order` 方法：返回一个 `SchedulingOrder` 结构，包含三个迭代器，分别表示优先级高的长执行任务、新任务和优先级低的长执行任务。

     ```rust
     pub(super) fn scheduling_order(
         &self,
     ) -> SchedulingOrder<
         impl Iterator<Item = &CanisterId>,
         impl Iterator<Item = &CanisterId>,
         impl Iterator<Item = &CanisterId>,
     > {
         // ...
     }
     ```

   - `filter_canisters` 方法：根据 Canister 的状态和堆增量限制过滤 Canister，返回一个新的 `RoundSchedule` 实例和受限   Canister ID 的向量。

     ```rust
     pub(super) fn filter_canisters<F>(
         &self,
         canister_filter: F,
         heap_delta_limit: NumBytes,
     ) -> (Self, Vec<CanisterId>)
     where
         F: Fn(&CanisterId) -> Option<&CanisterState>,
     {
         // ...
     }
     ```

3. 核心逻辑：

   - `scheduling_order` 方法中的核心逻辑是将 Canister ID 分为三类：优先级高的长执行任务、新任务和优先级低的长执行任务。然后，将这三类任务分别用迭代器表示，并将它们组合成一个 `SchedulingOrder` 结构。

   - `filter_canisters` 方法的核心逻辑是根据 Canister 的状态和堆增量限制过滤 Canister。首先，通过 `canister_filter` 函数筛选出有效 Canister，并将它们的堆增量累加，直到达到 `heap_delta_limit`。接着，将未达到限制的 Canister 分为两类：新任务和长执行任务。最后，将这两类任务的 Canister ID 分别存储在新的 `RoundSchedule` 实例中，并返回该实例以及受限 Canister ID 的向量。

总之，这段代码定义了一个名为 `RoundSchedule` 的结构体，用于存储执行调度过程中的 Canister ID 顺序。它还实现了一些与 Canister ID 过滤和排序相关的方法，以便在调度器中使用。









这段代码是 `RoundSchedule` 结构体的实现部分，包含一个构造函数 `new`，和两个方法 `iter` 和 `scheduling_order`。

### new 方法

这个方法用于创建一个新的 `RoundSchedule` 实例，它接受以下参数：

1. `scheduler_cores`：调度器的总核心数。
2. `long_execution_cores`：用于长时间执行任务的核心数。
3. `ordered_new_execution_canister_ids`：具有新任务的 Canister ID 的有序列表。
4. `ordered_long_execution_canister_ids`：具有长时间执行任务的 Canister ID 的有序列表。

构造函数的实现保证 `long_execution_cores` 不会超过 `ordered_long_execution_canister_ids` 的长度。这可以确保分配给长时间执行任务的核心数不会超过实际的长时间执行 Canister 数量。

### iter 方法

这个方法返回一个 Iterator，用于迭代整个调度周期内的 Canister ID。首先，迭代长时间执行任务的 Canister，然后迭代具有新任务的 Canister。

### scheduling_order 方法

这个方法返回一个 `SchedulingOrder` 实例，表示整个调度周期内的 Canister ID 分组。它包含以下三个 Canister ID 分组：

1. `prioritized_long_canister_ids`：具有优先级的长时间执行任务的 Canister ID。在这个分组中，只包含前 `long_execution_cores` 个具有最高优先级的长时间执行任务的 Canister。
2. `new_canister_ids`：具有新任务的 Canister ID。这些 Canister 将在新执行核心上进行调度，以确保它们的预订和低延迟。
3. `opportunistic_long_canister_ids`：剩余具有长时间执行任务的 Canister ID。它们将跨所有核心按优先级顺序进行调度，从下一个可用的新执行核心开始。

这个方法的主要目的是在调度器中为 Canister 分配核心，以确保不同类型的任务能够得到合理的资源分配和执行顺序。

```rust
impl RoundSchedule {
    pub fn new(
        scheduler_cores: usize,
        long_execution_cores: usize,
        ordered_new_execution_canister_ids: Vec<CanisterId>,
        ordered_long_execution_canister_ids: Vec<CanisterId>,
    ) -> Self {
        RoundSchedule {
            scheduler_cores,
            long_execution_cores: long_execution_cores
                .min(ordered_long_execution_canister_ids.len()),
            ordered_new_execution_canister_ids,
            ordered_long_execution_canister_ids,
        }
    }

    pub(super) fn iter(&self) -> impl Iterator<Item = &CanisterId> {
        self.ordered_long_execution_canister_ids
            .iter()
            .chain(self.ordered_new_execution_canister_ids.iter())
    }

    pub(super) fn scheduling_order(
        &self,
    ) -> SchedulingOrder<
        impl Iterator<Item = &CanisterId>,
        impl Iterator<Item = &CanisterId>,
        impl Iterator<Item = &CanisterId>,
    > {
        SchedulingOrder {
            // To guarantee progress and minimize the potential waste of an abort, top
            // `long_execution_cores` canisters with prioritized long execution mode and highest
            // priority get scheduled on long execution cores.
            prioritized_long_canister_ids: self
                .ordered_long_execution_canister_ids
                .iter()
                .take(self.long_execution_cores),
            // Canisters with no pending long executions get scheduled across new execution
            // cores according to their round priority as the regular scheduler does. This will
            // guarantee their reservations; and ensure low latency except immediately after a long
            // message execution.
            new_canister_ids: self.ordered_new_execution_canister_ids.iter(),
            // Remaining canisters with long pending executions get scheduled across
            // all cores according to their priority order, starting from the next available core onto which a new
            // execution canister would have been scheduled.
            opportunistic_long_canister_ids: self
                .ordered_long_execution_canister_ids
                .iter()
                .skip(self.long_execution_cores),
        }
    }
}
```



### RoundSchedule 方法

接下来，我们将分析 `RoundSchedule` 结构体中的一些关键方法。

#### filter_canisters 方法

```rust
pub fn filter_canisters(
        &self,
        canisters: &BTreeMap<CanisterId, CanisterState>,
        heap_delta_rate_limit: NumBytes,
        rate_limiting_of_heap_delta: FlagStatus,
    ) -> (Self, Vec<CanisterId>) {
        let mut rate_limited_canister_ids = vec![];

        // Collect all active canisters and their next executions.
        //
        // It is safe to use a `HashMap`, as we'll only be doing lookups.
        let canister_next_executions: HashMap<_, _> = canisters
            .iter()
            .filter_map(|(canister_id, canister)| {
                if rate_limiting_of_heap_delta == FlagStatus::Enabled
                    && canister.scheduler_state.heap_delta_debit >= heap_delta_rate_limit
                {
                    // Record and filter out rate limited canisters.
                    rate_limited_canister_ids.push(*canister_id);
                    return None;
                }

                let next_execution = canister.next_execution();
                match next_execution {
                    // Filter out canisters with no messages or with paused installations.
                    NextExecution::None | NextExecution::ContinueInstallCode => None,

                    NextExecution::StartNew | NextExecution::ContinueLong => {
                        Some((canister_id, next_execution))
                    }
                }
            })
            .collect();

        let ordered_new_execution_canister_ids = self
            .ordered_new_execution_canister_ids
            .iter()
            .filter(|canister_id| canister_next_executions.contains_key(canister_id))
            .cloned()
            .collect();

        let ordered_long_execution_canister_ids = self
            .ordered_long_execution_canister_ids
            .iter()
            .filter(
                |canister_id| match canister_next_executions.get(canister_id) {
                    Some(NextExecution::ContinueLong) => true,

                    // We expect long execution, but there is none,
                    // so the long execution was finished in the
                    // previous inner round.
                    //
                    // We should avoid scheduling this canister to:
                    // 1. Avoid the canister to bypass the logic in
                    //    `apply_scheduling_strategy()`.
                    // 2. Charge canister for resources at the end
                    //    of the round.
                    Some(NextExecution::StartNew) => false,

                    None // No such canister. Should not happen.
                        | Some(NextExecution::None) // Idle canister.
                        | Some(NextExecution::ContinueInstallCode) // Subnet message.
                         => false,
                },
            )
            .cloned()
            .collect();

        (
            RoundSchedule::new(
                self.scheduler_cores,
                self.long_execution_cores,
                ordered_new_execution_canister_ids,
                ordered_long_execution_canister_ids,
            ),
            rate_limited_canister_ids,
        )
    }
```

这个方法会生成一个只包含活跃 Canister 的调度计划，并返回一组受限 Canister 的集合。在这个方法中，会根据 Canister 的 next_execution 过滤 Canister。如果 Canister 的堆增量超过限制，它们将被视为受限并被过滤掉。

#### partition_canisters_to_cores 方法

```rust
pub(super) fn partition_canisters_to_cores(
        &self,
        mut canisters: BTreeMap<CanisterId, CanisterState>,
    ) -> (Vec<Vec<CanisterState>>, BTreeMap<CanisterId, CanisterState>) {
        let mut canisters_partitioned_by_cores = vec![vec![]; self.scheduler_cores];

        let mut idx = 0;
        let scheduling_order = self.scheduling_order();
        for canister_id in scheduling_order.prioritized_long_canister_ids {
            let canister_state = canisters.remove(canister_id).unwrap();
            canisters_partitioned_by_cores[idx].push(canister_state);
            idx += 1;
        }
        let last_prioritized_long = idx;
        let new_execution_cores = self.scheduler_cores - last_prioritized_long;
        debug_assert!(new_execution_cores > 0);
        for canister_id in scheduling_order.new_canister_ids {
            let canister_state = canisters.remove(canister_id).unwrap();
            canisters_partitioned_by_cores[idx].push(canister_state);
            idx = last_prioritized_long
                + (idx - last_prioritized_long + 1) % new_execution_cores.max(1);
        }
        for canister_id in scheduling_order.opportunistic_long_canister_ids {
            let canister_state = canisters.remove(canister_id).unwrap();
            canisters_partitioned_by_cores[idx].push(canister_state);
            idx = (idx + 1) % self.scheduler_cores;
        }

        (canisters_partitioned_by_cores, canisters)
    }
```

这个方法将 Canister 分配给可用的执行核心。给定一个 RoundSchedule，它将按照调度顺序，将 Canister 分配给执行核心。例如，具有优先级的长时间执行任务的 Canister 将被分配给长时间执行核心，具有新任务的 Canister 将被分配给其他核心，剩余具有长时间执行任务的 Canister 将根据优先级跨所有核心进行分配。

这个方法的结果是一组按核心划分的 Canister 和一个包含未执行 Canister 的映射。
