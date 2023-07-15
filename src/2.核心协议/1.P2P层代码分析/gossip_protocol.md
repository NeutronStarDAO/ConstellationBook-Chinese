这段代码实现了 Gossip 协议的功能，即在 IC 中传播信息。代码中主要定义了一个 `Gossip` 特征（trait）和它的实现 `GossipImpl` ，就好像制定了一份规则（trait）和相应的执行方案（impl）。

Gossip 广播协议为副本提供了具有最终/有限传输保证的工件池。P2P 层将这些池中的工件视为二进制 blob 结构。代码的核心功能是实现 Gossip 协议以广播和接收工件。我们将按照结构体和方法的顺序对代码进行拆分和分析。



## 特征（Trait）

假如你正在编写一个关于动物园的电脑游戏。在这个游戏中，你有各种各样的动物，像狮子，猴子，大象等等。每一种动物都有一些相同的行为，比如吃饭，睡觉，移动。但是，它们各自的行为方式可能会有所不同。比如猴子可能会在树上跳来跳去，而大象只会在地面上缓慢移动。

那在 Rust 中，`trait` 就像是一份 “ 行为清单 ” 或者说是 “ 能力列表 ” 。这份清单列出了一种动物应该有哪些能力，比如吃饭、睡觉、移动。然后每一种动物（比如狮子、猴子、大象）都需要自己去决定如何完成这个 “ 行为清单 ” 上的任务。比如，“ 移动 ” 对于猴子来说可能就是在树上跳来跳去，对于大象就是在地面上缓慢移动。

也就是说，`trait` 其实就是一种工具，帮助我们定义了一个或多个行为，这些行为是我们期望所有实现了这个 `trait` 的类型都应该有的。然后每一种具体的类型，比如狮子、猴子和大象，都需要自己决定如何实现这些行为。

通过这种方式，我们就可以让不同的类型共享相同的行为，同时还可以保持每种类型对这些行为的独特实现。所以叫 “ 特征 ” 。这也是 Rust 中的多态性的一种体现，让我们能够用一种通用的方式处理不同的类型，同时还能保留各自的独特性。



所以什么是特征？😏

特征就每个个体特有的性质。

比如下面这段代码：

```rust
pub trait Gossip {
    // 定义了四种关联类型
    type GossipAdvert;
    type GossipChunkRequest;
    type GossipChunk;
    type GossipRetransmissionRequest;

    // 定义了一系列方法
    fn on_gossip_advert(&self, gossip_advert: Self::GossipAdvert, peer_id: NodeId);
    fn on_chunk_request(&self, gossip_request: Self::GossipChunkRequest, node_id: NodeId);
    fn on_gossip_chunk(&self, gossip_chunk: Self::GossipChunk, peer_id: NodeId);
    fn broadcast_advert(&self, advert_request: Self::GossipAdvert, dst: ArtifactDestination);
    fn on_gossip_retransmission_request(&self, gossip_request: Self::GossipRetransmissionRequest, node_id: NodeId);
    fn on_peer_up(&self, peer_id: NodeId);
    fn on_peer_down(&self, peer_id: NodeId);
    fn on_gossip_timer(&self);
}
```

这个 `Gossip` 特征定义了一种 Gossip 协议，任何实现了这个特征的类型都需要遵循这个协议，提供上述方法的具体实现。**这样，不同的类型就可以通过实现这个特征来表明它们都支持 Gossip 协议。**让我们逐个来看这个特征中定义的各个部分。

这个特征定义了四种关联类型：`GossipAdvert`、`GossipChunkRequest`、`GossipChunk` 和 `GossipRetransmissionRequest`。关联类型是 Rust 中一个高级的概念，它们在特征内部定义，并被特征的方法使用。实现特征的具体类型需要指定这些关联类型的具体类型。

然后，我们看到这个特征定义了一系列方法。所有实现了 `Gossip` 特征的类型都需要提供这些方法的具体实现：

1. `on_gossip_advert`: 当我们从某个副本接收到一个公告（advert）时，这个方法会被调用。这个方法需要两个参数，一个是公告本身，一个是发送公告的副本的 ID 。

2. `on_chunk_request`: 当我们从某个副本接收到一个 chunk 请求时，这个方法会被调用。这个方法同样需要两个参数，一个是 chunk 请求，一个是发送请求的副本的 ID 。

