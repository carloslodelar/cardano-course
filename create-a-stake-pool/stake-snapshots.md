# Stake snapshots

We are eligible to start producing blocks 2 epochs after registering the stake pool. This is ater having take part of 2 snapshots. See section 11.1 of the [Shelley Ledger Specification](https://github.com/input-output-hk/cardano-ledger/releases/latest/download/shelley-ledger.pdf)

<figure><img src="../.gitbook/assets/shelley-spec-.png" alt=""><figcaption></figcaption></figure>

At every epoch transition the system performs a stake distribution snapshot consisting of:

* _Stake: a mapping of credentials to lovelace._&#x20;
* _Delegations_: a mapping of credentials to stake pools
* _Pool Parameters_, storing the pool parameters of each stake pool



At any given time we must store the last three snapshots, they are used for . The oldest one is used in the current epoch for the rewards calculation, the second is used in slot leader election process. the latest one is not actively used on the current epoch.&#x20;

To keep track of these three snapshots we use the names Mark, Set, Go. Where  **Mark** is the most recent one,  after an epoch it becomes **Set** and after another epoch it becomes **Go**.  &#x20;

On the Current epoch, **Mark** is not used, **Set** is used for the slot leader election process, and **Go** is used for the rewards calculation. \


<figure><img src="../.gitbook/assets/MARKSETGO.png" alt=""><figcaption></figcaption></figure>

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
