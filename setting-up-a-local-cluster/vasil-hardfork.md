# Vasil Hardfork

As always, let's start by tweaking our config.json file:

```bash
sed -i configuration/config.json \
-e 's/LastKnownBlockVersion-Major":6/LastKnownBlockVersion-Major":7/'
```

Create the proposal saying that we want to move to protocol version 7.0

```bash
cardano-cli governance create-update-proposal \
--genesis-verification-key-file genesis-keys/non.e.shelley.000.vkey \
--genesis-verification-key-file genesis-keys/non.e.shelley.001.vkey \
--out-file transactions/update.v7.proposal \
--epoch $(cardano-cli query tip --testnet-magic 42 | jq .epoch) \
--protocol-major-version "7" \
--protocol-minor-version "0" 
```

```bash
CHANGE=$(($(cardano-cli query utxo --address $(cat pool1/payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -cs '.[0] | to_entries | .[] | .value.value.lovelace') - 1000000))
```

```bash
cardano-cli transaction build-raw \
--alonzo-era \
--fee 1000000 \
--invalid-hereafter $(expr $(cardano-cli query tip --testnet-magic 42 | jq .slot) + 1000) \
--tx-in $(cardano-cli query utxo --address $(cat pool1/payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -r 'keys[]') \
--tx-out $(cat pool1/payment.addr)+$CHANGE \
--update-proposal-file transactions/update.v7.proposal \
--out-file transactions/update.v7.proposal.txbody
```

```bash
cardano-cli transaction sign \
--tx-body-file transactions/update.v7.proposal.txbody \
--signing-key-file pool1/payment.skey \
--signing-key-file bft0/shelley.000.skey \
--signing-key-file bft1/shelley.001.skey \
--out-file transactions/update.v7.proposal.txsigned
```

```
cardano-cli transaction submit --testnet-magic 42 --tx-file transactions/update.v7.proposal.txsigned
```

```bash
cardano-cli query tip --testnet-magic 42
```
