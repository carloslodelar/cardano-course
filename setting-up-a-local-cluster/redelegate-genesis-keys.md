# Redelegate genesis keys

In a real-life scenario a holder of a genesis key may wish to delegate to a new key. &#x20;

Generate a new (Shelley era) delegation key pair:

```bash
cardano-cli genesis key-gen-delegate \
--verification-key-file delegate-keys/new.shelley.delegate.000.vkey \
--signing-key-file delegate-keys/new.shelley.delegate.000.skey \
--operational-certificate-issue-counter-file delegate-keys/new.shelley.delegate.000.certificate.counter
```

Issue a new delegation certificate for genesis key 000

```bash
cardano-cli governance create-genesis-key-delegation-certificate \
--genesis-verification-key-file genesis-keys/non.e.shelley.000.vkey \
--genesis-delegate-verification-key-file delegate-keys/new.shelley.delegate.000.vkey \
--vrf-verification-key-file bft0/shelley.000.vrf.vkey \
--out-file genesis-keys/genesis.delegation.cert
```

Submit the certificate in a transaction, we need to sign it with the genesis key.&#x20;

```bash
cardano-cli query utxo --address $(cat utxo-keys/user1.payment.addr) --testnet-magic 42
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
7a7226867b8ea7eee300465cc59ec1303bc386a524048a77f6cea9f897431fad     1        29899999997665742 lovelace + TxOutDatumNon
```

```bash
CHANGE=$((29899999997665742 - 1000000))
```

```
cardano-cli transaction build-raw \
--alonzo-era \
--fee 1000000 \
--tx-in $(cardano-cli query utxo --address $(cat utxo-keys/user1.payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -r 'keys[]') \
--tx-out $(cat utxo-keys/user1.payment.addr)+$CHANGE \
--certificate-file genesis-keys/genesis.delegation.cert \
--out-file transactions/tx9.raw
```

```
cardano-cli transaction sign \
--tx-body-file transactions/tx9.raw \
--signing-key-file utxo-keys/user1.payment.skey \
--signing-key-file genesis-keys/shelley.000.skey \
--out-file transactions/tx9.signed
```

```
cardano-cli transaction submit \
--testnet-magic 42 \
--tx-file transactions/tx9.signed
```
