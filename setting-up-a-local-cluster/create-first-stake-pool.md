---
cover: ../.gitbook/assets/cluster (1).png
coverY: 0
---

# Creating a first stake pool

The node is now in the Shelley era, but block production is still controlled by the BFT nodes. Create the first stake pool for the system:

```bash
mkdir pool1
```

## Generating pool owner keys

You need an address and funds (for the pool owner):

```bash
cardano-cli address key-gen \
--verification-key-file pool1/payment.vkey \
--signing-key-file pool1/payment.skey
```

To delegate your stake to the pool, you will need stake keys:

```bash
cardano-cli stake-address key-gen \
--verification-key-file pool1/stake.vkey \
--signing-key-file pool1/stake.skey
```

Build the address:

```bash
cardano-cli address build \
--payment-verification-key-file pool1/payment.vkey \
--stake-verification-key-file pool1/stake.vkey \
--out-file pool1/payment.addr \
--testnet-magic 42
```

Send some funds to the pool owner address from `user1.payment.addr`:

```bash
cardano-cli transaction build \
--shelley-era \
--testnet-magic 42 \
--invalid-hereafter $(expr $(cardano-cli query tip --testnet-magic 42 | jq .slot) + 1000) \
--tx-in $(cardano-cli query utxo --address $(cat utxo-keys/user1.payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -r 'keys[]') \
--tx-out $(cat pool1/payment.addr)+50000000000000 \
--change-address $(cat utxo-keys/user1.payment.addr) \
--out-file transactions/tx3.raw
```

Sign the transaction:

```bash
cardano-cli transaction sign \
--tx-body-file transactions/tx3.raw \
--signing-key-file utxo-keys/user1.payment.skey \
--testnet-magic 42 \
--out-file transactions/tx3.signed
```

Submit it to the blockchain:

```bash
cardano-cli transaction submit \
--testnet-magic 42 \
--tx-file transactions/tx3.signed
```

```bash
cardano-cli query utxo --address $(cat pool1/payment.addr) --testnet-magic 42
```

```bash
  TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
d8dad0d24242b26e037f9b0120030fc2d5c5449d859ecd3267f6453dd66bf0c3     0        50000000000000
```

## Generating stake pool keys

Generate cold keys:

```bash
cardano-cli node key-gen \
--cold-verification-key-file pool1/cold.vkey \
--cold-signing-key-file pool1/cold.skey \
--operational-certificate-issue-counter-file pool1/opcert.counter
```

Generate VRF keys:

```bash
cardano-cli node key-gen-VRF \
--verification-key-file pool1/vrf.vkey \
--signing-key-file pool1/vrf.skey
```

Generate KES keys:

```bash
cardano-cli node key-gen-KES \
--verification-key-file pool1/kes.vkey \
--signing-key-file pool1/kes.skey
```

To generate the operational certificate:

```bash
cardano-cli node issue-op-cert \
--kes-verification-key-file pool1/kes.vkey \
--cold-signing-key-file pool1/cold.skey \
--operational-certificate-issue-counter pool1/opcert.counter \
--kes-period 0 \
--out-file pool1/opcert.cert
```

You can now create a topology file for the pool and update the BFT nodes' topologies so that they include the pool:  

```bash
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

```bash
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

```bash
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

Next, write a script to start the stake pool node:  

```bash
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

```bash
chmod +x pool1/startnode.sh
```

You should be ready to start the node from the pool1 directory.  

The pool cannot create blocks just yet. You need to register it to the blockchain.

## Registering the stake address

Create a registration certificate:

```bash
cardano-cli stake-address registration-certificate \
--stake-verification-key-file pool1/stake.vkey \
--out-file pool1/stake.cert
```

Registering a stake address requires a two ada deposit:  

```bash
 cardano-cli query protocol-parameters --testnet-magic 42 | grep stakeAddressDeposit   
     "stakeAddressDeposit": 2000000,
```

Use the `build-raw` command this time, note that it does not automatically calculate fees and change. Again, for simplicity, pay a one ada fee and add the two ada deposit. Use a variable for the change calculations:    

{% code overflow="wrap" %}
```bash
CHANGE=$(($(cardano-cli query utxo --address $(cat pool1/payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -cs '.[0] | to_entries | .[] | .value.value') - 3000000))
```
{% endcode %}

Build:

```bash
cardano-cli transaction build-raw \
--shelley-era \
--fee 1000000 \
--invalid-hereafter $(expr $(cardano-cli query tip --testnet-magic 42 | jq .slot) + 1000) \
--tx-in $(cardano-cli query utxo --address $(cat pool1/payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -r 'keys[]') \
--tx-out $(cat pool1/payment.addr)+$CHANGE \
--certificate-file pool1/stake.cert \
--out-file transactions/tx4.raw
```

Sign:

```bash
cardano-cli transaction sign \
--tx-body-file transactions/tx4.raw \
--signing-key-file pool1/payment.skey \
--signing-key-file pool1/stake.skey \
--testnet-magic 42 \
--out-file transactions/tx4.signed
```

Submit:

```bash
cardano-cli transaction submit \
--testnet-magic 42 \
--tx-file transactions/tx4.signed
```

## Registering a stake pool

Now, it's time to register the stake pool. On a real network, you would need to have metadata for your pool so that it could be properly displayed by wallets. It is not strictly necessary, but for learning purposes, you can use this same file [https://git.io/JJWdJ ](https://git.io/JJWdJ)from the documentation.

Get the file:

```bash
wget https://git.io/JJWdJ -O pool1/poolmetadata.json
```

Get the metadata hash and save it to a file:  

```bash
cardano-cli stake-pool metadata-hash \
--pool-metadata-file pool1/poolmetadata.json --out-file pool1/poolmetadata.hash
```

Generate the registration certificate:  

```bash
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
--metadata-hash $(cat pool1/poolmetadata.hash) \
--out-file pool1/pool-registration.cert
```

Create a delegation certificate to honor your pledge:  

```bash
cardano-cli stake-address delegation-certificate \
--stake-verification-key-file pool1/stake.vkey \
--cold-verification-key-file pool1/cold.vkey \
--out-file pool1/delegation.cert
```

Query the protocol parameters again to find the pool deposit:  

```bash
cardano-cli query protocol-parameters --testnet-magic 42 | grep stakePoolDeposit
    "stakePoolDeposit": 500000000,

```

Now, to make a 500 ada deposit, submit both the delegation certificate and the registration certificate:  

```bash
CHANGE=$(($(cardano-cli query utxo --address $(cat pool1/payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -cs '.[0] | to_entries | .[] | .value.value') - 501000000))
```

```bash
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

```bash
cardano-cli transaction sign \
--tx-body-file transactions/tx5.raw \
--signing-key-file pool1/payment.skey \
--signing-key-file pool1/stake.skey \
--signing-key-file pool1/cold.skey \
--testnet-magic 42 \
--out-file transactions/tx5.signed
```

```bash
cardano-cli transaction submit \
--testnet-magic 42 \
--tx-file transactions/tx5.signed
```

```bash
cardano-cli stake-pool id --cold-verification-key-file pool1/cold.vkey --output-format "hex"
66da34a16bd8582679442d045514ecd9817f24199e771d017ad7a8c2
```

Now you should wait for the next epoch transition for the stake snapshot to pick up your pool, and two epochs for your pool to start producing blocks. For this to work, you must lower the decentralization parameter (currently set to 1).   
