# The Vasil Hardfork

The decentralization parameter is removed in Babbage era (Vasil Hardfork). This means that our BFT nodes will not be able to continue producing blocks in Babbage. We will need at least one more stake pool:

&#x20;

```
mkdir pool2
```

We will need an address and funds (for the pool owner):

```
cardano-cli address key-gen \
--verification-key-file pool2/payment.vkey \
--signing-key-file pool2/payment.skey
```

We will delegate our stake to our pool, so we will need stake keys

```
cardano-cli stake-address key-gen \
--verification-key-file pool2/stake.vkey \
--signing-key-file pool2/stake.skey
```

And we build our address with:

```
cardano-cli address build \
--payment-verification-key-file pool2/payment.vkey \
--stake-verification-key-file pool2/stake.vkey \
--out-file pool2/payment.addr \
--testnet-magic 42
```

Let's send some funds to our pool owner address:

```
cardano-cli transaction build \
--alonzo-era \
--testnet-magic 42 \
--invalid-hereafter $(expr $(cardano-cli query tip --testnet-magic 42 | jq .slot) + 1000) \
--tx-in $(cardano-cli query utxo --address $(cat utxo-keys/user1.payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -r 'keys[]') \
--tx-out $(cat pool1/payment.addr)+50000000000000 \
--change-address $(cat utxo-keys/user1.payment.addr) \
--out-file transactions/tx6.raw
```

Sign it

```
cardano-cli transaction sign \
--tx-body-file transactions/tx3.raw \
--signing-key-file utxo-keys/user1.payment.skey \
--testnet-magic 42 \
--out-file transactions/tx6.signed
```

Submit to the blockchain

```
cardano-cli transaction submit \
--testnet-magic 42 \
--tx-file transactions/tx6.signed
```

```
cardano-cli query utxo --address $(cat pool2/payment.addr) --testnet-magic 42
```

Generate cold keys

```
cardano-cli node key-gen \
--cold-verification-key-file pool2/cold.vkey \
--cold-signing-key-file pool2/cold.skey \
--operational-certificate-issue-counter-file pool2/opcert.counter
```

Generate VRF keys

```
cardano-cli node key-gen-VRF \
--verification-key-file pool2/vrf.vkey \
--signing-key-file pool2/vrf.skey
```

&#x20;Generate KES keys

```
cardano-cli node key-gen-KES \
--verification-key-file pool2/kes.vkey \
--signing-key-file pool2/kes.skey
```

To generate the operational certificate:

```
cardano-cli node issue-op-cert \
--kes-verification-key-file pool1/kes.vkey \
--cold-signing-key-file pool2/cold.skey \
--operational-certificate-issue-counter pool2/opcert.counter \
--kes-period 0 \
--out-file pool2/opcert.cert
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
     },
     {
       "addr": "127.0.0.1",
       "port": 3003,
       "valency": 1
     }
   ]
 }
EOF
```

```
cat > pool2/topology.json <<EOF
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
