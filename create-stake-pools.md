# Create first stake pool

So we are now in Shelley era, but block production is still controlled by our BFT nodes. Let's create the first stake pool for our system:

```
mkdir pool1
```

We need an address and funds (for the pool owner):

```
cardano-cli address key-gen \
--verification-key-file pool1/payment.vkey \
--signing-key-file pool1/payment.skey
```

We want to delegate our stake to our pool, so we will need stake keys

```
cardano-cli stake-address key-gen \
--verification-key-file pool1/stake.vkey \
--signing-key-file pool1/stake.skey
```

Build the address:

```
cardano-cli address build \
--payment-verification-key-file pool1/payment.vkey \
--stake-verification-key-file pool1/stake.vkey \
--out-file pool1/payment.addr \
--testnet-magic 42
```

Send some funds to our pool owner address from `user1.payment.addr`

```
cardano-cli transaction build \
--shelley-era \
--testnet-magic 42 \
--invalid-hereafter $(expr $(cardano-cli query tip --testnet-magic 42 | jq .slot) + 1000) \
--tx-in $(cardano-cli query utxo --address $(cat utxo-keys/user1.payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -r 'keys[]') \
--tx-out $(cat pool1/payment.addr)+50000000000000 \
--change-address $(cat utxo-keys/user1.payment.addr) \
--out-file transactions/tx3.raw
```

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

```
cardano-cli query utxo --address $(cat pool1/payment.addr) --testnet-magic 42
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
d8dad0d24242b26e037f9b0120030fc2d5c5449d859ecd3267f6453dd66bf0c3     0        50000000000000
```

Generate cold keys

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
cardano-cli node key-gen-KES \
--verification-key-file pool1/kes.vkey \
--signing-key-file pool1/kes.skey
```

To generate the operational certificate:

```
cardano-cli node issue-op-cert \
--kes-verification-key-file pool1/kes.vkey \
--cold-signing-key-file pool1/cold.skey \
--operational-certificate-issue-counter pool1/opcert.counter \
--kes-period 0 \
--out-file pool1/opcert.cert
```

Let's create a topology file for our pool and update the bft nodes topology so tat they include our pool.&#x20;

```
cat > pool1/topology.json <<EOF
{
   "Producers": [
     {
       "addr": "127.0.0.1",
       "port": 3000,
       "valency": 1
     },
     {
       "addr": "127.0.0.1",
       "port": 3001,
       "valency": 1
     }
   ]
 }
EOF
```

```
cat > bft0/topology.json <<EOF
{
   "Producers": [
     {
       "addr": "127.0.0.1",
       "port": 3001,
       "valency": 1
     },
     {
       "addr": "127.0.0.1",
       "port": 3002,
       "valency": 1
     }
   ]
 }
EOF
```

```
cat > bft1/topology.json <<EOF
{
   "Producers": [
     {
       "addr": "127.0.0.1",
       "port": 3000,
       "valency": 1
     },
     {
       "addr": "127.0.0.1",
       "port": 3002,
       "valency": 1
     }
   ]
 }
EOF
```

And let's have a simple script to start the pool node.&#x20;

```
cat > pool1/startnode.sh <<EOF
#!/usr/bin/env bash

cardano-node run \
--config ../configuration/config.json \
--topology topology.json \
--database-path db \
--socket-path node.socket \
--port 3002 \
--shelley-kes-key kes.skey \
--shelley-vrf-key vrf.skey \
--shelley-operational-certificate opcert.cert
EOF
```

Give it executable permissions:

```
chmod +x pool1/startnode.sh
```

Start the node from the pool1 directory.

Our pool cannot create blocks just yet. We need to register it. To the blockchain

#### Register stake key

Create a registration certificate

```
cardano-cli stake-address registration-certificate \
--stake-verification-key-file pool1/stake.vkey \
--out-file pool1/stake.cert
```

Create a transaction to register the stake key

```
cardano-cli query utxo --address $(cat pool1/payment.addr) --testnet-magic 42
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
3c40b6543a5888a61f630a28de520f6f7ecccc23d0e1e08fd9510a4e61156926     1        99998000000
```

```
cardano-cli query protocol-parameters --testnet-magic 42 | grep stakeAddressDeposit
    "stakeAddressDeposit": 2000000,

