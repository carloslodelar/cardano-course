# Move funds to shelley address

```
cardano-cli key convert-byron-key \
--byron-signing-key-file byron/payment.000.key \
--out-file byron/payment.000.converted.key \
--byron-payment-key-type
```

```
cardano-cli key convert-byron-key \
--byron-signing-key-file byron/payment.001.key \
--out-file byron/payment.001.converted.key \
--byron-payment-key-type
```

```
cardano-cli address key-gen \
--verification-key-file shelley/utxo-keys/payment.000.vkey \
--signing-key-file shelley/utxo-keys/payment.000.skey
```

```
cardano-cli address build \
--payment-verification-key-file shelley/utxo-keys/payment.000.vkey \
--testnet-magic 42 \
--out-file shelley/utxo-keys/payment.000.addr

```

```
cardano-cli byron transaction txid --tx transactions/tx0.tx

bd946b437bfdb4fb592afbc71cf253d473a3da03f391c3d0987530312483062d
```

```
cardano-cli byron transaction txid --tx transactions/tx1.tx

b6e52648a2a855de4600e4d0b75dd4f1e1a32868b20092d23ae959551c1d2216
```

```
cardano-cli transaction build-raw \
--shelley-era \
--invalid-hereafter 100000 \
--fee 1000000 \
--tx-in bd946b437bfdb4fb592afbc71cf253d473a3da03f391c3d0987530312483062d#0 \
--tx-in b6e52648a2a855de4600e4d0b75dd4f1e1a32868b20092d23ae959551c1d2216#0 \
--tx-out $(cat shelley/utxo-keys/payment.000.addr)+35999997000000 \
--out-file transactions/tx3.raw 
```

```
cardano-cli transaction sign \
--tx-body-file transactions/tx3.raw \
--signing-key-file byron/payment.000.converted.key \
--signing-key-file byron/payment.001.converted.key \
--testnet-magic 42 \
--out-file transactions/tx3.signed
```

```
cardano-cli transaction submit \
--tx-file transactions/tx3.signed --testnet-magic 42
```

```
cardano-cli query utxo --whole-utxo --testnet-magic 42
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
e1f4d19c74f5d362d521a7170653c1f91771b6f331a0f4e58f73148ce1b3a6cb     0        35999997000000
```
