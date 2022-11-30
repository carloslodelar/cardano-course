# Create first stake pool

So we are now in Shelley era, but block production is still controlled by our BFT nodes. Let's create the first stake pool for our system:

```
mkdir pool1
```

We will need an address and funds (for the pool owner):

```
cardano-cli address key-gen \
--verification-key-file pool1/payment.vkey \
--signing-key-file pool1/payment.skey
```

We will delegate our stake to our pool, so we will need stake keys

```
cardano-cli stake-address key-gen \
--verification-key-file pool1/stake.vkey \
--signing-key-file pool1/stake.skey
```

And we build our address with:

```
cardano-cli address build \
--payment-verification-key-file pool1/payment.vkey \
--stake-verification-key-file pool1/stake.vkey \
--out-file pool1/payment.addr \
--testnet-magic 42
```

Let's send some funds to our pool owner address:

{% code overflow="wrap" %}
```
cardano-cli transaction build \
--shelley-era \
--tx-in $(cardano-cli query utxo --address $(cat utxo-keys/user1.payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -r 'keys[]') \
--tx-out $(cat pool1/payment.addr)+99998000000 \
--change-address $(cat utxo-keys/user1.payment.addr) \
--out-file transactions/tx3.raw
```
{% endcode %}

Sign it

```
cardano-cli transaction sign \
--tx-body-file transactions/tx3.raw \
--signing-key-file utxo-keys/user1.payment.skey \
--testnet-magic 42 \
--out-file transactions/tx3.signed
```

Submit to the blockchain

```
cardano-cli transaction submit \
--testnet-magic 42 \
--tx-file transactions/tx3.signed
```

Generate cold keys.&#x20;

```
cardano-cli node key-gen \
--cold-verification-key-file pool1/cold.vkey \
--cold-signing-key-file pool1/cold.skey \
--operational-certificate-issue-counter-file pool1/opcert.counter
```

Generate VRF keys

```
cardano-cli node key-gen-VRF \
--verification-key-file pool1/vrf.vkey \
--signing-key-file pool1/vrf.skey
```

&#x20;Generate KES keys

```
cardano-cli node key-gen-KES
--verification-key-file pool1/kes.vkey
--signing-key-file pool1/kes.skey
```

To generate the operational certificate:

```
cardano-cli node issue-op-cert \
--kes-verification-key-file pool1/kes.vkey \
--cold-signing-key-file pool1/c0old.skey \
--operational-certificate-issue-counter pool1/opcert.counter \
--kes-period 0 \
--out-file pool1/opcert.cert
```
