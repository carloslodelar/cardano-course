---
description: 'Objective: Create payment and stake keys, build addresses'
---

# Create keys and addresses

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
--testnet-magic 1
```