3. `on_gossip_chunk`: 当我们从某个副本接收到一个 gossip chunk 时，这个方法会被调用。它将 chunk 添加到正在构建的 artifact 中。如果下载完成，artifact 将被交给 artifact 管理器。参数同样包括 chunk 本身和发送 chunk 的副本 ID 。

4. `broadcast_advert`: 这个方法用于向其他副本广播一个公告。它需要两个参数，一个是公告本身，一个是公告的目标。

5. `on_gossip_retransmission_request`: 当我们从某个副本接收到一个重新传输的请求时，这个方法会被调用。参数包括请求本身和发送请求的副本的 ID 。

6. `on_peer_up` 和 `on_peer_down`: 这两个方法分别用于处理副本上线和下线的事件。它们只需要一个参数，即上线或下线的副本的 ID 。

7. `on_gossip_timer`: 这个方法会被一个专门的线程定期调用。

有了 Gossip 特征，我们再接着往下看。👇



`ArtifactDestination` 是一个枚举类型。枚举类型是一种特殊的数据类型，可以把它看成一个菜单、或者是选择题。

我给你举一个例子。你要订一杯咖啡，咖啡有好几种口味，比如浓咖啡（Espresso）、美式咖啡（Americano）、卡布奇诺（Cappuccino）。我们可以把这些咖啡口味想象成一个枚举类型，叫做 `CoffeeFlavor`，然后 `Espresso` 、`Americano` 、`Cappuccino` 就是这个枚举类型的几个选项。

在代码中，你可以声明一个变量的类型为 `CoffeeFlavor` ，然后给它赋值为 `Espresso` 或者 `Americano` 或者 `Cappuccino` 。这样当你在处理这个变量的时候，你就知道它只能是这几种咖啡中的一种，不会有别的奇怪的选项出现。

这样的好处是，当你需要处理一种有限的几个选项的情况时，代码会更清晰、更安全，你可以明确地知道这个变量的所有可能的值，而不必担心出现意外的情况。

枚举类型 `ArtifactDestination` 定义了如何分发广告的策略。目前只有一个选项：`SendToAllPeers`，也就是将公告发送给所有的副本。

```rust
#[derive(Clone, Debug, PartialEq, Eq)]
pub enum ArtifactDestination {
    SendToAllPeers,
}
```



`ReceiveCheckCache` 是一个类型别名。

```rust
pub(crate) type ReceiveCheckCache = LruCache<CryptoHash, ()>;
```

在编程中，类型别名是指为已有的类型创建一个新的名称。这个新的名称与原始的类型是等价的，你可以用这个新名称来声明变量和函数，就像你使用原始的类型一样。

类型别名通常用于使代码更具有可读性和理解性。例如，你可能有一个复杂的数据类型，比如 `HashMap<String, Vec<(String, i32)>>`，每次都写这么长的类型可能会很麻烦，也难以理解。你可以为它创建一个类型别名，比如 `type MyComplexType = HashMap<String, Vec<(String, i32)>>;`，然后你就可以在代码中使用 `MyComplexType` 来代替那个复杂的类型了。



`pub(crate) type ReceiveCheckCache = LruCache<CryptoHash, ()>;` 这一行就定义了一个类型别名。

这里，`ReceiveCheckCache` 是为 `LruCache<CryptoHash, ()>` 类型创建的别名。这样一来，开发者就可以用 `ReceiveCheckCache` 来代替 `LruCache<CryptoHash, ()>`，使得代码更简洁，也更易于理解。

这里定义了一个公开的 ReceiveCheckCache 类型，它利用 LruCache 来实现了一个以 CryptoHash 为键，记录某个哈希是否被访问的 LRU 缓存。它是一个 LRU 缓存，用于检查是否最近接收到了某个特定的 artifact 。

键类型是 artifact 的 `CryptoHash`，这是一个哈希值的类型，可能代表了交易的哈希或账户地址的哈希。

值类型是单元类型 `()` ，也就是一个空元组，表示我们并不关心值，只关心键（也就是是否接收过这个 artifact ）。

pub(crate) 表示这个类型在当前 crate 中是公开的，可以被 crate 中的其他模块使用。

LruCache 是一个泛型结构体，实现了一个固定大小的 LRU 缓存。它会根据访问顺序移除最近最少使用的条目，以保持缓存大小。





`GossipImpl` 结构体是 `GossipMessage` 特征的一个实现。这个结构体有很多字段，包括 artifact 管理器、日志、度量标准、副本 ID 、子网 ID 、注册表客户端、共识池缓存、下载优先级、副本管理器、传输层、通道映射器等。这些字段基本上是为了实现 P2P gossip 功能所需要的所有组件和资源。

