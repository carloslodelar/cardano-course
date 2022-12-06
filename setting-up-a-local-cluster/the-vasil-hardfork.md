# The Vasil Hardfork

The decentralization parameter is removed in Babbage era (Vasil Hardfork). This means that our BFT nodes will not be able to continue producing blocks in Babbage. We will need at least one more stake pool:

```
mkdir pool2
```

Create keys and an address to send some funds to our second pool.

The payment keys

```
cardano-cli address key-gen \
--verification-key-file pool2/payment.vkey \
--signing-key-file pool2/payment.skey
```

The stake keys

```
cardano-cli stake-address key-gen \
--verification-key-file pool2/stake.vkey \
--signing-key-file pool2/stake.skey
```

Build the address

```
cardano-cli address build \
--payment-verification-key-file pool2/payment.vkey \
--stake-verification-key-file pool2/stake.vkey \
--out-file pool2/payment.addr \
--testnet-magic 42
```

Build the transactions to send some funds to our second pool owner

```
cardano-cli transaction build \
--alonzo-era \
--testnet-magic 42 \
--invalid-hereafter $(expr $(cardano-cli query tip --testnet-magic 42 | jq .slot) + 1000) \
--tx-in $(cardano-cli query utxo --address $(cat utxo-keys/user1.payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -r 'keys[]') \
--tx-out $(cat pool2/payment.addr)+50000000000000 \
--change-address $(cat utxo-keys/user1.payment.addr) \
--out-file transactions/tx6.raw
```

Sign it

```
cardano-cli transaction sign \
--tx-body-file transactions/tx6.raw \
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

The operational certificate:

```
cardano-cli node issue-op-cert \
--kes-verification-key-file pool2/kes.vkey \
--cold-signing-key-file pool2/cold.skey \
--operational-certificate-issue-counter pool2/opcert.counter \
--kes-period 0 \
--out-file pool2/opcert.cert
```

Create a topology file for pool2 and update pool1 topology. We will shut down bft0 and bft1 after the Vasil hardfork so we don't need to update them.&#x20;

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

And let's have a simple script to start the pool node.&#x20;

```
cat > pool2/startnode.sh <<EOF
#!/usr/bin/env bash

cardano-node run \
--config ../configuration/config.json \
--topology topology.json \
--database-path db \
--socket-path node.socket \
--port 3003 \
--shelley-kes-key kes.skey \
--shelley-vrf-key vrf.skey \
--shelley-operational-certificate opcert.cert
EOF
```

Give it executable permissions:

```
chmod +x pool2/startnode.sh
```

Start the node from the pool2 directory.

Our pool cannot create blocks just yet. We need to register it. To the blockchain

#### Register stake key

Create a registration certificate

```
cardano-cli stake-address registration-certificate \
--stake-verification-key-file pool2/stake.vkey \
--out-file pool2/stake.cert
```

Create a transaction to register the stake key

```
cardano-cli query utxo --address $(cat pool2/payment.addr) --testnet-magic 42
```

```
cardano-cli query protocol-parameters --testnet-magic 42 | grep stakeAddressDeposit
    "stakeAddressDeposit": 2000000,

