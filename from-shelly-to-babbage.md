# From Shelly to Babbage

Let's continue upgrading our system to later eras. First we move from Shelley to Allegra.

```
sed -i configuration/shelley-genesis.json \
-e 's/"major": 1/"major": 3/'
```

```
sed -i configuration/config.json \
-e 's/LastKnownBlockVersion-Major":2/LastKnownBlockVersion-Major":3/' \
-e 's/"MaxKnownMajorProtocolVersion":2/"MaxKnownMajorProtocolVersion":7/'
```

This means we need to go to protocol version 3.0&#x20;

```
cardano-cli governance create-update-proposal \
--genesis-verification-key-file genesis-keys/non.e.shelley.000.vkey \
--genesis-verification-key-file genesis-keys/non.e.shelley.001.vkey \
--out-file transactions/update.v3.proposal \
--epoch $(cardano-cli query tip --testnet-magic 42 | jq .epoch) \
--protocol-major-version "3" \
--protocol-minor-version "0" 
```

```
CHANGE=$(($(cardano-cli query utxo --address $(cat pool1/payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -cs '.[0] | to_entries | .[] | .value.value') - 1000000))
```

```
cardano-cli transaction build-raw \
--shelley-era \
--fee 1000000 \
--invalid-hereafter $(expr $(cardano-cli query tip --testnet-magic 42 | jq .slot) + 1000) \
--tx-in $(cardano-cli query utxo --address $(cat pool1/payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -r 'keys[]') \
--tx-out $(cat pool1/payment.addr)+$CHANGE \
--update-proposal-file transactions/update.v3.proposal \
--out-file transactions/update.v3.proposal.txbody
```

```
cardano-cli transaction sign \
--tx-body-file transactions/update.v3.proposal.txbody \
--signing-key-file pool1/payment.skey \
--signing-key-file bft0/shelley.000.skey \
--signing-key-file bft1/shelley.001.skey \
--out-file transactions/update.v3.proposal.txsigned
```

```
cardano-cli transaction submit --testnet-magic 42 --tx-file transactions/update.v3.proposal.txsigned
```
