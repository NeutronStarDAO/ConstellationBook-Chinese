# discovery

这段代码的主要作用是监视拓扑变化并根据注册表信息更新节点状态。主要包括 GossipImpl 结构体的实现，其中包括 `refresh_topology` 和 `merge_subnet_membership` 两个方法，以及一个用于提取节点 Socket 地址的 `get_peer_addr` 函数。同时，还包含了一个测试模块，用于测试这些方法和函数的正确性。

## 1. GossipImpl 结构体的实现

这个实现包含了两个方法：`refresh_topology` 和 `merge_subnet_membership`。

#### 1.1 `refresh_topology` 方法

**功能**：根据最新的注册表值更新 peer manager 状态。这个方法可以根据最新的注册表信息更新节点状态，确保节点之间的连接始终保持最新。

**代码解析**：

1. 获取最新的注册表版本。
2. 合并子网成员。
3. 检查当前节点是否在子网中。
4. 移除不在子网内的节点。
5. 如果当前节点不在子网内，则退出以避免将节点添加到节点列表中。
6. 将节点添加到 peer manager。







整个代码片段的作用是监控拓扑变化，通过轮询注册表来实现。当子网中的节点发生添加或移除时，执行相应的 add_peer 和 remove_peer 调用。实现了拓扑变化的监控，能够根据注册表的变化动态更新 peer manager 的状态。

整个代码分为以下几个部分：

1. 模块注释和引用
2. GossipImpl 结构体的实现
3. get_peer_addr 函数
4. tests 模块

## 1. 模块注释和引用

```rust
//! The module is responsible for watching for topology changes by polling the registry.
//! If nodes are added or removed from the subnet of the current node, then the appropriate
//! 'add_peer' and 'remove_peer' calls are executed.

use crate::gossip_protocol::GossipImpl;
use ic_logger::error;
use ic_protobuf::registry::node::v1::NodeRecord;
use ic_registry_client_helpers::subnet::SubnetTransportRegistry;
use ic_types::{NodeId, RegistryVersion};
use std::{
    collections::BTreeMap,
    net::{IpAddr, SocketAddr},
    str::FromStr,
};
```

这部分代码主要是模块注释和引用。模块注释描述了该模块的功能：监控拓扑变化，通过轮询注册表来实现。当子网中的节点发生添加或移除时，执行相应的 add_peer 和 remove_peer 调用。

## 2. GossipImpl 结构体的实现

```rust
impl GossipImpl {
    // Update the peer manager state based on the latest registry value.
    pub(crate) fn refresh_topology(&self) {
        ...
    }

    // Merge node records from subnet_membership_version (provided by consensus)
    // to latest_registry_version. This returns the current subnet membership set.
    fn merge_subnet_membership(
        &self,
        latest_registry_version: RegistryVersion,
    ) -> BTreeMap<NodeId, NodeRecord> {
        ...
    }
}
```

这部分是 GossipImpl 结构体的实现。它包含了两个方法：`refresh_topology` 和 `merge_subnet_membership`。

`refresh_topology` 方法的作用是根据最新的注册表值更新 peer manager 的状态。它首先获取最新的注册表版本，然后合并来自子网成员关系版本的节点记录，然后根据这些记录更新 peer manager 的状态。

```rust
pub(crate) fn refresh_topology(&self) {
        let latest_registry_version = self.registry_client.get_latest_version();
        self.metrics
            .registry_version_used
            .set(latest_registry_version.get() as i64);

        let subnet_nodes = self.merge_subnet_membership(latest_registry_version);
        let self_not_in_subnet = !subnet_nodes.contains_key(&self.node_id);

        // If a peer is not in the nodes within this subnet, remove.
        // If self is not in the subnet, remove all peers.
        for peer_id in self.get_current_peer_ids().iter() {
            if !subnet_nodes.contains_key(peer_id) || self_not_in_subnet {
                self.remove_peer(peer_id);
            }
        }
        // If self is not subnet, exit early to avoid adding nodes to the list of peers.
        if self_not_in_subnet {
            return;
        }
        // Add in nodes to peer manager.
        for (node_id, node_record) in subnet_nodes.iter() {
            match get_peer_addr(node_record) {
                None => {
                    // Invalid socket addresses should not be pushed in the registry/config on first place.
                    error!(self.log, "Invalid socket addr: node_id = {:?}", *node_id);
                    // If getting the peer socket fails, remove the node. This removal makes it possible
                    // to attempt a re-addition on the next refresh cycle.
                    self.remove_peer(node_id)
                }
                Some(peer_addr) => self.add_peer(*node_id, peer_addr, latest_registry_version),
            }
        }
    }
```

`merge_subnet_membership` 方法的作用是合并从子网成员关系版本（由共识提供）到最新注册表版本的节点记录。它返回当前子网成员关系集合。

合并从 `subnet_membership_version`（由共识提供）到 `latest_registry_version` 的节点记录。返回当前子网成员集。这个方法可以合并不同版本的子网成员信息，确保子网成员信息始终保持最新。

1. 创建一个空的 BTreeMap 用于存储子网节点。
2. 遍历从最小的注册表版本到最大的注册表版本的所有版本。
3. 获取特定版本下的节点记录，并将它们插入到子网节点 BTreeMap 中。

```rust
fn merge_subnet_membership(
    &self,
    latest_registry_version: RegistryVersion,
) -> BTreeMap<NodeId, NodeRecord> {
    let subnet_membership_version = self
        .consensus_pool_cache
        .get_oldest_registry_version_in_use();
    let mut subnet_nodes = BTreeMap::new();
    // Iterate from min(consensus_registry_version,latest_local_registry_version) to max(consensus_registry_version,latest_local_registry_version).
    // The `consensus_registry_version` is extracted from the latest CUP seen.
    // The `latest_local_registry_version` is the latest registry version known to this node.
    // In almost any case `latest_local_registry_version >= consensus_registry_version` but there may exist cases where this condition does not hold.
    // In that case we should at least include our latest local view of the subnet.
    for version in subnet_membership_version
        .get()
        .min(latest_registry_version.get())
        ..=subnet_membership_version
            .get()
            .max(latest_registry_version.get())
    {
        let version = RegistryVersion::from(version);
        let node_records = self
            .registry_client
            .get_subnet_transport_infos(self.subnet_id, version)
            .unwrap_or(None)
            .unwrap_or_default();
        for node in node_records {
            subnet_nodes.insert(node.0, node.1);
        }
    }
    subnet_nodes
}
```



## 3. get_peer_addr 函数

从给定的节点记录中获取节点的 Socket 地址。

遍历节点记录的 `p2p_flow_endpoints`，从中提取 IP 地址和端口号，然后创建一个 SocketAddr 对象。

```rust
fn get_peer_addr(node_record: &NodeRecord) -> Option<SocketAddr> {
    ...
}
```

`get_peer_addr` 函数接受一个 `NodeRecord` 引用，并返回该节点的 `SocketAddr`。如果无法从节点记录中获取有效的 `SocketAddr`，则返回 `None`。



## 4. tests 模块

```rust
#[cfg(test)]
mod tests {
    ...
}
```

`tests` 模块包含了针对上述函数的测试代码。测试代码主要包括两个测试函数：`test_get_peer_addr` 和 `test_merge_subnet_membership`。这些测试用于验证上述函数的正确性。

