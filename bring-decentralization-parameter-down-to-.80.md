# Bring decentralization parameter down to .80

We will create an update proposal to lower the decentralization parameter. This way, our pool stake will start producing blocks.

Let's create the proposal:

```
cardano-cli governance create-update-proposal \
--out-file transactions/update.D80.proposal \
--epoch $(cardano-cli query tip --testnet-magic 42 | jq .epoch) \
--genesis-verification-key-file bft0/shelley.000.vkey \
--genesis-verification-key-file bft1/shelley.001.vkey \
--decentralization-parameter 80/100
```

```
cardano-cli transaction build \
--tx-in $(cardano-cli query utxo --address $(cat pool1/payment.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -r 'keys[]') \ \
--change-address $(cat pool1/payment.addr) \
--update-proposal-file transactions/update.D80.proposal \
--testnet-magic 42 \
--out-file transactions/update.D80.proposal.txbody
```

```
cardano-cli transaction sign \
--tx-body-file transactions/update.D80.proposal.txbody \
--signing-key-file pool1/payment.skey \
--signing-key-file bft0/shelley.000.skey \
--signing-key-file bft1/shelley.001.skey \
--out-file transactions/update.D80.proposal.txsigned
```

```
cardano-cli transaction submit \
--testnet-magic 42 \ 
--tx-file transactions/update.D80.proposal.txsigned
```
