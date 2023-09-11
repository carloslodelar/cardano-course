# Stake snapshots

At every epoch transition, the system performs a stake distribution snapshot. These snapshots are used for the slot leader election process and consist of:

* _Stake_: a mapping of credentials to lovelace&#x20;
* _Delegations_: a mapping of credentials to stake pools
* _Pool parameters_: each stake pool's parameters storage

At any given time, the system must store the last three snapshots. To keep track of snapshots, `Mark, Set, and Go` naming is used. `Mark` is the most recent one, which becomes `Set` after an epoch and `Go` after another epoch.&#x20;

&#x20;A snapshot **Mark** at epoch transition from *EpochN* to *EpochN+1* will be used for the slot leader election process during *EpochN+2*, when it becomes **Set**. Finally, it is employed for rewards calculation during *Epoch N+3* when it becomes **Go**.&#x20;

<figure><img src="../.gitbook/assets/SNAPSHOT1.png" alt=""><figcaption></figcaption></figure>

On any current epoch, **Mark** is not actively used, **Set** is used for the slot leader election process, and **Go** is used for the rewards calculation:

<figure><img src="../.gitbook/assets/MARKSETGO (1).png" alt=""><figcaption></figcaption></figure>

For more details, please see section 11.1 of the [Shelley ledger specification](https://github.com/input-output-hk/cardano-ledger/releases/latest/download/shelley-ledger.pdf).

<figure><img src="../.gitbook/assets/shelley-spec-.png" alt=""><figcaption></figcaption></figure>

You can query the stake snapshots:

{% code overflow="wrap" %}
```
cardano-cli query stake-snapshot --testnet-magic 2 --stake-pool-id <POOL_ID>
```
{% endcode %}

```
{
    "activeStakeGo": 145769691498797,
    "activeStakeMark": 147258956481483,
    "activeStakeSet": 146034637994668,
    "poolStakeGo": 9497641630,
    "poolStakeMark": 9497641630,
    "poolStakeSet": 9497641630
}
```
