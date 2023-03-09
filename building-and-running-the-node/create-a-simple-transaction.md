---
description: 'Objective: Learn how to use build-raw and build to create a simple transaction'
cover: ../.gitbook/assets/CABAL (1).png
coverY: 0
---

# Create a simple transaction

We can use `cardano-cli` to create transactions.

```bash
cardano-cli transaction
```

### Create a transaction with `build-raw`

we will need the protocol parameters, so that we can later calculate the transaction fee

{% code overflow="wrap" %}
```bash
cardano-cli query protocol-parameters --testnet-magic 2 --out-file pparams.json
```
{% endcode %}

We need to know the transaction hash and index we will send funds from

```bash
cardano-cli query utxo --address $(cat payment.addr) --testnet-magic 2
```

&#x20;Let's send 5,000 ada to our `paymentwithstake.addr`

{% code overflow="wrap" %}
```bash
cardano-cli transaction build-raw --babbage-era \
--tx-in $(cardano-cli query utxo --address $(cat payment.addr) --testnet-magic 2 --out-file  /dev/stdout | jq -r 'keys[0]') \
--tx-out $(cat paymentwithstake.addr)+5000000000 \
--tx-out $(cat payment.addr)+5000000000 \
--fee 0 \
--protocol-params-file pparams.json \
--out-file tx.draft
```
{% endcode %}

```bash
cardano-cli transaction calculate-min-fee --tx-body-file tx.draft \
--testnet-magic 2 \
--protocol-params-file pparams.json \
--tx-in-count 1 \
--tx-out-count 2 \
--witness-count 1 
```

This will tell us the transaction fee that we need to pay, for example

```
171353 Lovelace
```

Now we need to rebuild the transaction body adding the fees and recalculating the change that we will send to `payment.addr`&#x20;

```
 echo $((10000000000 - 5000000000 - 171353))
 code
 > 4999828647
```

{% code overflow="wrap" %}
```
cardano-cli transaction build-raw --babbage-era \
--tx-in $(cardano-cli query utxo --address $(cat payment.addr) --testnet-magic 2 --out-file  /dev/stdout | jq -r 'keys[0]') \
--tx-out $(cat paymentwithstake.addr)+5000000000 \
--tx-out $(cat payment.addr)+4999828647 \
--fee 171353 \
--protocol-params-file pparams.json \
--out-file tx.raw
```
{% endcode %}

Now we just need to sign and submit the transaction. Of course, we sign it with the `payment.skey` that we generated in the first place.&#x20;

```
cardano-cli transaction sign \
--tx-body-file tx.raw \
--signing-key-file payment.skey \
--testnet-magic 2 \
--out-file tx.signed
```

```bash
cardano-cli transaction submit \
--testnet-magic 2 \
--tx-file tx.signed 
```

### Create a transaction with `build`

Now, let's send the rest of the funds in `payment.addr` to `paymentwithstake.addr` This time we will use the build command which will automatically take care of the fees, simplifying the process

{% code overflow="wrap" %}
```
cardano-cli transaction build --babbage-era \
--testnet-magic 2 \
--tx-in $(cardano-cli query utxo --address $(cat payment.addr) --testnet-magic 2 --out-file  /dev/stdout | jq -r 'keys[0]') \
--change-address $(cat paymentwithstake.addr) \
--out-file tx.raw
```
{% endcode %}

Sign and submit as before

```
cardano-cli transaction sign \
--tx-body-file tx.raw \
--signing-key-file payment.skey \
--testnet-magic 2 \
--out-file tx.signed
```

```
cardano-cli transaction submit \
--testnet-magic 2 \
--tx-file tx.signed 
```

#### The delegation cycle

The delegation cycle comprises four (4) epochs:&#x20;

The cycle starts when the stakeholder delegates his stake to a stake pool. This involves the creation of a _**Delegation Certificate**_ that is then embedded in a _**transaction**_ and _**registered**_ in the blockchain.  In our example below, this happens at any slot of _**Epoch N**_.&#x20;

Then, **** the protocol captures a stake snapshot (_**stakedist)**_ at the last slot of _**Epoch N,**_ recording 3 important pieces of information:&#x20;

1. The balance of each stake key registered in the blockchain,
2. To which stake pool each stake key is delegated,&#x20;
3. The parameters (pool cost, margin, pledge, etc) that each stake pool has set.&#x20;

This snapshot is then used at the end of _**Epoch N+1**_ to randomly select the slot leaders for _**Epoch N+2**_. This process is the essence of Ouroboros as a Proof-of-Stake consensus algorithm:  the bigger the stake, the more chances to be slot leader.  &#x20;

During _**Epoch N+2**_ the stake pools produce the blocks they are entitled to as per the slot leader election. Naturally, stake pools that control more stake will be entitled to more blocks.

In the transition between _**Epoch N+2 and Epoch N+3,**_ a new snapshot registers the rewards collected. And the protocol uses the stake distribution recorded at the end of _**Epoch N**_  to calculate how much of the rewards belongs to each stake key,

Finally, at the transition between _**Epoch N+3**_ and _**Epoch N+4**_  the protocol distributes the rewards to all stake keys. Which show up in wallets at the very start of _**Epoch N+4**_ and credited to the wallet's balance and will count as part of the stake for the snapshot to be taken at the last slot of _**Epoch N+4**_



<figure><img src="../.gitbook/assets/delegationcycle.gif" alt=""><figcaption></figcaption></figure>