```rust
/// The canonical implementation of the `GossipMessage` trait.
pub(crate) struct GossipImpl {
    /// 用来处理接收到的artifact
    pub artifact_manager: Arc<dyn ArtifactManager>,
    /// 用于记录日志
    pub log: ReplicaLogger,
    /// 用于收集和报告度量数据
    pub gossip_metrics: GossipMetrics,

    /// 副本的id
    pub node_id: NodeId,
    /// 子网的id
    pub subnet_id: SubnetId,
    /// 用来和注册表服务进行交互
    pub registry_client: Arc<dyn RegistryClient>,
    /// 共识池的缓存
    pub consensus_pool_cache: Arc<dyn ConsensusPoolCache>,
    /// 下载优先级调度器
    pub prioritizer: Arc<dyn DownloadPrioritizer>,
    /// 当前的副本列表
    pub current_peers: Mutex<PeerContextMap>,
    /// 传输层
    pub transport: Arc<dyn Transport>,
    /// 通道映射器
    pub transport_channel_mapper: TransportChannelIdMapper,
    /// 正在下载的artifact列表
    pub artifacts_under_construction: RwLock<ArtifactDownloadListImpl>,
    /// 下载管理的度量数据
    pub metrics: DownloadManagementMetrics,
    /// gossip 协议的配置
    pub gossip_config: GossipConfig,
    /// 检查 artifact 是否已下载的缓存
    pub receive_check_caches: RwLock<HashMap<NodeId, ReceiveCheckCache>>,
    /// 记录了优先函数调用
    pub pfn_invocation_instant: Mutex<Instant>,
    /// 记录了注册表刷新
    pub registry_refresh_instant: Mutex<Instant>,
    /// 记录了重新传输请求的时间
    pub retransmission_request_instant: Mutex<Instant>,
}
```

每一个成员都是公开的（`pub`），意味着它们可以在包含 `GossipImpl` 的模块或者 crate 中被访问。

`pub artifact_manager: Arc<dyn ArtifactManager>,` 这里使用了 `Arc<dyn ArtifactManager>` 类型，这是 Rust 中一种特殊的类型。`Arc` 是原子引用计数（Atomic Reference Count）的缩写，用于在多线程环境下安全地共享某个值的所有权。`dyn ArtifactManager` 是动态分发的 Trait 对象，它代表了可以被当作 `ArtifactManager` 使用的任何类型的对象。所以 `Arc<dyn ArtifactManager>` 表示的是一个可以在多个线程中共享并且实现了 `ArtifactManager` 特征的对象。

`log` 是 `ReplicaLogger` 类型，`gossip_metrics` 是 `GossipMetrics` 类型。我们可以假设这两者是一些自定义类型，对应了一些特殊的功能，如记录日志和追踪统计信息。





接下来是对 `GossipImpl` 结构体的实现，定义了这个结构体的行为。它同时实现了 `Gossip` 这个特征。

`GossipImpl` 结构体实现的 `new` 函数是它的构造函数，它创建一个新的 `GossipImpl` 实例。这个函数接受一系列参数，包括 `node_id`、`subnet_id` 等，这些都是网络中需要用到的关键参数。在构造函数中，它也创建了一些其他的实例，比如下载优先级管理器（`prioritizer`）、下载管理指标（`metrics`）等，然后将它们和输入的参数一同存储在新创建的 `GossipImpl` 结构体实例中。最后，构造函数调用 `refresh_topology` 函数刷新拓扑信息，然后返回新创建的 `GossipImpl` 实例。

