## 随机信标 random_beacon_maker

BLS 阈值签名

（ToDo：画一个树状图显示函数之间的调用关系和功能）



### on_state_change

如果符合提议信标共享的要求，就提议。

其中包含一个名为 `on_state_change` 的函数。这个函数接受一个类型为 `PoolReader` 的参数 pool，返回一个 `Option<RandomBeaconShare>` 类型的值。这个函数的作用是获取当前节点的信息以及池中的信息，并根据这些信息判断当前节点是否需要生成随机信标。如果需要生成随机信标，则生成该信标并将其封装在 `RandomBeaconShare` 结构体中返回。

函数首先获取当前节点的 ID 和当前高度，然后从池中获取高度对应的随机信标。接下来，它获取下一个高度并尝试从池中获取该高度对应的随机信标。如果池中没有这个随机信标，并且当前节点属于该高度的随机信标委员会，则生成该随机信标并返回。

生成随机信标的过程涉及到创建一个 `RandomBeaconContent` 结构体，该结构体包含了高度和哈希值。然后，函数会调用一个名为 `active_low_threshold_transcript` 的函数获取当前高度的转录。如果能够成功获取转录，则调用 crypto.sign 方法对 `RandomBeaconContent` 结构体进行签名。签名成功后，函数将签名信息封装在 `RandomBeaconShare` 结构体中返回。如果任何一个步骤出现问题，则函数将返回 `None` 。

```rust
pub fn on_state_change(&self, pool: &PoolReader<'_>) -> Option<RandomBeaconShare> {
    trace!(self.log, "on_state_change");
    let my_node_id = self.replica_config.node_id;
    let height = pool.get_notarized_height(); // 获取已公证过区块的高度
    let beacon = pool.get_random_beacon(height)?; // 获取已公证区块高度的随机信标，即上一轮的随机信标
    let next_height = height.increment(); // 就是当前轮次的高度（大家正在出块签名，没有敲定区块，所以叫“下一个块”）
    let next_beacon = pool.get_random_beacon(next_height); // 获取下一个区块高度的随机信标
    match self.membership.node_belongs_to_threshold_committee(
        my_node_id,
        next_height,
        RandomBeacon::committee(),
    ) {
        Err(MembershipError::RegistryClientError(_)) => None,
        Err(MembershipError::NodeNotFound(_)) => {
            panic!("This node does not belong to this subnet")
        }
        Err(MembershipError::UnableToRetrieveDkgSummary(h)) => {
            error!(
                self.log,
                "Couldn't find transcript at height {} with finalized height {} and CUP height {}",
                h,
                pool.get_finalized_height(),
                pool.get_catch_up_height()
            );
            None
        }
        Ok(is_beacon_maker)
        // 这段代码中使用了一个match表达式，它匹配一个包含两个元素的元组。
        // 第一个元素是一个布尔值，表示是否需要为下一个高度创建新的随机信标；
        // 第二个元素是下一个高度。
        //
        // 如果第一个元素为true，则执行第一分支，否则执行第二分支。
        //
        // 在第一个分支中，如果当前节点是信标制造者，并且下一个高度没有生成随机信标，
        // 并且当前节点没有为下一个高度获取到随机信标的分享，那就创造随机信标。
        // 在第二个分支中，返回None。
            if is_beacon_maker
                && next_beacon.is_none()
                && !pool
                    .get_random_beacon_shares(next_height)
                    .any(|s| s.signature.signer == my_node_id) =>
        {
            let content =
            // 将上一轮的随机信标哈希一下
            RandomBeaconContent::new(next_height, ic_types::crypto::crypto_hash(&beacon));
            // 涉及到一个问题，即是否适合使用起始块 h 的 dkg_id 来生成高度为 h 
            // 的随机信标。之所以这是可能的，是因为我们实际上只有在高度为 h 
            // 的区块存在后才生成该高度的随机信标，并且我们只在验证高度为 h 的区
            // 块时使用高度为 h-1 的随机信标。因此，使用起始块的 dkg_id 来
            // 生成随机信标不会影响到区块的有效性和正确性，因为实际上是在区块生成
            // 后才使用随机信标，以决定下一轮的副本出块排名。
            if let Some(transcript) =
                active_low_threshold_transcript(pool.as_cache(), next_height)
            {
                match self.crypto.sign(&content, my_node_id, transcript.dkg_id) {
                    Ok(signature) => Some(RandomBeaconShare { content, signature }),
                    Err(err) => {
                        error!(self.log, "Couldn't create a signature: {:?}", err);
                        None
                    }
                }
            } else {
                error!(
                    self.log,
                    "Couldn't find the transcript at height {}", height
                );
                None
            }
        }
        _ => None,
    }
}
```



