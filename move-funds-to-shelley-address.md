# Move funds to a Shelley address

We need to convert our Byron signing key to a corresponding Shelley era key:

```bash
cardano-cli key convert-byron-key \
--byron-signing-key-file utxo-keys/payment.000.key \
--out-file utxo-keys/payment.000.converted.key \
--byron-payment-key-type
```

Now let's generate a new set of Shelley era keys:

```bash
cardano-cli address key-gen \
--verification-key-file utxo-keys/user1.payment.vkey \
--signing-key-file utxo-keys/user1.payment.skey
```

We build the address with

```bash
cardano-cli address build \
--payment-verification-key-file utxo-keys/user1.payment.vkey \
--testnet-magic 42 \
--out-file utxo-keys/user1.payment.addr
```

Build the transaction to move the funds from the their current Byron era address to our newly created Shelley era `user1.payment.add`This address will have all the funds for now:

```bash
cardano-cli transaction build-raw \
--shelley-era \
--invalid-hereafter $((cardano-cli query tip --testnet-magic 42 | jq .slot)+1000) \
--fee 1000000 \
--tx-in $(cardano-cli byron transaction txid --tx transactions/tx0.tx)#0 \
--tx-out $(cat utxo-keys/user1.payment.addr)+29999999998000000 \
--out-file transactions/tx1.raw
```

```bash
cardano-cli transaction sign \
--tx-body-file transactions/tx1.raw \
--signing-key-file utxo-keys/payment.000.converted.key \
--testnet-magic 42 \
--out-file transactions/tx1.signed
```

```bash
cardano-cli transaction submit \
--tx-file transactions/tx1.signed --testnet-magic 42
```

```bash
cardano-cli query utxo --whole-utxo --testnet-magic 42
```

```
   TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
e1f4d19c74f5d362d521a7170653c1f91771b6f331a0f4e58f73148ce1b3a6cb     0     29999999998000000
```