```rust
impl GossipImpl {
    /// The constructor creates a new *Gossip* component.
    ///
    /// The *Gossip* component interacts with the download manager
    /// component, which initiates and tracks downloads of artifacts
    /// from a peer group.
    #[allow(clippy::too_many_arguments)]
    pub fn new(
        node_id: NodeId,
        subnet_id: SubnetId,
        consensus_pool_cache: Arc<dyn ConsensusPoolCache>,
        registry_client: Arc<dyn RegistryClient>,
        artifact_manager: Arc<dyn ArtifactManager>,
        transport: Arc<dyn Transport>,
        transport_channels: Vec<TransportChannelId>,
        log: ReplicaLogger,
        metrics_registry: &MetricsRegistry,
    ) -> Self {
        let prioritizer = Arc::new(DownloadPrioritizerImpl::new(
            artifact_manager.as_ref(),
            DownloadPrioritizerMetrics::new(metrics_registry),
        ));
        let gossip_config = fetch_gossip_config(registry_client.clone(), subnet_id);
        let gossip = GossipImpl {
            artifact_manager,
            log: log.clone(),
            gossip_metrics: GossipMetrics::new(metrics_registry),
            node_id,
            subnet_id,
            consensus_pool_cache,
            prioritizer,
            current_peers: Mutex::new(PeerContextMap::default()),
            registry_client,
            transport,
            transport_channel_mapper: TransportChannelIdMapper::new(transport_channels),
            artifacts_under_construction: RwLock::new(ArtifactDownloadListImpl::new(log)),
            metrics: DownloadManagementMetrics::new(metrics_registry),
            gossip_config,
            receive_check_caches: RwLock::new(HashMap::new()),
            pfn_invocation_instant: Mutex::new(Instant::now()),
            registry_refresh_instant: Mutex::new(Instant::now()),
            retransmission_request_instant: Mutex::new(Instant::now()),
        };
        gossip.refresh_topology();
        gossip
    }
}
```



`GossipImpl` 结构体实现了 `Gossip` 这个特征，这个特征定义了一系列网络上需要进行的操作，比如当接收到新的广告信息（`on_gossip_advert`），当收到对某一部分数据的请求（`on_chunk_request`），当收到某一部分数据（`on_gossip_chunk`）等等。这个特征的实现具体定义了当网络上发生这些事件时，`GossipImpl` 应该如何行动。

例如，`on_gossip_advert` 函数定义了当收到新的广告信息时，`GossipImpl` 首先会检查它是否已经有这个广告中的数据（`artifact`），如果有，它就不会做任何事情。如果没有，它就会处理这个广告，然后尝试去下载这个广告中的数据。

```rust
/// Canonical Implementation for the *Gossip* trait.
impl Gossip for GossipImpl {
    type GossipAdvert = GossipAdvert;
    type GossipChunkRequest = GossipChunkRequest;
    type GossipChunk = GossipChunk;
    type GossipRetransmissionRequest = ArtifactFilter;

    /// The method is called when a new advert is received from the
    /// peer with the given node ID.
    ///
    /// Adverts for artifacts that have been downloaded before are
    /// dropped.  If the artifact is not available locally, the advert
    /// is added to this peer's advert list.
    fn on_gossip_advert(&self, gossip_advert: GossipAdvert, peer_id: NodeId) {
        let _timer = self
            .gossip_metrics
            .op_duration
            .with_label_values(&["in_advert"])
            .start_timer();
        if self
            .artifact_manager
            .has_artifact(&gossip_advert.artifact_id)
        {
            return;
        }

        // The download manager handles the received advert.
        self.on_advert(gossip_advert, peer_id);
        // The next download is triggered for the given peer ID.
        let _ = self.download_next(peer_id);
    }

    /// The method handles the given chunk request received from the peer with
    /// the given node ID.
    fn on_chunk_request(&self, chunk_request: GossipChunkRequest, node_id: NodeId) {
        let _timer = self
            .gossip_metrics
            .op_duration
            .with_label_values(&["in_chunk_request"])
            .start_timer();

        let artifact_chunk = match self
            .artifact_manager
            .get_validated_by_identifier(&chunk_request.artifact_id)
        {
            Some(artifact) => artifact.get_chunk(chunk_request.chunk_id).ok_or_else(|| {
                self.gossip_metrics.requested_chunks_not_found.inc();
                P2PError {
                    p2p_error_code: P2PErrorCode::NotFound,
                }
            }),
            None => {
                self.gossip_metrics.requested_chunks_not_found.inc();
                Err(P2PError {
                    p2p_error_code: P2PErrorCode::NotFound,
                })
            }
        };

        let gossip_chunk = GossipChunk {
            request: chunk_request,
            artifact_chunk,
        };
        let message = GossipMessage::Chunk(gossip_chunk);
        self.transport_send(message, node_id);
    }

    /// The method adds the given chunk to the corresponding artifact
    /// under construction.
    fn on_gossip_chunk(&self, gossip_chunk: GossipChunk, peer_id: NodeId) {
        let _timer = self
            .gossip_metrics
            .op_duration
            .with_label_values(&["in_chunk"])
            .start_timer();
        self.on_chunk(gossip_chunk, peer_id);
        let _ = self.download_next(peer_id);
    }

    /// The method broadcasts the given advert to other peers.
    fn broadcast_advert(&self, advert: GossipAdvert, dst: ArtifactDestination) {
        let _timer = self
            .gossip_metrics
            .op_duration
            .with_label_values(&["out_advert"])
            .start_timer();

        let (peers, label) = match dst {
            ArtifactDestination::SendToAllPeers => (self.get_current_peer_ids(), "all_peers"),
        };
        self.metrics
            .adverts_by_action
            .with_label_values(&[label])
            .inc_by(peers.len() as u64);

        let message = GossipMessage::Advert(advert);
        for peer_id in peers {
            self.transport_send(message.clone(), peer_id);
        }
    }

    /// The method reacts to a retransmission request from another
    /// peer.
    ///
    /// All validated artifacts that pass the given filter are
    /// collected and sent to the peer.
    fn on_gossip_retransmission_request(
        &self,
        gossip_retransmission_request: Self::GossipRetransmissionRequest,
        peer_id: NodeId,
    ) {
        let _timer = self
            .gossip_metrics
            .op_duration
            .with_label_values(&["out_retransmission"])
            .start_timer();
        let _ = self.on_retransmission_request(&gossip_retransmission_request, peer_id);
    }

    fn on_peer_up(&self, peer_id: NodeId) {
        let _timer = self
            .gossip_metrics
            .op_duration
            .with_label_values(&["peer_up"])
            .start_timer();
        info!(self.log, "Peer is up: {:?}", peer_id);
        self.peer_connection_up(peer_id)
    }

    fn on_peer_down(&self, peer_id: NodeId) {
        let _timer = self
            .gossip_metrics
            .op_duration
            .with_label_values(&["peer_down"])
            .start_timer();
        info!(self.log, "Peer is down: {:?}", peer_id);
        self.peer_connection_down(peer_id)
    }

    /// The method is called on a periodic timer event.
    ///
    /// The periodic invocation of this method guarantees IC liveness.
    /// Specifically, the following actions occur on each call:
    ///
    /// - It polls all artifact clients, enabling the IC to make
    /// progress without the need for any external triggers.
    ///
    /// - It checks each peer for request timeouts and advert download
    /// eligibility.
    ///
    /// In short, the method is a catch-all for a periodic and
    /// holistic refresh of IC state.
    fn on_gossip_timer(&self) {
        let _timer = self
            .gossip_metrics
            .op_duration
            .with_label_values(&["timer"])
            .start_timer();
        self.on_timer();
    }
}
```



