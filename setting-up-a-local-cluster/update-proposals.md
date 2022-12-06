# Update proposals

### Cardano Eras

Cardano has come through different **eras**: Byron --> Shelley --> Allegra --> Mary --> Alonzo --> Babbage (current). Each era introduced a new set of functionalities, more eras will come in the future introducing more functionalities.

| Era     | Key features                                                                                        | Consensus                        | Protocol Versions (Major.Minor) | Hardfork Event Name            |
| ------- | --------------------------------------------------------------------------------------------------- | -------------------------------- | :-----------------------------: | ------------------------------ |
| Byron   | POS                                                                                                 | <p>Ouroboros classic<br>PBFT</p> |        <p>0.0<br>1.0</p>        | <p>Genesis<br>Byron reboot</p> |
| Shelley | <p>Decentralization of block production<br>Rewards</p>                                              | TPraos                           |               2.0               | Shelley                        |
| Allegra | Token locking                                                                                       | TPraos                           |               3.0               | Allegra                        |
| Mary    | Native assets                                                                                       | TPraos                           |               4.0               | Mary                           |
| Alonzo  | Smart contracts with Plutus                                                                         | TPraos                           |        <p>5.0<br>6.0</p>        | Alonzo                         |
| Babbage | <p>PlutusV2<br>Reference inputs<br>Inline datums<br>Reference scripts<br>Removal of d parameter</p> | Praos                            |               7.0               | Vasil                          |

Transitioning from one era to the next is triggered by an **Update Proposal** that updates the protocol version.

### Update proposals

### Byron era update proposals:

<figure><img src="../.gitbook/assets/Screen Shot 2022-12-06 at 9.57.28.png" alt=""><figcaption></figcaption></figure>

The general mechanism for updating protocol parameters in Byron is as follows:

1. **Update proposal is Registered:** an update proposal starts with a transaction that proposes new values for some protocol parameters or a new protocol version. This kind of transaction can only be initiated by a Genesis key via its Delegate
2. **Accumulating votes:** Genesis key delegates **vote** for or against the proposal. The proposal must accumulate a sufficient number of votes before it can be confirmed. The threshold is determined by **minThd** field of the [softforkRule protocol parameter](https://github.com/input-output-hk/cardano-ledger/blob/2a0abd500b9e01efe6dc47146fa8b805ef9ef307/eras/byron/ledger/impl/src/Cardano/Chain/Update/SoftforkRule.hs#L24)
3. **Confirmed (enough votes):** The system records the 'SlotNo' of the slot in which the required threshold of votes was met. At this point 2k slots (2 times the security parameter k) need to pass before the update is stably confirmed and can be _endorsed_. Endorsements for proposals that are not yet stably-confirmed are not invalid but rather silently ignored.
4. **Stably-confirmed:** The last required vote is 2k slots deep. Ready to accumulate endorsements. A block whose header's protocol version number is that of the proposal is interpreted as an **endorsement**. In other words, the nodes are ready for the upgrade. Once the number of endorsers satisfies a threshold (same as for voting), the confirmed proposal becomes a **candidate proposal**.
5. **Candidate:** Enough nodes have endorsed the proposal. At this point a further 2k slots need to pass before the update becomes a stable candidate and can be adopted.
6. **Stable candidate:** The last required endorsement is 2k slots deep.

If there was no stable candidate proposal, then nothing happens. Everything is retained; in particular, a candidate proposal whose threshold-satisfying endorsement was not yet stable will be adopted at the subsequent epoch unless it is surpassed in the meantime.

A "null" update proposal is one that neither increase the protocol version nor the software version. Such update proposals are invalid according to the Byron specification.

### Update proposals in Shelley and subsequent eras:

The update mechanism in Shelley is simpler than it is in Byron. There is no distinction between votes and proposals: to \`\`vote'' for a proposal, a Genesis delegate submits the exact same proposal on a regular transaction signed by the delegate key. There is also no separate endorsement step.

The procedure is as follows:

1. **Register the proposal:** During each epoch, a genesis key can submit (via its delegates) zero, one, or many proposals; each submission overrides the previous one. Proposals can be explicitly marked to be for future epochs; in that case, these are simply not considered until that epoch is reached.
2. **Voting:** The window for Submitting proposals ends 6k/f slots before the end of the epoch. Where f is the _active slot coefficient_.
3. **Quorum:** At the end of the epoch, if the majority of nodes (as determined by the **Quorum** specification constant, which must be greater than half the nodes) have most recently submitted the same exact proposal, then it is adopted.
4. **Update:** The update is applied at the epoch boundary.

<figure><img src="../.gitbook/assets/Screen Shot 2022-12-06 at 10.06.41.png" alt=""><figcaption></figcaption></figure>

### Update proposals in practice

Until Voltaire era is released, only the holders of the **Genesis delegate keys** can submit and vote on proposals. From time to time you will find it usefull to deploy a local or private testnet, in that case you will need to use the governance commands to upgrade your networl to the desired era. Here is how you do it:

#### Update proposal in Byron era

We use **cardano-cli** to create, submit and vote the update proposal

```bash
cardano-cli byron governance
Usage: cardano-cli byron governance COMMAND

  Byron governance commands

Available commands:
  create-update-proposal   Create an update proposal.
  create-proposal-vote     Create an update proposal vote.
  submit-update-proposal   Submit an update proposal.
  submit-proposal-vote     Submit a proposal vote.
```

For example, to go from Byron OBFT (protocol version 1.0.0) to Shelley era (protocol version 2.0) we do:

Create the update proposal

```bash
cardano-cli byron governance create-update-proposal \
--filepath update.v2.proposal \
--testnet-magic 42 \
--signing-key bft0/byron.000.key \
--protocol-version-major "2" \
--protocol-version-minor "0" \
--protocol-version-alt "0" \
--application-name "cardano-sl" \
--software-version-num "1" \
--system-tag "linux" \
--installer-hash 0
```

Where `byron.000.key` is the Genesis delegate signing key (of a BFT node). The proposal is saved in the file `updateprotov1.proposal`which we need to submit to the chain.

Submit the proposal&#x20;

```bash
cardano-cli byron submit-update-proposal \
--testnet-magic 42 \
--filepath update.v2.proposal
```

When the proposal is submitted, the nodes logs will show that the proposal is registered

```bash
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate = 
ByronUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateVersion = 2.0.0, 
protocolUpdateState = UpdateRegistered (SlotNo 931)}]}))
```

It's time for the delegates to vote

```bash
cardano-cli byron governance create-proposal-vote \
--proposal-filepath update.v2.proposal \
--testnet-magic 42 \
--signing-key bft1/byron.001.key \
--vote-yes \
--output-filepath update.v2.vote
```

```bash
cardano-cli byron submit-proposal-vote  \
--testnet-magic 42 \
--filepath update.v2.vote
```

When enough votes support the proposal, the nodes logs show that the proposal is confirmed.

```bash
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate =
 ByronUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateVersion = 2.0.0, 
 protocolUpdateState = UpdateConfirmed (SlotNo 938)}]}))
