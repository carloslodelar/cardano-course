---
cover: ../.gitbook/assets/cluster (1).png
coverY: 0
---

# From Shelly to Alonzo

### Allegra Hardfork

Let's continue upgrading our system to later eras. First we move from Shelley to Allegra. To achieve this we need to upgrade to protocol version 3.0

{% tabs %}
{% tab title="Linux" %}
```bash
sed -i configuration/config.json \
-e 's/LastKnownBlockVersion-Major":2/LastKnownBlockVersion-Major":3/' \
-e 's/"MaxKnownMajorProtocolVersion":2/"MaxKnownMajorProtocolVersion":7/'
```
{% endtab %}

{% tab title="macOS" %}
```bash
gsed -i configuration/config.json \
-e 's/LastKnownBlockVersion-Major":2/LastKnownBlockVersion-Major":3/' \
-e 's/"MaxKnownMajorProtocolVersion":2/"MaxKnownMajorProtocolVersion":7/'
```
{% endtab %}
{% endtabs %}

Create the proposal:

```bash
cardano-cli governance create-update-proposal \
--genesis-verification-key-file genesis-keys/non.e.shelley.000.vkey \
--genesis-verification-key-file genesis-keys/non.e.shelley.001.vkey \
--out-file transactions/update.v3.proposal \
--epoch $(cardano-cli query tip --testnet-magic 42 | jq .epoch) \
--protocol-major-version "3" \
--protocol-minor-version "0" 
```

{% code overflow="wrap" %}
```bash
CHANGE=$(($(cardano-cli query utxo --address $(cat pool1/payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -cs '.[0] | to_entries | .[] | .value.value') - 1000000))
```
{% endcode %}

```bash
cardano-cli transaction build-raw \
--shelley-era \
--fee 1000000 \
--invalid-hereafter $(expr $(cardano-cli query tip --testnet-magic 42 | jq .slot) + 1000) \
--tx-in $(cardano-cli query utxo --address $(cat pool1/payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -r 'keys[]') \
--tx-out $(cat pool1/payment.addr)+$CHANGE \
--update-proposal-file transactions/update.v3.proposal \
--out-file transactions/update.v3.proposal.txbody
```

```bash
cardano-cli transaction sign \
--tx-body-file transactions/update.v3.proposal.txbody \
--signing-key-file pool1/payment.skey \
--signing-key-file bft0/shelley.000.skey \
--signing-key-file bft1/shelley.001.skey \
--out-file transactions/update.v3.proposal.txsigned
```

```bash
cardano-cli transaction submit --testnet-magic 42 \
--tx-file transactions/update.v3.proposal.txsigned
```

Wait for the next epoch transition to see the Allegra hardfork:

### Mary hardfork

To get to Mary era we upgrade to protocol version 4.0&#x20;

{% tabs %}
{% tab title="Linux" %}
```bash
sed -i configuration/config.json \
-e 's/LastKnownBlockVersion-Major":3/LastKnownBlockVersion-Major":4/'
```
{% endtab %}

{% tab title="macOS" %}
```bash
gsed -i configuration/config.json \
-e 's/LastKnownBlockVersion-Major":3/LastKnownBlockVersion-Major":4/'
```
{% endtab %}
{% endtabs %}

```bash
cardano-cli governance create-update-proposal \
--genesis-verification-key-file genesis-keys/non.e.shelley.000.vkey \
--genesis-verification-key-file genesis-keys/non.e.shelley.001.vkey \
--out-file transactions/update.v4.proposal \
--epoch $(cardano-cli query tip --testnet-magic 42 | jq .epoch) \
--protocol-major-version "4" \
--protocol-minor-version "0" 
```

{% code overflow="wrap" %}
```bash
CHANGE=$(($(cardano-cli query utxo --address $(cat pool1/payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -cs '.[0] | to_entries | .[] | .value.value') - 1000000))
```
{% endcode %}

```bash
cardano-cli transaction build-raw \
--shelley-era \
--fee 1000000 \
--invalid-hereafter $(expr $(cardano-cli query tip --testnet-magic 42 | jq .slot) + 1000) \
--tx-in $(cardano-cli query utxo --address $(cat pool1/payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -r 'keys[]') \
--tx-out $(cat pool1/payment.addr)+$CHANGE \
--update-proposal-file transactions/update.v4.proposal \
--out-file transactions/update.v4.proposal.txbody
```

