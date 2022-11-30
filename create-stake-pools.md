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

```
cardano-cli transaction build \
--shelley-era \
--testnet-magic 42 \
--invalid-hereafter $(expr $(cardano-cli query tip --testnet-magic 42 | jq .slot) + 1000) \
--tx-in $(cardano-cli query utxo --address $(cat utxo-keys/user1.payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -r 'keys[]') \
--tx-out $(cat pool1/payment.addr)+99998000000 \
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
cardano-cli transaction build \
--shelley-era \
--testnet-magic 42 \
--invalid-hereafter $(expr $(cardano-cli query tip --testnet-magic 42 | jq .slot) + 1000) \
--tx-in $(cardano-cli query utxo --address $(cat pool1/payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -r 'keys[]') \
--certificate-file pool1/stake.cert \
--change-address $(cat pool1/payment.addr) \
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
