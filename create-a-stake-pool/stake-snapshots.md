# Stake snapshots

We will start producing blocks 2 epochs after registering the stake pool. See section 11.1 of the [Shelley Ledger Specification](https://github.com/input-output-hk/cardano-ledger/releases/latest/download/shelley-ledger.pdf)

<figure><img src="../.gitbook/assets/shelley-spec-.png" alt=""><figcaption></figcaption></figure>

You can query the stake snapshots:

{% code overflow="wrap" %}
```
cardano-cli query stake-snapshot --testnet-magic 2 --stake-pool-id <POOL_ID>
```
{% endcode %}