```

Once the last vote is 2k slots deep the proposal is stably confirmed

```bash
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate = 
ByronUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateVersion = 2.0.0, 
protocolUpdateState = UpdateStablyConfirmed (fromList [])}]}))
```

Nodes can start endorsing the proposal by updating their nodes to a shelley compatible version, updating the configuration file and restarting their nodes:

```bash
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate =
 ByronUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateVersion = 2.0.0, 
 protocolUpdateState = UpdateCandidate (SlotNo 1028) (EpochNo 3)}]}))
```

When the last required endorsement is 2k slots deep, it becomes an stable candidate

```bash
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate =
 ByronUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateVersion = 2.0.0, 
 protocolUpdateState = UpdateStableCandidate (EpochNo 3)}]}))
```

At the epoch transition we should se the hardfork happening:

```bash
Event: LedgerUpdate (HardForkUpdateTransitionDone <EraIndex Byron> <EraIndex Shelley> (EpochNo 3))
```

#### Update proposal in Shelley and subsequent eras

In Shelley and following eras we use a different command:

```bash
cardano-cli governance create-update-proposal
Usage: cardano-cli governance create-update-proposal --out-file FILE
            --epoch NATURAL
            (--genesis-verification-key-file FILE)
            [--protocol-major-version NATURAL --protocol-minor-version NATURAL]
            [--decentralization-parameter RATIONAL]
            [--extra-entropy HEX | --reset-extra-entropy]
            [--max-block-header-size NATURAL]
            [--max-block-body-size NATURAL]
            [--max-tx-size NATURAL]
            [--min-fee-constant LOVELACE]
            [--min-fee-linear NATURAL]
            [--min-utxo-value NATURAL]
            [--key-reg-deposit-amt NATURAL]
            [--pool-reg-deposit NATURAL]
            [--min-pool-cost NATURAL]
            [--pool-retirement-epoch-boundary INT]
            [--number-of-pools NATURAL]
            [--pool-influence RATIONAL]
            [--monetary-expansion RATIONAL]
            [--treasury-expansion RATIONAL]
            [--utxo-cost-per-word LOVELACE]
            [--price-execution-steps RATIONAL --price-execution-memory RATIONAL]
            [--max-tx-execution-units (INT, INT)]
            [--max-block-execution-units (INT, INT)]
            [--max-value-size INT]
            [--collateral-percent INT]
            [--max-collateral-inputs INT]
            [--utxo-cost-per-byte LOVELACE]
            [--cost-model-file FILE]
