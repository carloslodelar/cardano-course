# Pool Operations/Maintenance

### Query stake snapshot

```bash
cardano-cli query stake-snapshot \
--testnet-magic 2 \
--stake-pool-id <pool_id>
```

### Query leadership schedule

```bash
cardano-cli query leadership-schedule \
--testnet-magic 2 \
--cold-verification-key-file cold.vkey \
--genesis shelley-genesis.json \
--vrf-signing-key-file vrf.skey \
--current
```

```bash
cardano-cli query leadership-schedule \
--testnet-magic 2 \
--cold-verification-key-file cold.vkey \
--genesis shelley-genesis.json \
--vrf-signing-key-file vrf.skey \
--next
```

### Check the validity of your KES keys:

```bash
cardano-cli query kes-period-info \
--testnet-magic 2 \
--op-cert-file opcert.cert
```

### Renew KES keys and operational certificate

```bash
cardano-cli node key-gen-KES \
  --verification-key-file kes.vkey \
  --signing-key-file kes.skey
```

```bash
  cardano-cli node issue-op-cert --kes-verification-key-file kes.vkey \
  --cold-signing-key-file cold.skey \
  --operational-certificate-issue-counter-file opcert.counter \
  --kes-period <current kes period> \
  --out-file opcert.cert
```

### Withdraw rewards

Build the stake address

```bash
cardano-cli stake-address build \
--testnet-magic 2 \
--stake-verification-key-file stake.vkey \
--out-file stake.addr
```

Query its balance

```bash
cardano-cli query stake-address-info \
--testnet-magic 2 \
--address $(cat stake.addr)
```

example output&#x20;

```
[
    {
        "address": "stake_test1urdflexy7e3uuflpvs0zycrc7zc8rtg9hguglvwq545j7mqf09vna",
        "delegation": "pool15r46gslrnhpe7ekvecfl895sm2pzhsxa8dhwh9ymgf06st3l86e",
        "rewardAccountBalance": 1834628277
    }
]
```

Build a transaction to withdraw rewards, use `--witness-override 2.` It will be signed by stake and payment keys

<pre><code>cardano-cli transaction build \
--testnet-magic 2 \
<strong>--witness-override 2 \
</strong><strong>--tx-in $(cardano-cli query utxo --address $(cat payment.addr) --testnet-magic 2 --out-file  /dev/stdout | jq -r 'keys[0]') \
</strong>--withdrawal $(cat stake.addr)+1834628277 \
--change-address $(cat payment.addr) \
--out-file withdraw_tx.raw
</code></pre>

```
cardano-cli transaction sign \
--tx-body-file withdraw_tx.raw  \
--signing-key-file payment.skey \
--signing-key-file stake.skey \
--testnet-magic 2 \
--out-file withdraw_tx.signed
```

```
cardano-cli transaction submit \
--tx-file withdraw_rewards.signed \
--testnet-magic 2
```

### Changing pool parameters

Updating pool parameters is done with a new registration certificate. This time you will not pay the 500 ADA deposit.&#x20;

```bash
cardano-cli stake-pool registration-certificate \
--testnet-magic 2 \
--cold-verification-key-file cold.vkey \
--vrf-verification-key-file vrf.vkey \
--pool-pledge <AMOUNT TO PLEDGE IN LOVELACE> \
--pool-cost <POOL COST PER EPOCH IN LOVELACE> \
--pool-margin <POOL OPERATOR MARGIN > \
--pool-reward-account-verification-key-file stake.vkey \
--pool-owner-stake-verification-key-file stake.vkey \
--pool-relay-ipv4 <RELAY NODE PUBLIC IP> \
--pool-relay-port <RELAY NODE PORT> \
--metadata-url <URL> \
--metadata-hash <POOL METADATA HASH> \
--out-file pool-registration.cert
```

Submit `pool-registration.cert` in a transaction.&#x20;

On a machine with a running node:

```
cardano-cli transaction build \
--testnet-magic 2 \
--tx-in $(cardano-cli query utxo --address $(cat payment.addr) --testnet-magic 2 --out-file  /dev/stdout | jq -r 'keys[0]') \
--change-address $(cat payment.addr) \
--witness-override 3 \
--certificate-file pool-registration.cert \
--out-file tx.raw
```

Sign the `tx.raw` on your air-gapped machine

```
cardano-cli transaction sign \
--testnet-magic 2 \
--tx-body-file tx.raw \
--signing-key-file payment.skey \
--signing-key-file stake.skey \
--signing-key-file cold.skey \
--out-file tx.signed
```

Back on the machine with the running node, submit `tx.signed`&#x20;

```
cardano-cli transaction submit \
--testnet-magic 2 \
--tx-file tx.sigend 
```

### Update cardano-node and cardano-cli

1. Build cardano-node and cardano-cli on you local machine
2. Upload the new binaries to your node server

```
scp cardano-node cardano-cli user@host:~/ 
```

3. Stop your node

```
sudo systemctl stop cardano-node.service
```

4. Replace the old binaries with the new ones.

```
mv cardano* /usr/local/bin 
```

5. Restart the node

```
sudo systemctl start cardano-node.service
```

### Retire a stake pool

{% hint style="info" %}
Pool deposit is refunded to rewards address. do not unregister the stake address before you get the deposit back. \
\
Make sure to read  [Retire a stake pool](https://github.com/input-output-hk/cardano-node/blob/master/doc/stake-pool-operations/12\_retire\_stakepool.md#retiring-a-stake-pool)
{% endhint %}

```
cardano-cli stake-pool deregistration-certificate \
  --cold-verification-key-file cold.vkey \
  --epoch <future epoch> \
  --out-file pool-deregistration.cert
```

```
cardano-cli transaction build \
--testnet-magic 2 \
--tx-in $(cardano-cli query utxo --address $(cat payment.addr) --testnet-magic 2 --out-file  /dev/stdout | jq -r 'keys[0]') \
--change-address $(cat payment.addr) \
--witness-override 2 \
--certificate-file  pool-deregistration.cert \
--out-file tx.raw
```

```
cardano-cli transaction sign \
--testnet-magic 2 \
--tx-body-file tx.raw \
--signing-key-file payment.skey \
--signing-key-file cold.skey \
--out-file tx.signed
```

```
cardano-cli transaction submit \
--testnet-magic 2 \
--tx-file tx.signed
```

