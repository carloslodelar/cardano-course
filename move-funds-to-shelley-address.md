# Move funds to shelley address

```
cardano-cli key convert-byron-key \
--byron-signing-key-file utxo-keys/payment.000.key \
--out-file utxo-keys/payment.000.converted.key \
--byron-payment-key-type
```

```
cardano-cli address key-gen \
--verification-key-file utxo-keys/user1.payment.vkey \
--signing-key-file utxo-keys/user1.payment.skey
```

```
cardano-cli address build \
--payment-verification-key-file utxo-keys/user1.payment.vkey \
--testnet-magic 42 \
--out-file utxo-keys/user1.payment.addr
```

```
cardano-cli transaction build-raw \
--shelley-era \
--invalid-hereafter 12000 \
--fee 1000000 \
--tx-in $(cardano-cli byron transaction txid --tx transactions/tx0.tx)#0 \
--tx-out $(cat utxo-keys/user1.payment.addr)+29999999998000000 \
--out-file transactions/tx1.raw
```

```
cardano-cli transaction sign \
--tx-body-file transactions/tx1.raw \
--signing-key-file utxo-keys/payment.000.converted.key \
--testnet-magic 42 \
--out-file transactions/tx1.signed
```

```
cardano-cli transaction submit \
--tx-file transactions/tx1.signed --testnet-magic 42
```

```
cardano-cli query utxo --whole-utxo --testnet-magic 42
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
e1f4d19c74f5d362d521a7170653c1f91771b6f331a0f4e58f73148ce1b3a6cb     0        35999997000000
```