```

{% code overflow="wrap" %}
```
CHANGE=$(($(cardano-cli query utxo --address $(cat pool2/payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -cs '.[0] | to_entries | .[] | .value.value.lovelace') - 3000000))
```
{% endcode %}

```
cardano-cli transaction build-raw \
--alonzo-era \
--fee 1000000 \
--invalid-hereafter $(expr $(cardano-cli query tip --testnet-magic 42 | jq .slot) + 1000) \
--tx-in $(cardano-cli query utxo --address $(cat pool2/payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -r 'keys[]') \
--tx-out $(cat pool2/payment.addr)+$CHANGE \
--certificate-file pool2/stake.cert \
--out-file transactions/tx7.raw
```

```
cardano-cli transaction sign \
--tx-body-file transactions/tx7.raw \
--signing-key-file pool2/payment.skey \
--signing-key-file pool2/stake.skey \
--testnet-magic 42 \
--out-file transactions/tx7.signed
```

```
cardano-cli transaction submit \
--testnet-magic 42 \
--tx-file transactions/tx7.signed
```

#### Register stake pool

Now, it's time to register the stake pool. On a real network we would need to have metadata for our pool so tat it can be properly displayed by wallets.&#x20;

The pool metadata must meet [this requirements](https://docs.cardano.org/development-guidelines/operating-a-stake-pool/public-stake-pools) &#x20;

We will use the same file [https://git.io/JJWdJ ](https://git.io/JJWdJ)

Get the file

```bash
wget https://git.io/JJWdJ -O pool2/poolmetadata.json
```

Get the metadata hash

```
cardano-cli stake-pool metadata-hash --pool-metadata-file pool2/poolmetadata.json
5c93c2c47f115c0184e5b2499b179b36dc385a45a2a809bb3ec2e34047dae04e
```

For connivence, lets save it to a file&#x20;

```
cardano-cli stake-pool metadata-hash --pool-metadata-file pool2/poolmetadata.json --out-file pool2/poolmetadata.hash
```

Generate the registration certificate

```
cardano-cli stake-pool registration-certificate \
--cold-verification-key-file pool2/cold.vkey \
--vrf-verification-key-file pool2/vrf.vkey \
--pool-pledge 1000000000000 \
--pool-cost 340000000 \
--pool-margin 10/100 \
--pool-reward-account-verification-key-file pool2/stake.vkey \
--pool-owner-stake-verification-key-file pool2/stake.vkey \
--testnet-magic 42 \
--pool-relay-ipv4 127.0.0.1 \
--pool-relay-port 3003 \
--metadata-url https://git.io/JJWdJ \
--metadata-hash $(cat pool2/poolmetadata.hash) \
--out-file pool2/pool-registration.cert
```

Create a delegation certificate

```
cardano-cli stake-address delegation-certificate \
--stake-verification-key-file pool2/stake.vkey \
--cold-verification-key-file pool2/cold.vkey \
--out-file pool2/delegation.cert
```

Now, let's query the protocol parameters again to find the pool deposit:

```bash
cardano-cli query protocol-parameters --testnet-magic 42 | grep stakePoolDeposit
    "stakePoolDeposit": 500000000,

```

So we need to make a deposit of 500 test ADA

Let's submit both, the delegation certificate and the registration certificate:&#x20;

```
CHANGE=$(($(cardano-cli query utxo --address $(cat pool2/payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -cs '.[0] | to_entries | .[] | .value.value.lovelace') - 501000000))
```

```
cardano-cli transaction build-raw \
--alonzo-era \
--fee 1000000 \
--invalid-hereafter $(expr $(cardano-cli query tip --testnet-magic 42 | jq .slot) + 10000) \
--tx-in $(cardano-cli query utxo --address $(cat pool2/payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -r 'keys[]') \
--tx-out $(cat pool2/payment.addr)+$CHANGE \
--certificate-file pool2/pool-registration.cert \
--certificate-file pool2/delegation.cert \
--out-file transactions/tx8.raw
```

```
cardano-cli transaction sign \
--tx-body-file transactions/tx8.raw \
--signing-key-file pool2/payment.skey \
--signing-key-file pool2/stake.skey \
--signing-key-file pool2/cold.skey \
--testnet-magic 42 \
--out-file transactions/tx8.signed
```

```
cardano-cli transaction submit \
--testnet-magic 42 \
--tx-file transactions/tx8.signed
```

```
cardano-cli stake-pool id --cold-verification-key-file pool2/cold.vkey --output-format "hex"
```

Wait 2 epoch for pool2 to start producing blocks&#x20;

#### Let's bring decentralization down to 0

```
cardano-cli governance create-update-proposal \
--out-file transactions/update.D0.proposal \
--epoch $(cardano-cli query tip --testnet-magic 42 | jq .epoch) \
--genesis-verification-key-file genesis-keys/non.e.shelley.000.vkey \
--genesis-verification-key-file genesis-keys/non.e.shelley.001.vkey \
--decentralization-parameter 0/100
```

```
CHANGE=$(($(cardano-cli query utxo --address $(cat pool1/payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -cs '.[0] | to_entries | .[] | .value.value.lovelace') - 1000000))
```

```
cardano-cli transaction build-raw \
--shelley-era \
--fee 1000000 \
--invalid-hereafter $(expr $(cardano-cli query tip --testnet-magic 42 | jq .slot) + 1000) \
--tx-in $(cardano-cli query utxo --address $(cat pool1/payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -r 'keys[]') \
--tx-out $(cat pool1/payment.addr)+$CHANGE \
--update-proposal-file transactions/update.D0.proposal \
--out-file transactions/update.D0.proposal.txbody
```

```
cardano-cli transaction sign \
--tx-body-file transactions/update.D0.proposal.txbody \
--signing-key-file pool1/payment.skey \
--signing-key-file bft0/shelley.000.skey \
--signing-key-file bft1/shelley.001.skey \
--out-file transactions/update.D0.proposal.txsigned
```

```
cardano-cli transaction submit --testnet-magic 42 --tx-file transactions/update.D0.proposal.txsigned
```

{% code overflow="wrap" %}
```
Event: LedgerUpdate (HardForkUpdateInEra S (S (S (S (Z (WrapLedgerUpdate {unwrapLedgerUpdate = ShelleyUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateProposal = UpdateProposal {proposalParams = PParams {_minfeeA = SNothing, _minfeeB = SNothing, _maxBBSize = SNothing, _maxTxSize = SNothing, _maxBHSize = SNothing, _keyDeposit = SNothing, _poolDeposit = SNothing, _eMax = SNothing, _nOpt = SNothing, _a0 = SNothing, _rho = SNothing, _tau = SNothing, _d = SJust (0 % 1), _extraEntropy = SNothing, _protocolVersion = SNothing, _minPoolCost = SNothing, _coinsPerUTxOWord = SNothing, _costmdls = SNothing, _prices = SNothing, _maxTxExUnits = SNothing, _maxBlockExUnits = SNothing, _maxValSize = SNothing, _collateralPercentage = SNothing, _maxCollateralInputs = SNothing}, proposalVersion = Nothing, proposalEpoch = EpochNo 37}, protocolUpdateState = UpdateState {proposalVotes = [KeyHash "0bc4c92289432762de7910d9db34ac709228581aea448d0bc3cc8fa6",KeyHash "a9bd5e3a633a9b46a889f974bfea91af8e4d0af9595a9bcd1537472c"], proposalReachedQuorum = True}}]}))))))
```
{% endcode %}

#### The hardfork

```
sed -i configuration/config.json \
-e 's/LastKnownBlockVersion-Major":6/LastKnownBlockVersion-Major":7/'
```

```
cardano-cli governance create-update-proposal \
--genesis-verification-key-file genesis-keys/non.e.shelley.000.vkey \
--genesis-verification-key-file genesis-keys/non.e.shelley.001.vkey \
--out-file transactions/update.v7.proposal \
--epoch $(cardano-cli query tip --testnet-magic 42 | jq .epoch) \
--protocol-major-version "7" \
--protocol-minor-version "0" 
```

```
CHANGE=$(($(cardano-cli query utxo --address $(cat pool1/payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -cs '.[0] | to_entries | .[] | .value.value.lovelace') - 1000000))
```

```
cardano-cli transaction build-raw \
--alonzo-era \
--fee 1000000 \
--invalid-hereafter $(expr $(cardano-cli query tip --testnet-magic 42 | jq .slot) + 1000) \
--tx-in $(cardano-cli query utxo --address $(cat pool1/payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -r 'keys[]') \
--tx-out $(cat pool1/payment.addr)+$CHANGE \
--update-proposal-file transactions/update.v7.proposal \
--out-file transactions/update.v7.proposal.txbody
```

```
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

```
{
    "block": 17002,
    "epoch": 38,
    "era": "Babbage",
    "hash": "6d5d9c23c0679e4005283c9bc27faf487b0407f454
1a40c8c5ce7a623cee0831",
    "slot": 327374,
    "syncProgress": "100.00"
}
```