### get_notarized_height

它没有参数。该函数返回一个 `Height` 值。

该函数的目的是获取已公证块的最大高度。以下是该函数的实现：

首先，使用 `get_catch_up_height` 方法获取尚未被处理的最高的抓取包高度，并将其存储在 `catch_up_height` 变量中。

然后，从池中提取所有已验证的公证，并使用 `max_height` 方法获取已公证块的最大高度。如果没有找到已公证块，则返回 `catch_up_height`。

最后，从 `catch_up_height` 和已公证块的最大高度中选择较大的值，并将其作为结果返回。

```rust
pub fn get_notarized_height(&self) -> Height {
    let catch_up_height = self.get_catch_up_height();
    self.pool
        .validated()
        .notarization()
        .max_height()
        .unwrap_or(catch_up_height)
        .max(catch_up_height)
}
```



### get_random_beacon

它有两个参数：`self`，它是一个对象实例的引用，和 `height`，它是一个 `Height` 值。该函数返回一个 `Option<RandomBeacon>`。

该函数的目的是在给定高度处获取一个有效的随机信标。以下是该函数的实现：

首先，使用 `cmp` 方法比较给定高度与 `get_catch_up_height` 方法返回的高度。`get_catch_up_height` 返回的是尚未被处理的最高的抓取包高度。

如果给定高度小于 `get_catch_up_height` 返回的高度，则说明该高度的随机信标尚未产生，返回 `None`。

如果给定高度等于 `get_catch_up_height` 返回的高度，则说明该高度的随机信标已经被包含在最高的抓取包中，因此从抓取包中提取随机信标并用 `Some` 包装返回。

如果给定高度大于 `get_catch_up_height` 返回的高度，则说明该高度的随机信标已经在池中验证通过。因此，从池中提取验证通过的随机信标，然后使用 `get_only_by_height` 方法，通过给定高度获取该随机信标。如果可以获取，则返回它，否则返回 `None`。

```rust
pub fn get_random_beacon(&self, height: Height) -> Option<RandomBeacon> {
    match height.cmp(&self.get_catch_up_height()) {
        Ordering::Less => None,
        Ordering::Equal => Some(
            self.get_highest_catch_up_package()
                .content
                .random_beacon
                .as_ref()
                .clone(),
        ),
        Ordering::Greater => self
            .pool
            .validated()
            .random_beacon()
            .get_only_by_height(height)
            .ok(),
    }
}
```



### active_low_threshold_transcript

它有两个参数：reader ，它是一个实现了 `ConsensusPoolCache trait` 的对象引用，和 height，它是一个 Height 值。该函数返回一个 `Option<NiDkgTranscript>` 。

该函数的目的是为了获取给定高度的活动低阈值 NiDkg 记录。调用了 `get_active_data_at` 函数，并传入 reader 和 height 作为参数，函数返回值被映射，以提取返回的数据对象的 `low_threshold_transcript` 字段。

如果 `get_active_data_at` 函数返回 None，则整个表达式的值为 None 。否则，将提取 `low_threshold_transcript` 字段，并用 Option 包装返回。

```rust
/// 如果找到，就返回给定高度的当前低转录本。
pub fn active_low_threshold_transcript(
    reader: &dyn ConsensusPoolCache,
    height: Height,
) -> Option<NiDkgTranscript> {
    get_active_data_at(reader, height).map(|data| data.low_threshold_transcript)
}
```



### get_active_data_at

如果找到了给定高度处的活跃 DKGData ，则返回该 DKGData 。

```rust
fn get_active_data_at(reader: &dyn ConsensusPoolCache, height: Height) -> Option<DkgData> {
    // 在确定活动的DKG数据时需要注意的问题。当使用最新的finalized DKG summary时，
    // 存在一个问题，即当批处理滞后于共识时，会出现无法计算下一个块或CUP的情况。这是因为只有当
    // 最终块到达特定高度且指向至少该高度的认证状态时，我们才能创建CUP。为了解决这个问题，注释
    // 提出了一种解决方案：首先尝试使用CUP的汇总块来确定活动的DKG数据，如果失败，则尝试使用最
    // 新的finalized DKG summary块。这样可以避免出现无法计算下一个块或CUP的情况。
    get_active_data_at_given_summary(reader.catch_up_package().content.block.get_value(), height)
        .or_else(|| get_active_data_at_given_summary(&reader.summary_block(), height))
}
```



