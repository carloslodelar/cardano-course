# Registering a stake pool

## [Register the stake address](https://github.com/input-output-hk/cardano-node-wiki/blob/main/docs/stake-pool-operations/5_register_key.md)

```
cardano-cli stake-address registration-certificate \
--stake-verification-key-file stake.vkey \
--out-file registration.cert
```

```
cardano-cli transaction build \
--testnet-magic 2 \
--witness-override 2 \
--tx-in $(cardano-cli query utxo --address $(cat payment.addr) --testnet-magic 2 --out-file  /dev/stdout | jq -r 'keys[0]') \
--change-address $(cat payment.addr) \
--certificate-file registration.cert \
--out-file tx.raw
```

```
cardano-cli transaction sign \
--tx-body-file tx.raw \
--signing-key-file payment.skey \
--signing-key-file stake.skey \
--testnet-magic 2 \
--out-file tx.signed
```

```
cardano-cli transaction submit \
--testnet-magic 2 \
--tx-file tx.signed 
```

## Register a [stake pool with metadata](https://github.com/input-output-hk/cardano-node-wiki/blob/main/docs/stake-pool-operations/8_register_stakepool.md)

The stake pool registration certificate optionally contains a content hash and a URL (up to 64 bytes).

* JSON containing:
  * **Name** of up to 50 characters&#x20;
  * **Description** of up to 255 characters
  * **Ticker** of 3-5 characters
  * **Homepage** with additional information about the pool
* All characters in the metadata will be UTF8 encoded
* No more than 512 bytes

```
  {
  "name": "TestPool",
  "description": "The pool that tests all the pools",
  "ticker": "TEST",
  "homepage": "https://teststakepool.com"
  }
```

For example: [https://880w.short.gy/clrsp.json](https://880w.short.gy/clrsp.json)

To get the metadata hash, run:

```
wget https://880w.short.gy/clrsp.json
```

```
cardano-cli stake-pool metadata-hash --pool-metadata-file clrsp.json
3c914463aa1cddb425fba48b21c4db31958ea7a30e077f756a82903f30e04905
```

## [Generate a stake pool registration certificate](https://github.com/input-output-hk/cardano-node-wiki/blob/main/docs/stake-pool-operations/8_register_stakepool.md#generate-stake-pool-registration-certificate)

### Create the registration certificate:

```
cardano-cli stake-pool registration-certificate \
--cold-verification-key-file cold.vkey \
--vrf-verification-key-file vrf.vkey \
--pool-pledge 9000000000 \
--pool-cost 340000000 \
--pool-margin 0.05 \
--pool-reward-account-verification-key-file stake.vkey \
--pool-owner-stake-verification-key-file stake.vkey \
--testnet-magic 2 \
--pool-relay-ipv4 <RELAY NODE PUBLIC IP> \
--pool-relay-port <RELAY NODE PORT> \
--metadata-url https://git.io/JJWdJ \
--metadata-hash <POOL METADATA HASH> \
--out-file pool-registration.cert
```

### Create the delegation certificate to honor the pledge:

```
cardano-cli stake-address delegation-certificate \
--stake-verification-key-file stake.vkey \
--cold-verification-key-file cold.vkey \
--out-file delegation.cert
```

### Submit certificates:

```
cardano-cli transaction build \
--testnet-magic 2 \
--witness-override 3 \
--tx-in $(cardano-cli query utxo --address $(cat payment.addr) --testnet-magic 2 --out-file  /dev/stdout | jq -r 'keys[1]') \
--change-address $(cat payment.addr) \
--certificate-file pool-registration.cert \
--certificate-file delegation.cert \
--out-file tx.raw
```

```
cardano-cli transaction sign \
--tx-body-file tx.raw \
--signing-key-file payment.skey \
--signing-key-file cold.skey \
--signing-key-file stake.skey \
--testnet-magic 2 \
--out-file tx.signed
```

```
cardano-cli transaction submit \
--testnet-magic 2 \
--tx-file tx.signed 
```

### [Get delegation funds from the testnet faucet](https://docs.cardano.org/cardano-testnet/tools/faucet)

The faucet only takes the Bech32 pool ID:

```
cardano-cli stake-pool id \
--cold-verification-key-file cold.vkey \
--output-format bech32
```

### [Issue an operational certificate](https://github.com/input-output-hk/cardano-node-wiki/blob/main/docs/stake-pool-operations/7_KES_period.md)

#### Calculate the current KES period.

There are 129600 slots in a KES period:

```
cat shelley-genesis.json | grep KES
"slotsPerKESPeriod": 129600,
"maxKESEvolutions": 62,
```

The current slot can be obtained using a query tip:

```
cardano-cli query tip --testnet-magic 2 

>
{
    "block": 552276,
    "epoch": 143,
    "era": "Babbage",
    "hash": "fe9c9b4b3f70262f60183bf0d8e72077c559345b22b47d4d4af91d4e5a4b5994",
    "slot": 12380321,
    "syncProgress": "100.00"
}
```

Divide:

```
echo $((12380321 / 129600))
```

Or in one line:

```
echo $(($(cardano-cli query tip --testnet-magic 2 | jq .slot) / 129600))
```

On the air-gapped machine, run:

```
cardano-cli node issue-op-cert --kes-verification-key-file kes.vkey \
--cold-signing-key-file cold.skey \
--operational-certificate-issue-counter-file opcert.counter \
--kes-period <current kes period> \
--out-file opcert.cert
```