```

It is possible to update parameters without changing the protocol version. In this case, of course, the update will not trigger a transition into a new era.

Let's take for example a dummy chain that is already on Shelley and running TPraos consensus with d parameter = 1. To submit an update proposal that brings the decentralization parameter down to .80 we first:

create the proposal using the shelley genesis non extended verification keys:

```bash
cardano-cli governance create-update-proposal \
--out-file update.D80.proposal \
--epoch $(cardano-cli query tip --testnet-magic 42 | jq .epoch) \
--genesis-verification-key-file genesis-keys/non.e.shelley.000.vkey \
--genesis-verification-key-file genesis-keys/non.e.shelley.001.vkey \
--decentralization-parameter 80/100
```

For the proposal to be adopted, it must be on chain 6k/f slots before the end of the epoch. In other words, it needs to be submitted by the required quorum during the first 4k/f slots of an epoch. On mainnet we have `k=2160` `f=0.05` so `4k/f = 172,800 slots`. Therefore, update proposals on mainnet need to be placed on-chain during the first 172,800 slots (48 hours) of an epoch.

If the current tip is within the first 4k/f slots of the epoch, we can set `--epoch <CURRENT EPOCH>` on the update propsal, otherwise we need to use `--epoch <CURRENT EPOCH + 1>`.

If we submit the proposal after 4k/f slots we get an error, unless it is for a future epoch.

Unlike in Byron, Shelley update proposals are submitted with a regular transaction, therefore we need to build the transaction body, sign it, pay fees, etc.

To keep it simple we will pay 1 ADA fee, and set a time to live of 1000 slots:

```bash
CHANGE=$(($(cardano-cli query utxo --address $(cat payment.addr) \
--testnet-magic 42 --out-file  /dev/stdout | jq -cs '.[0] | to_entries | .[] | .value.value') - 1000000))

cardano-cli transaction build-raw \
--shelley-era \
--fee 1000000 \
--invalid-hereafter $(expr $(cardano-cli query tip --testnet-magic 42 | jq .slot) + 1000) \
--tx-in $(cardano-cli query utxo --address $(cat payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -r 'keys[]') \
--tx-out $(cat payment.addr)+$CHANGE \
--update-proposal-file update.D80.proposal \
--out-file update.D80.proposal.txbody
```

We sign with the 2 keys of our dummy blockchain in one go. In a real life situation Genesis delegates might sign and submit an identical proposal in separate transactions.

```bash
cardano-cli transaction sign \
--tx-body-file update.D80.proposal.txbody \
--signing-key-file payment.skey \
--signing-key-file bft0/shelley.000.skey \
--signing-key-file bft1/shelley.001.skey \
--out-file update.D80.proposal.txsigned

cardano-cli transaction submit --testnet-magic 42 --tx-file update.D80.proposal.txsigned
```

Shelley update proposals are shown differently in the logs, the proposal, and the notice of quorum reached.

```bash
Event: LedgerUpdate (HardForkUpdateInEra S (Z (WrapLedgerUpdate {unwrapLedgerUpdate =
 ShelleyUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateProposal = UpdateProposal
 {proposalParams = PParams {_minfeeA = SNothing, _minfeeB = SNothing, _maxBBSize = SNothing,
 _maxTxSize = SNothing, _maxBHSize = SNothing, _keyDeposit = SNothing, _poolDeposit = SNothing,
 _eMax = SNothing, _nOpt = SNothing, _a0 = SNothing, _rho = SNothing, _tau = SNothing, _d = SJust
 (4 % 5), _extraEntropy = SNothing, _protocolVersion = SNothing, _minUTxOValue = SNothing,
 _minPoolCost = SNothing}, proposalVersion = Nothing, proposalEpoch = EpochNo 27},
 protocolUpdateState = UpdateState {proposalVotes = [KeyHash
 "5feef96101e0386c56f7661e46438642156f8d1af7e8fec92b1a402a",KeyHash "b8b358d5093f6d1e3a5128a36fe70031c15b7d57817e44f7460142a0"], proposalReachedQuorum = True}}]})))
```

The adoption of the proposal:

```bash
Event: LedgerUpdate (HardForkUpdateInEra S (Z (WrapLedgerUpdate {unwrapLedgerUpdate =
 ShelleyUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateProposal = UpdateProposal
 {proposalParams = PParams {_minfeeA = SNothing, _minfeeB = SNothing, _maxBBSize = SNothing,
 _maxTxSize = SNothing, _maxBHSize = SNothing, _keyDeposit = SNothing, _poolDeposit = SNothing,
 _eMax = SNothing, _nOpt = SNothing, _a0 = SNothing, _rho = SNothing, _tau = SNothing, _d = SJust
 (4 % 5), _extraEntropy = SNothing, _protocolVersion = SNothing, _minUTxOValue = SNothing,
 _minPoolCost = SNothing}, proposalVersion = Nothing, proposalEpoch = EpochNo 27},
 protocolUpdateState = UpdateState {proposalVotes = [KeyHash
 "5feef96101e0386c56f7661e46438642156f8d1af7e8fec92b1a402a",KeyHash "b8b358d5093f6d1e3a5128a36fe70031c15b7d57817e44f7460142a0"], proposalReachedQuorum = True}}]})))
```

#### Going from Shelley to Babbage

Updating the chain to a new protocol version of course, requires that we run a node that is compatible with the era we want to get into. The latest version "knows" about all eras from Byron to Babbage.
