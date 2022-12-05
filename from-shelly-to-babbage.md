# From Shelly to Babbage

### Allegra Hardfork

Let's continue upgrading our system to later eras. First we move from Shelley to Allegra. To achieve this we need to upgrade to protocol version 3.0

```
sed -i configuration/config.json \
-e 's/LastKnownBlockVersion-Major":2/LastKnownBlockVersion-Major":3/' \
-e 's/"MaxKnownMajorProtocolVersion":2/"MaxKnownMajorProtocolVersion":7/'
```

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

Wait for the next epoch transition to see the Allegra hardfork:

#### Mary hardfork

To get to Mary era we upgrade to protocol version 4.0&#x20;

```
sed -i configuration/config.json \
-e 's/LastKnownBlockVersion-Major":2/LastKnownBlockVersion-Major":4/'
```

```
cardano-cli governance create-update-proposal \
--genesis-verification-key-file genesis-keys/non.e.shelley.000.vkey \
--genesis-verification-key-file genesis-keys/non.e.shelley.001.vkey \
--out-file transactions/update.v4.proposal \
--epoch $(cardano-cli query tip --testnet-magic 42 | jq .epoch) \
--protocol-major-version "4" \
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
--update-proposal-file transactions/update.v4.proposal \
--out-file transactions/update.v4.proposal.txbody
```

```
cardano-cli transaction sign \
--tx-body-file transactions/update.v4.proposal.txbody \
--signing-key-file pool1/payment.skey \
--signing-key-file bft0/shelley.000.skey \
--signing-key-file bft1/shelley.001.skey \
--out-file transactions/update.v4.proposal.txsigned
```

```
cardano-cli transaction submit --testnet-magic 42 --tx-file transactions/update.v4.proposal.txsigned
```

Wait for the next epoch transition to see the Mary hardfork:

### Alonzo hardfork

Getting to Alonzo will require the Alonzo Hardfork AND the Alonzo intra-era hardfork, this is, we will move from protocol version 4.0 to 5.0 followed by 6.0&#x20;

```
sed -i configuration/config.json \
-e 's/LastKnownBlockVersion-Major":2/LastKnownBlockVersion-Major":5/'
```

```
cardano-cli governance create-update-proposal \
--genesis-verification-key-file genesis-keys/non.e.shelley.000.vkey \
--genesis-verification-key-file genesis-keys/non.e.shelley.001.vkey \
--out-file transactions/update.v5.proposal \
--epoch $(cardano-cli query tip --testnet-magic 42 | jq .epoch) \
--protocol-major-version "5" \
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
--update-proposal-file transactions/update.v5.proposal \
--out-file transactions/update.v5.proposal.txbody
```

```
cardano-cli transaction sign \
--tx-body-file transactions/update.v5.proposal.txbody \
--signing-key-file pool1/payment.skey \
--signing-key-file bft0/shelley.000.skey \
--signing-key-file bft1/shelley.001.skey \
--out-file transactions/update.v5.proposal.txsigned
```

```
cardano-cli transaction submit --testnet-magic 42 --tx-file transactions/update.v5.proposal.txsigned
```