```bash
cardano-cli transaction sign \
--tx-body-file transactions/update.v4.proposal.txbody \
--signing-key-file pool1/payment.skey \
--signing-key-file bft0/shelley.000.skey \
--signing-key-file bft1/shelley.001.skey \
--out-file transactions/update.v4.proposal.txsigned
```

```bash
cardano-cli transaction submit --testnet-magic 42 \
--tx-file transactions/update.v4.proposal.txsigned
```

Wait for the next epoch transition to see the Mary hardfork:

### Alonzo hardfork and Alonzo intra-era hardfork

Getting to Alonzo will require the Alonzo Hardfork AND the Alonzo intra-era hardfork, this is, we will move from protocol version 4.0 to 5.0 and then to 6.0&#x20;

{% tabs %}
{% tab title="Linux" %}
```bash
sed -i configuration/config.json \
-e 's/LastKnownBlockVersion-Major":4/LastKnownBlockVersion-Major":5/'
```
{% endtab %}

{% tab title="macOS" %}
```bash
gsed -i configuration/config.json \
-e 's/LastKnownBlockVersion-Major":4/LastKnownBlockVersion-Major":5/'
```
{% endtab %}
{% endtabs %}

```bash
cardano-cli governance create-update-proposal \
--genesis-verification-key-file genesis-keys/non.e.shelley.000.vkey \
--genesis-verification-key-file genesis-keys/non.e.shelley.001.vkey \
--out-file transactions/update.v5.proposal \
--epoch $(cardano-cli query tip --testnet-magic 42 | jq .epoch) \
--protocol-major-version "5" \
--protocol-minor-version "0" 
```

{% code overflow="wrap" %}
```
CHANGE=$(($(cardano-cli query utxo --address $(cat pool1/payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -cs '.[0] | to_entries | .[] | .value.value.lovelace') - 1000000))
```
{% endcode %}

```bash
cardano-cli transaction build-raw \
--mary-era \
--fee 1000000 \
--invalid-hereafter $(expr $(cardano-cli query tip --testnet-magic 42 | jq .slot) + 1000) \
--tx-in $(cardano-cli query utxo --address $(cat pool1/payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -r 'keys[]') \
--tx-out $(cat pool1/payment.addr)+$CHANGE \
--update-proposal-file transactions/update.v5.proposal \
--out-file transactions/update.v5.proposal.txbody
```

```bash
cardano-cli transaction sign \
--tx-body-file transactions/update.v5.proposal.txbody \
--signing-key-file pool1/payment.skey \
--signing-key-file bft0/shelley.000.skey \
--signing-key-file bft1/shelley.001.skey \
--out-file transactions/update.v5.proposal.txsigned
```

```bash
cardano-cli transaction submit --testnet-magic 42 \
--tx-file transactions/update.v5.proposal.txsigned
```

### Alonzo intra-era hardfork

{% tabs %}
{% tab title="Linux" %}
```bash
sed -i configuration/config.json \
-e 's/LastKnownBlockVersion-Major":5/LastKnownBlockVersion-Major":6/'
```
{% endtab %}

{% tab title="macOS" %}
```bash
gsed -i configuration/config.json \
-e 's/LastKnownBlockVersion-Major":5/LastKnownBlockVersion-Major":6/'
```
{% endtab %}
{% endtabs %}

```bash
cardano-cli governance create-update-proposal \
--genesis-verification-key-file genesis-keys/non.e.shelley.000.vkey \
--genesis-verification-key-file genesis-keys/non.e.shelley.001.vkey \
--out-file transactions/update.v6.proposal \
--epoch $(cardano-cli query tip --testnet-magic 42 | jq .epoch) \
--protocol-major-version "6" \
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
--update-proposal-file transactions/update.v6.proposal \
--out-file transactions/update.v6.proposal.txbody
```

```bash
cardano-cli transaction sign \
--tx-body-file transactions/update.v6.proposal.txbody \
--signing-key-file pool1/payment.skey \
--signing-key-file bft0/shelley.000.skey \
--signing-key-file bft1/shelley.001.skey \
--out-file transactions/update.v6.proposal.txsigned
```

```bash
cardano-cli transaction submit --testnet-magic 42 \
--tx-file transactions/update.v6.proposal.txsigned
```
