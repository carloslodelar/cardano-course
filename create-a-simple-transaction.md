---
description: 'Objective: Learn how to use build-raw and build to create a simple transaction'
---

# Create a simple transaction

We can use `cardano-cli` to create transactions.

```bash
cardano-cli transaction
```

### Create a transaction with `build-raw`

We need to know the transaction hash and index we will send funds from

```bash
cardano-cli query utxo --address $(cat payment.addr) --testnet-magic 1
```

we will need the protocol parameters, so that we can later calculate the transaction fee

{% code overflow="wrap" %}
```bash
cardano-cli query protocol-parameters --testnet-magic 1 --out-file pparams.json
```
{% endcode %}

&#x20;Let's send 5,000 ada to our `paymentwithstake.addr`

```
cardano-cli transaction build-raw --babbage-era \
--tx-in 
```
