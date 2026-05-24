# 信标链规范与 API (Beacon Chain Spec and APIs)

以太坊权益证明 (Proof-of-Stake, PoS) 的核心是一个被称为“信标链 (Beacon Chain)”的系统链。信标链存储并管理着验证者注册表 (registry of validators)。在权益证明的初始部署阶段，成为验证者的唯一机制是向以太坊工作量证明 (Proof-of-Work, PoW) 链上的存款合约发起单向（Capella 升级后可提款）的 ETH 交易。当信标链处理了存款收据、达到了激活余额并完成了排队过程时，验证者就会被激活 (activation)。退出 (Exit) 可以是自愿退出 (voluntary exit)，也可以是因不当行为 (misbehavior) 而受到的强制罚没处罚。信标链上的主要负载来源是“见证 (attestations)”。见证既是对分片区块 (shard block) 可用性的投票（在后续升级中推出），同时也是对信标区块的权益证明投票（阶段 0，Phase 0）。

本节将涵盖信标链规范和 API 的一些重要部分。您也可以查看完整的 [信标链规范 (Beacon Chain Spec)](https://github.com/ethereum/consensus-specs/blob/dev/specs/phase0/beacon-chain.md) 和 [信标 API 参考 (Beacon API reference)](https://ethereum.github.io/beacon-APIs/#/)。

信标 API (Beacon API) 是共识层客户端 (Consensus Layer Clients，即信标节点 Beacon Nodes) 提供的 REST 端点 (endpoint)。它是读取共识相关信息的接口，供验证者客户端 (validator clients) 使用。

## 容器 (Containers)

`BeaconState`

```python
class BeaconState(Container):
    # 版本控制 (Versioning)
    genesis_time: uint64
    genesis_validators_root: Root
    slot: Slot
    fork: Fork
    # 历史记录 (History)
    latest_block_header: BeaconBlockHeader
    block_roots: Vector[Root, SLOTS_PER_HISTORICAL_ROOT]
    state_roots: Vector[Root, SLOTS_PER_HISTORICAL_ROOT]
    historical_roots: List[Root, HISTORICAL_ROOTS_LIMIT]
    # Eth1
    eth1_data: Eth1Data
    eth1_data_votes: List[Eth1Data, EPOCHS_PER_ETH1_VOTING_PERIOD * SLOTS_PER_EPOCH]
    eth1_deposit_index: uint64
    # 注册表 (Registry)
    validators: List[Validator, VALIDATOR_REGISTRY_LIMIT]
    balances: List[Gwei, VALIDATOR_REGISTRY_LIMIT]
    # 随机性 (Randomness)
    randao_mixes: Vector[Bytes32, EPOCHS_PER_HISTORICAL_VECTOR]
    # 罚没 (Slashings)
    slashings: Vector[Gwei, EPOCHS_PER_SLASHINGS_VECTOR]  # 每个 epoch 罚没的有效余额总和 (Per-epoch sums of slashed effective balances)
    # 参与度 (Participation)
    previous_epoch_participation: List[ParticipationFlags, VALIDATOR_REGISTRY_LIMIT]
    current_epoch_participation: List[ParticipationFlags, VALIDATOR_REGISTRY_LIMIT]
    # 终局性 (Finality)
    justification_bits: Bitvector[JUSTIFICATION_BITS_LENGTH]  # 为每个近期已合理化的 epoch 设置位 (Bit set for every recent justified epoch)
    previous_justified_checkpoint: Checkpoint
    current_justified_checkpoint: Checkpoint
    finalized_checkpoint: Checkpoint
    # 非活动状态 (Inactivity)
    inactivity_scores: List[uint64, VALIDATOR_REGISTRY_LIMIT]
    # 同步 (Sync)
    current_sync_committee: SyncCommittee
    next_sync_committee: SyncCommittee
    # 执行 (Execution)
    latest_execution_payload_header: ExecutionPayloadHeader
    # 提款 (Withdrawals)
    next_withdrawal_index: WithdrawalIndex
    next_withdrawal_validator_index: ValidatorIndex
    # 自 Capella 升级起生效的深度历史 (Deep history valid from Capella onwards)
    historical_summaries: List[HistoricalSummary, HISTORICAL_ROOTS_LIMIT]
    deposit_requests_start_index: uint64
    deposit_balance_to_consume: Gwei
    exit_balance_to_consume: Gwei
    earliest_exit_epoch: Epoch
    consolidation_balance_to_consume: Gwei
    earliest_consolidation_epoch: Epoch
    pending_deposits: List[PendingDeposit, PENDING_DEPOSITS_LIMIT]
    pending_partial_withdrawals: List[PendingPartialWithdrawal, PENDING_PARTIAL_WITHDRAWALS_LIMIT]
    pending_consolidations: List[PendingConsolidation, PENDING_CONSOLIDATIONS_LIMIT]
```

#### `BeaconBlockBody`

```python
class BeaconBlockBody(Container):
    randao_reveal: BLSSignature
    eth1_data: Eth1Data  # Eth1 数据投票 (Eth1 data vote)
    graffiti: Bytes32  # 任意数据 (Arbitrary data)
    # 操作 (Operations)
    proposer_slashings: List[ProposerSlashing, MAX_PROPOSER_SLASHINGS]
    attester_slashings: List[AttesterSlashing, MAX_ATTESTER_SLASHINGS_ELECTRA]
    attestations: List[Attestation, MAX_ATTESTATIONS_ELECTRA]
    deposits: List[Deposit, MAX_DEPOSITS]
    voluntary_exits: List[SignedVoluntaryExit, MAX_VOLUNTARY_EXITS]
    sync_aggregate: SyncAggregate
    # 执行 (Execution)
    execution_payload: ExecutionPayload
    bls_to_execution_changes: List[SignedBLSToExecutionChange, MAX_BLS_TO_EXECUTION_CHANGES]
    blob_kzg_commitments: List[KZGCommitment, MAX_BLOB_COMMITMENTS_PER_BLOCK]
    execution_requests: ExecutionRequests
```
