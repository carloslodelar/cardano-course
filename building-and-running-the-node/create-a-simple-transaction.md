---
cover: ../.gitbook/assets/CABAL (1).png
coverY: 0
---

# Creating a simple transaction

You can use `cardano-cli` to create simple transactions:

```bash
cardano-cli transaction
```

## Creating a transaction with `build-raw`

To create a transaction using `build-raw`, you will need the protocol parameters. These parameters are necessary for calculating the transaction fee at a later stage:

{% code overflow="wrap" %}
```bash
cardano-cli query protocol-parameters --testnet-magic 2 --out-file pparams.json
```
{% endcode %}

You will also need to know the transaction hash and index from which you intend to send funds:

```bash
cardano-cli query utxo --address $(cat payment.addr) --testnet-magic 2
```

&#x20;Let's send 5,000 ada to the `paymentwithstake.addr`:

```bash
cardano-cli transaction build-raw --babbage-era \
--tx-in $(cardano-cli query utxo --address $(cat payment.addr) --testnet-magic 2 --out-file  /dev/stdout | jq -r 'keys[0]') \
--tx-out $(cat paymentwithstake.addr)+5000000000 \
--tx-out $(cat payment.addr)+5000000000 \
--fee 0 \
--protocol-params-file pparams.json \
--out-file tx.draft
```

```bash
cardano-cli transaction calculate-min-fee --tx-body-file tx.draft \
--testnet-magic 2 \
--protocol-params-file pparams.json \
--tx-in-count 1 \
--tx-out-count 2 \
--witness-count 1 
```

This information will enable you to determine the transaction fee that needs to be paid, for example:

```
171353 Lovelace
```

Now, you need to reconstruct the transaction body by including the fees and recalculating the change that you intend to send to `payment.addr`&#x20;

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

Sign and submit the transaction using the `payment.skey` generated earlier:&#x20;

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

## Creating a transaction with `build`

Now, let's send the rest of the funds from `payment.addr` to `paymentwithstake.addr`. This time, you can use the build command, which automatically handles the fees, streamlining the process:

{% code overflow="wrap" %}
```
cardano-cli transaction build --babbage-era \
--testnet-magic 2 \
--tx-in $(cardano-cli query utxo --address $(cat payment.addr) --testnet-magic 2 --out-file  /dev/stdout | jq -r 'keys[0]') \
--change-address $(cat paymentwithstake.addr) \
--out-file tx.raw
```
{% endcode %}

Sign and submit:

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