```

{% code overflow="wrap" %}
```
CHANGE=$(($(cardano-cli query utxo --address $(cat pool1/payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -cs '.[0] | to_entries | .[] | .value.value') - 3000000))
```
{% endcode %}

```
cardano-cli transaction build-raw \
--shelley-era \
--fee 1000000 \
--invalid-hereafter $(expr $(cardano-cli query tip --testnet-magic 42 | jq .slot) + 1000) \
--tx-in $(cardano-cli query utxo --address $(cat pool1/payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -r 'keys[]') \
--tx-out $(cat pool1/payment.addr)+$CHANGE \
--certificate-file pool1/stake.cert \
--out-file transactions/tx4.raw
```

```
cardano-cli transaction sign \
--tx-body-file transactions/tx4.raw \
--signing-key-file pool1/payment.skey \
--signing-key-file pool1/stake.skey \
--testnet-magic 42 \
--out-file transactions/tx4.signed
```

```
cardano-cli transaction submit \
--testnet-magic 42 \
--tx-file transactions/tx4.signed
```

#### Register stake pool

Now, it's time to register the stake pool. On a real network we would need to have metadata for our pool so tat it can be properly displayed by wallets.&#x20;

The pool metadata must meet [this requirements](https://docs.cardano.org/development-guidelines/operating-a-stake-pool/public-stake-pools) &#x20;

We will use the same file [https://git.io/JJWdJ ](https://git.io/JJWdJ)

Get the file

```bash
wget https://git.io/JJWdJ -O pool1/poolmetadata.json
```

Get the metadata hash

```
cardano-cli stake-pool metadata-hash --pool-metadata-file pool1/poolmetadata.json
5c93c2c47f115c0184e5b2499b179b36dc385a45a2a809bb3ec2e34047dae04e
```

For convinenve, lets save it to a file&#x20;

```
cardano-cli stake-pool metadata-hash --pool-metadata-file pool1/poolmetadata.json --out-file poolmetadata.hash
```

Generate the registration certificate

```
cardano-cli stake-pool registration-certificate \
--cold-verification-key-file pool1/cold.vkey \
--vrf-verification-key-file pool1/vrf.vkey \
--pool-pledge 1000000000000 \
--pool-cost 340000000 \
--pool-margin 10/100 \
--pool-reward-account-verification-key-file pool1/stake.vkey \
--pool-owner-stake-verification-key-file pool1/stake.vkey \
--testnet-magic 42 \
--pool-relay-ipv4 127.0.0.1 \
--pool-relay-port 3002 \
--metadata-url https://git.io/JJWdJ \
--metadata-hash $(cat poolmetadata.hash) \
--out-file pool1/pool-registration.cert
```

Create a delegation certificate

```
cardano-cli stake-address delegation-certificate \
--stake-verification-key-file pool1/stake.vkey \
--cold-verification-key-file pool1/cold.vkey \
--out-file pool1/delegation.cert
```

Now, let's query the protocol parameters again to find the pool deposit:

```bash
cardano-cli query protocol-parameters --testnet-magic 42 | grep stakePoolDeposit
    "stakePoolDeposit": 500000000,

```

So we need to make a deposit of 500 test ADA

Let's submit both, the delegation certificate and the registration certificate:&#x20;

```
CHANGE=$(($(cardano-cli query utxo --address $(cat pool1/payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -cs '.[0] | to_entries | .[] | .value.value') - 501000000))
```

```
cardano-cli transaction build-raw \
--shelley-era \
--fee 1000000 \
--invalid-hereafter $(expr $(cardano-cli query tip --testnet-magic 42 | jq .slot) + 10000) \
--tx-in $(cardano-cli query utxo --address $(cat pool1/payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -r 'keys[]') \
--tx-out $(cat pool1/payment.addr)+$CHANGE \
--certificate-file pool1/pool-registration.cert \
--certificate-file pool1/delegation.cert \
--out-file transactions/tx5.raw
```

```
cardano-cli transaction sign \
--tx-body-file transactions/tx5.raw \
--signing-key-file pool1/payment.skey \
--signing-key-file pool1/stake.skey \
--signing-key-file pool1/cold.skey \
--testnet-magic 42 \
--out-file transactions/tx5.signed
```

```
cardano-cli transaction submit \
--testnet-magic 42 \
--tx-file transactions/tx5.signed
```

```
cardano-cli stake-pool id --cold-verification-key-file pool1/cold.vkey --output-format "hex"
66da34a16bd8582679442d045514ecd9817f24199e771d017ad7a8c2
```

Now we need to wait 1 epoch for the stake snapshot to pick up our pool and 2 epochs for our pool to start producing blocks. This of course, if we lower the decentralization parameter to something less than 1. Let's do that.&#x20;