### get_active_data_at_given_summary

根据给定的总结块（summary block）返回给定高度的活动 DKGData（DKG 数据）。总结块是一个固定高度的区块，其中包含了一些元数据，
例如用于验证下一个一致性证明（consensus proof）的公钥、DKG 数据的摘要等。总结块的高度通常是一些定期间隔的倍数，例如每 100 个区块一个总结块。

总结块包含了 DKGDatas 的摘要，其中包括了一个叫做 active_data 的字段，它指向了当前应该使用的 DKGData。这个函数会先检查给定总结块是否存在 active_data 字段并且是否与给定高度匹配，如果匹配则返回 active_data ，否则返回 None 。

```rust
fn get_active_data_at_given_summary(summary_block: &Block, height: Height) -> Option<DkgData> {
    let dkg_summary = &summary_block.payload.as_ref().as_summary().dkg;
    if dkg_summary.current_interval_includes(height) {
        Some(DkgData {
            registry_version: dkg_summary.registry_version,
            high_threshold_transcript: dkg_summary
                .current_transcript(&NiDkgTag::HighThreshold)
                .clone(),
            low_threshold_transcript: dkg_summary
                .current_transcript(&NiDkgTag::LowThreshold)
                .clone(),
        })
    } else if dkg_summary.next_interval_includes(height) {
        let get_transcript_for = |tag| {
            dkg_summary
                .next_transcript(&tag)
                .unwrap_or_else(|| dkg_summary.current_transcript(&tag))
                .clone()
        };
        Some(DkgData {
            registry_version: summary_block.context.registry_version,
            high_threshold_transcript: get_transcript_for(NiDkgTag::HighThreshold),
            low_threshold_transcript: get_transcript_for(NiDkgTag::LowThreshold),
        })
    } else {
        None
    }
}
```





### aggregate

其他副本对共享随机信标签名：

```rust
/// 这个函数用于将签名分享聚合成完整的工件。
/// 
/// Consensus 在其生命周期内会收到许多工件签名分享。
/// aggregate 尝试将这些签名分享聚合成相应内容的完整签名。
/// 
/// 例如，aggregate 可以获取一个 Vec<&RandomBeaconShare>，并输出一个 Vec<&RandomBeacon>。
/// 
/// aggregate 的行为如下：
/// 
/// 按内容对签名分享进行分组，生成从内容到签名分享向量的映射，每个签名分享都签署了相关内容
/// 对于每个分组（内容，分享）对，查找阈值并确定是否有足够的分享来为内容构造完整的签名
/// 如果可以构造完整的签名，则尝试从分享中构造完整的签名
/// 如果可以构造完整的签名，则使用给定的内容和完整的签名构造工件
/// 返回所有成功构造的工件
/// 
/// 参数：
/// artifact_shares - 工件分享向量，例如 Vec<&RandomBeaconShare>
#[allow(clippy::type_complexity)]
pub fn aggregate<
    Message: Eq + Ord + Clone + std::fmt::Debug + HasHeight + HasCommittee,
    CryptoMessage,
    Signature: Ord,
    KeySelector: Copy,
    CommitteeSignature,
    Shares: Iterator<Item = Signed<Message, Signature>>,
>(
    log: &ReplicaLogger,
    membership: &Membership,
    crypto: &dyn Aggregate<CryptoMessage, Signature, KeySelector, CommitteeSignature>,
    selector: Box<dyn Fn(&Message) -> Option<KeySelector> + '_>,
    artifact_shares: Shares,
) -> Vec<Signed<Message, CommitteeSignature>> {
    group_shares(artifact_shares)
        .into_iter()
        .filter_map(|(content_ref, shares)| {
            let selector = selector(&content_ref).or_else(|| {
                warn!(
                    log,
                    "aggregate: cannot find selector for content {:?}", content_ref
                );
                None
            })?;
            let threshold = match membership
                .get_committee_threshold(content_ref.height(), Message::committee())
            {
                Ok(threshold) => threshold,
                Err(err) => {
                    error!(log, "MembershipError: {:?}", err);
                    return None;
                }
            };
            if shares.len() < threshold {
                return None;
            }
            let shares_ref = shares.iter().collect();
            crypto
                .aggregate(shares_ref, selector)
                .ok()
                .map(|signature| {
                    let content = content_ref.clone();
                    Signed { content, signature }
                })
        })
        .collect()
}
```



