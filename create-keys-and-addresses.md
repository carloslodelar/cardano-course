---
description: 'Objective: Create payment and stake keys, build addresses'
---

# 1.3 Create keys and addresses

### A quick look to addresses

{% embed url="https://cips.cardano.org/cips/cip19/" %}

Create keys and addresses&#x20;

```bash
cardano-cli
```

```
cardano-cli address
```

```
cardano-cli address key-gen
```

Let's create a payment key pair

Lets create a directory for our keys:

```
mkdir -p keys
cd keys
```

### Generate a payment key pair and address

```
cardano-cli address key-gen \
--verification-key-file payment.vkey \
--signing-key-file payment.skey
```

Let's generate a type 6 address, one which associated stake can't be delegated.

```
cardano-cli address build \
--payment-verification-key-file payment.vkey \
--out-file payment.addr \
--testnet-magic 1
```

Just for fun, lets install `cardano-address`&#x20;

{% embed url="https://github.com/input-output-hk/cardano-addresses/releases" %}

Go to the src directory we created before

```
wget https://github.com/input-output-hk/cardano-addresses/releases/download/3.12.0/cardano-addresses-3.12.0-linux64.tar.gz
```

```
tar -xf cardano-addresses-3.12.0-linux64.tar.gzcodecode
```

```
chmod +x bin/cardano-address
```

```
mv bin/cardano-address ~/.local/bin/
```

We can use `cardano-address` to inspect our address. Back to the keys directory&#x20;

```
cat payment.addr | cardano-address address inspect
```

we should see something close to

```json
{
    "stake_reference": "none",
    "spending_key_hash_bech32": "addr_vkh1tc0da5jwszr8f875jcpdmx3lmfgfcuxxvexggufyvmnhg3r5p2x",
    "address_style": "Shelley",
    "spending_key_hash": "5e1eded24e8086749fd49602dd9a3fda509c70c6664c84712466e774",
    "network_tag": 0,
    "address_type": 6
}
```

Now, lets get some funds from the faucet:

{% embed url="https://docs.cardano.org/cardano-testnet/tools/faucet" %}

When sucesfull we can cardano-cli to verify that we have received the funds:

```
cardano-cli query utxo --address $(cat payment.addr) --testnet-magic 1
```

cardanoscan.io will also show the transaction

{% embed url="https://preprod.cardanoscan.io/" %}

### Generate a stake key pair and a type 0 address

```
cardano-cli stake-address key-gen \
--verification-key-file stake.vkey \
--signing-key-file stake.skey
```

```
cardano-cli address build \
--payment-verification-key-file payment.vkey \
--stake-verification-key-file stake.vkey \
--out-file paymentwithstake.addr \
--testnet-magic 2
```

```bash
cat paymentwithstake.addr | cardano-address address inspect
```

It should look something like this

```json
{
    "stake_reference": "by value",
    "stake_key_hash_bech32": "stake_vkh1kevs5lhejnxy3j6dnq8wsh6k66e6yn8atev5e3jy7asajygx8zx",
    "stake_key_hash": "b6590a7ef994cc48cb4d980ee85f56d6b3a24cfd5e594cc644f761d9",
    "spending_key_hash_bech32": "addr_vkh1tc0da5jwszr8f875jcpdmx3lmfgfcuxxvexggufyvmnhg3r5p2x",
    "address_style": "Shelley",
    "spending_key_hash": "5e1eded24e8086749fd49602dd9a3fda509c70c6664c84712466e774",
    "network_tag": 0,
    "address_type": 0
}
```
