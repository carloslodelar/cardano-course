---
cover: ../.gitbook/assets/CABAL (1).png
coverY: 0
---

# Creating keys and addresses

## A quick look at addresses

{% embed url="https://cips.cardano.org/cips/cip19/" %}

Create a directory for your keys:

```
mkdir -p keys
cd keys
```

## Generating a payment key pair and an address

To generate a key pair, run:

```
cardano-cli address key-gen \
--verification-key-file payment.vkey \
--signing-key-file payment.skey
```

## Building an address

Generate a type 6 address, the associated stake of which cannot be delegated:

```
cardano-cli address build \
--payment-verification-key-file payment.vkey \
--out-file payment.addr \
--testnet-magic 2
```

## Installing cardano-address

Just for fun, let's install `cardano-address`&#x20;

{% embed url="https://github.com/input-output-hk/cardano-addresses/releases" %}

Go to the previously created `src` directory:

```
wget https://github.com/input-output-hk/cardano-addresses/releases/download/3.12.0/cardano-addresses-3.12.0-linux64.tar.gz
```

```
tar -xf cardano-addresses-3.12.0-linux64.tar.gz
```

```
chmod +x bin/cardano-address
```

```
mv bin/cardano-address ~/.local/bin/
```

You can use `cardano-address` to inspect the address. Back to the keys directory&#x20;

```
cat payment.addr | cardano-address address inspect
```

you should see something like:

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

Now, you can get some funds from the faucet:

{% embed url="https://docs.cardano.org/cardano-testnet/tools/faucet" %}

## Querying the address balance

If successful, you can use `cardano-cli` to verify that you have received the funds:

```
cardano-cli query utxo --address $(cat payment.addr) --testnet-magic 2
```

cardanoscan.io will also show the transaction:

{% embed url="https://preprod.cardanoscan.io/" %}

## Generating a stake key pair and a type 0 address

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

You should see something like this:

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
