# Bring decentralization parameter down to .80

We will create an update proposal to lower the decentralization parameter. This way, our pool stake will start producing blocks.

Let's create the proposal:

```
cardano-cli governance create-update-proposal \
--out-file transactions/update.D80.proposal \
--epoch $(cardano-cli query tip --testnet-magic 42 | jq .epoch) \
--genesis-verification-key-file genesis-keys/non.e.shelley.000.vkey \
--genesis-verification-key-file genesis-keys/non.e.shelley.001.vkey \
--decentralization-parameter 80/100
```





