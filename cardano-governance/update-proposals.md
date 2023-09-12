---
cover: ../.gitbook/assets/governance.png
coverY: 202
---

# Update proposals

Currently, and until the release of the Voltaire era (decentralized governance), Cardano operates under a **federated governance** mechanism that allows updating protocol parameters, including the addition of new features. These updates are carried out through an update proposal process. Only the holders of the genesis delegate keys have the authority to submit and vote on proposals.

## Cardano ledger eras

Cardano has transitioned through different ledger eras: Byron, Shelley, Allegra, Mary, Alonzo, and the current era, Babbage. Each of these eras has brought forth a new set of functionalities, and there are more eras planned for the future, each introducing additional functionalities.

| Era     | Key features                                                                                        | Consensus                        | Protocol versions (major/minor) | Hard fork event               |
| ------- | --------------------------------------------------------------------------------------------------- | -------------------------------- | :-----------------------------: | -------------------------------- |
| Byron   | Proof of stake                                                                                                 | <p>Ouroboros Classic<br>PBFT</p> |        <p>0.0<br>1.0</p>        | <p>Genesis<br>Byron reboot</p>   |
| Shelley | <p>Decentralized block production<br>Rewards</p>                                              | TPraos                           |               2.0               | Shelley                          |
| Allegra | Token locking                                                                                       | TPraos                           |               3.0               | Allegra                          |
| Mary    | Native tokens                                                                                       | TPraos                           |               4.0               | Mary                             |
| Alonzo  | Plutus smart contracts                                                                         | TPraos                           |        <p>5.0<br>6.0</p>        | Alonzo                           |
| Babbage | <p>Plutus v2<br>Reference inputs<br>Inline datums<br>Reference scripts<br>Removal of d parameter</p> | Praos                            |        <p>7.0<br>8.0</p>        | <p>Vasil<br>Valentine (secp)</p> |

Transitioning from one era to the next is triggered by an **update proposal** that updates the protocol version.

## Update proposals

### Byron era update proposals

<figure><img src="../.gitbook/assets/upbyron.png" alt=""><figcaption></figcaption></figure>

The general mechanism for updating protocol parameters in Byron is as follows:

1. **Update proposal is registered.** An update proposal starts with a transaction that proposes new values for some protocol parameters or a new protocol version. This kind of transaction can only be initiated by a genesis key via its delegate.
2. **Accumulating votes.** Genesis key delegates **vote** for or against the proposal. The proposal must accumulate a sufficient number of votes before it can be confirmed. The threshold is determined by the **minThd** field of the [softforkRule protocol parameter](https://github.com/input-output-hk/cardano-ledger/blob/2a0abd500b9e01efe6dc47146fa8b805ef9ef307/eras/byron/ledger/impl/src/Cardano/Chain/Update/SoftforkRule.hs#L24).
3. **Confirmed (enough votes).** The system records the 'SlotNo' of the slot in which the required threshold of votes was met. At this point, 2k slots (two times the security parameter k) need to pass before the update is stably confirmed and can be _endorsed_. Endorsements for proposals that are not yet stably confirmed are not invalid but rather silently ignored.
4. **Stably-confirmed.** The last required vote is 2k slots deep. Ready to accumulate endorsements. A block whose header's protocol version number is that of the proposal is interpreted as an **endorsement**. In other words, the nodes are ready for the upgrade. Once the number of endorsers satisfies a threshold (same as for voting), the confirmed proposal becomes a **candidate proposal**.
5. **Candidate.** Enough nodes have endorsed the proposal. At this point, a further 2k slots need to pass before the update becomes a stable candidate and can be adopted.
6. **Stable candidate.** The last required endorsement is 2k slots deep.

If there is no stable candidate proposal, then no changes occur. Everything is retained, including a candidate proposal whose threshold-satisfying endorsement was not yet stable and will be adopted in the subsequent epoch unless it gets surpassed in the meantime.

A 'null' update proposal is one that neither increases the protocol version nor the software version. Such update proposals are considered invalid according to the Byron specification.

### Updating proposals in Shelley and subsequent eras

The update mechanism in Shelley is simpler than in Byron. There is no distinction between votes and proposals. To 'vote' for a proposal, a genesis delegate submits the exact same proposal via a regular transaction signed by the delegate key. Additionally, there is no separate endorsement step:

<figure><img src="../.gitbook/assets/upshelley.png" alt=""><figcaption></figcaption></figure>

The procedure is as follows:

1. **Register the proposal.** During each epoch, a genesis key can submit (via its delegates) zero, one, or many proposals; each submission overrides the previous one. Proposals can be explicitly marked for future epochs; in that case, they are simply not considered until that epoch is reached.
2. **Voting.** The window for submitting proposals ends 6k/f slots before the end of the epoch, where *k* is the security parameter and *f* is the _active slot coefficient_.
3. **Quorum.** At the end of the epoch, if the majority of nodes, determined by the **Quorum** specification constant (which must exceed half the total number of nodes), have most recently submitted the identical proposal, it becomes adopted.
4. **Update.** The update is applied at the epoch boundary.

Changing the values of the protocol parameters and global constants can be done via an update proposal.

Changing the values of the global constants always requires a software update, including an increase in the protocol version. An increase in the major version indicates a hard fork, while the minor version indicates a soft fork (meaning old software can validate but not produce new blocks).

On the other hand, updating protocol parameters can be done without a software update.

Until the Voltaire era is released, only the holders of the **genesis delegate keys** can submit and vote on proposals. From time to time, you will find it useful to deploy a local or private testnet. In that case, you will need to use the governance commands to upgrade your network to the desired era.