`fetch_gossip_config` 函数是从注册表中获取 Gossip 配置的函数。它先试着从注册表获取 gossip 配置，如果获取失败，就会使用默认的 gossip 配置。

```rust
fn fetch_gossip_config(
    registry_client: Arc<dyn RegistryClient>,
    subnet_id: SubnetId,
) -> GossipConfig {
    if let Ok(Some(Some(gossip_config))) =
        registry_client.get_gossip_config(subnet_id, registry_client.get_latest_version())
    {
        gossip_config
    } else {
        ic_types::p2p::build_default_gossip_config()
    }
}
```

这个函数接收两个参数，一个是实现了 `RegistryClient` 特征的对象的弧形指针（`Arc<dyn RegistryClient>`），另一个是 `subnet_id`。函数的返回类型是 `GossipConfig` 。

这个函数主要是从注册表中获取 Gossip 的配置信息。具体的操作步骤如下：

1. 首先，它会调用 `registry_client` 的 `get_gossip_config` 方法，尝试从注册表中获取指定子网（`subnet_id`）的 Gossip 配置。这个方法会返回一个 `Result` 类型的值，这个值可能是 `Ok`，也可能是 `Err`。如果是 `Ok`，那么它的值就是 `Some(Some(gossip_config))`，其中 `gossip_config` 就是我们需要的配置信息。
2. 接着，代码使用 `if let` 表达式来尝试匹配 `get_gossip_config` 方法的返回值。如果它的值确实是 `Ok(Some(Some(gossip_config)))`，那么这个函数就会直接返回 `gossip_config`。这就意味着我们成功地从注册表中获取到了 Gossip 配置。
3. 如果 `get_gossip_config` 方法的返回值不是 `Ok(Some(Some(gossip_config)))`，那么 `if let` 表达式的条件就不会成立，代码就会执行 `else` 分支。在 `else` 分支中，函数调用了 `ic_types::p2p::build_default_gossip_config()` 方法来创建一个默认的 Gossip 配置，然后返回这个配置。这就意味着当我们无法从注册表中获取 Gossip 配置时，我们会使用一个默认的配置。


