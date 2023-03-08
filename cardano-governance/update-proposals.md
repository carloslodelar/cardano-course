---
cover: ../.gitbook/assets/governance.png
coverY: 202
---

# Update proposals

Currently, and until Voltaire era (decentralized governance) is released, Cardano has a **federated governance** mechanism that allows updating the protocol parameters, i.e. adding new features. Such updates are done through and Update Proposal.  Only the holders of the Genesis delegate keys can submit and vote on proposals.

### Cardano Eras

Cardano has come through different **eras**: Byron --> Shelley --> Allegra --> Mary --> Alonzo --> Babbage (current). Each era introduced a new set of functionalities, more eras will come in the future introducing more functionalities.

| Era     | Key features                                                                                        | Consensus                        | Protocol Versions (Major.Minor) | Hardfork Event Name                     |
| ------- | --------------------------------------------------------------------------------------------------- | -------------------------------- | :-----------------------------: | --------------------------------------- |
| Byron   | POS                                                                                                 | <p>Ouroboros classic<br>PBFT</p> |        <p>0.0<br>1.0</p>        | <p>Genesis<br>Byron reboot</p>          |
| Shelley | <p>Decentralization of block production<br>Rewards</p>                                              | TPraos                           |               2.0               | Shelley                                 |
| Allegra | Token locking                                                                                       | TPraos                           |               3.0               | Allegra                                 |
| Mary    | Native assets                                                                                       | TPraos                           |               4.0               | Mary                                    |
| Alonzo  | Smart contracts with Plutus                                                                         | TPraos                           |        <p>5.0<br>6.0</p>        | Alonzo                                  |
| Babbage | <p>PlutusV2<br>Reference inputs<br>Inline datums<br>Reference scripts<br>Removal of d parameter</p> | Praos                            |        <p>7.0<br>8.0</p>        | <p>Vasil<br>secp intra era hardfork</p> |

Transitioning from one era to the next is triggered by an **Update Proposal** that updates the protocol version.

### Update proposals

### Byron era update proposals:

<figure><img src="../.gitbook/assets/upbyron.png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../.gitbook/assets/upshelley.png" alt=""><figcaption></figcaption></figure>

The procedure is as follows:

1. **Register the proposal:** During each epoch, a genesis key can submit (via its delegates) zero, one, or many proposals; each submission overrides the previous one. Proposals can be explicitly marked to be for future epochs; in that case, these are simply not considered until that epoch is reached.
2. **Voting:** The window for Submitting proposals ends 6k/f slots before the end of the epoch. Where k is the security parameter and f is the _active slot coefficient_.
3. **Quorum:** At the end of the epoch, if the majority of nodes (as determined by the **Quorum** specification constant, which must be greater than half the nodes) have most recently submitted the same exact proposal, then it is adopted.
4. **Update:** The update is applied at the epoch boundary.

Changing the values of the protocol parameters and global constants can be done via an Update Proposal.&#x20;

Changing the values of the Global Constants always require a software update, this is includes increasing the protocol version. An increase in the major version indicates a hard fork, and the minor version a soft fork (meaning old software can validate but not produce new blocks).&#x20;

In the other hand, updating Protocol Parameters can be done without a software update.&#x20;

Until Voltaire era is released, only the holders of the **Genesis delegate keys** can submit and vote on proposals. From time to time you will find it useful to deploy a local or private testnet, in that case you will need to use the governance commands to upgrade your network to the desired era.&#x20;
