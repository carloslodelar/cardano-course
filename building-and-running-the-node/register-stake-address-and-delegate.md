---
cover: ../.gitbook/assets/CABAL (1).png
coverY: 0
---

# Registering a stake address and delegating

In this tutorial, you will learn how to delegate stake and get rewarded for it.&#x20;

First, you need to register your stake key on the blockchain. This action incurs a deposit of two ada according to the protocol parameters:&#x20;

```
cat pparams.json | grep stakeAddressDeposit
    "stakeAddressDeposit": 2000000,
```

To register the stake key, you first need to produce a registration certificate. Take a look at the `cardano-cli stake-address` command:

```bash
cardano-cli stake-address

Available commands:
  key-gen                  Create a stake address key pair
  build                    Build a stake address
  key-hash                 Print the hash of a stake address key.
  registration-certificate Create a stake address registration certificate
  deregistration-certificate
                           Create a stake address deregistration certificate
  delegation-certificate   Create a stake address delegation certificate
```

You'd be interested to use `registration-certificate`:

```bash
cardano-cli stake-address registration-certificate \
--stake-verification-key-file stake.vkey \
--out-file registration.cert
```

This is how it looks like:

```bash
cat registration.cert

{
    "type": "CertificateShelley",
    "description": "Stake Address Registration Certificate",
    "cborHex": "82008200581cb6590a7ef994cc48cb4d980ee85f56d6b3a24cfd5e594cc644f761d9"
}
```

Now you need to submit your certificate to the blockchain. Let's use the `build` command. This time, you need to use a few more options to build the transaction. We will use `--certificate-file` to include the registration certificate, and `--witness-override` to specify that this will be signed by two witnesses – `payment.skey` and `stake.skey`:

{% code overflow="wrap" %}
```bash
cardano-cli transaction build --babbage-era \
--testnet-magic 2 \
--witness-override 2 \
--tx-in $(cardano-cli query utxo --address $(cat paymentwithstake.addr) --testnet-magic 2 --out-file  /dev/stdout | jq -r 'keys[1]') \
--change-address $(cat paymentwithstake.addr) \
--certificate-file registration.cert \
--out-file tx.raw
```
{% endcode %}

Sign it using both keys:

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

You can now delegate your stake. For this, you need to create a delegation certificate. Let's take another look at the `cardano-cli stake-address` command:

```
cardano-cli stake-addres


Available commands:
  key-gen                  Create a stake address key pair
  build                    Build a stake address
  key-hash                 Print the hash of a stake address key.
  registration-certificate Create a stake address registration certificate
  deregistration-certificate
                           Create a stake address deregistration certificate
  delegation-certificate   Create a stake address delegation certificate
```

To produce the delegation certificate, you need to know the ID of the pool that you will delegate to. The example below delegates to the pool that processed the registration certificate:&#x20;

```
cardano-cli stake-address delegation-certificate \
--stake-verification-key-file stake.vkey \
--stake-pool-id ff7b882facd434ac990c4293aa60f3b8a8016e7ad51644939597e90c \
--out-file delegation.cert
```

Build, sign, and submit the transaction with the certificate:

{% code overflow="wrap" %}
```
cardano-cli transaction build --babbage-era \
--testnet-magic 2 \
--witness-override 2 \
--tx-in $(cardano-cli query utxo --address $(cat paymentwithstake.addr) --testnet-magic 2 --out-file  /dev/stdout | jq -r 'keys[1]') \
--change-address $(cat paymentwithstake.addr) \
--certificate-file delegation.cert \
--out-file tx.raw
```
{% endcode %}

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

## The delegation cycle

The delegation cycle consists of four epochs.&#x20;

The cycle begins when a stakeholder delegates their stake to a stake pool. This process involves the creation of a _**delegation certificate**_, which is subsequently included in a _**transaction**_ and _**registered**_ on the blockchain. In the example below, this occurs at any slot within _**Epoch N**_.&#x20;

Then, the protocol captures a stake snapshot (_**stakedist)**_ at the last slot of _**Epoch N,**_, recording three important pieces of information:&#x20;

1. The balance of each stake key registered on the blockchain
2. To which stake pool each stake key is delegated&#x20;
3. The parameters (pool cost, margin, pledge, etc) that each stake pool has set.&#x20;

This snapshot is then used at the end of _**Epoch N+1**_ to randomly select the slot leaders for _**Epoch N+2**_. This process is the essence of Ouroboros as a proof-of-stake consensus algorithm: the bigger the stake, the more chances to become a slot leader.  &#x20;

During _**Epoch N+2**_, stake pools produce the blocks they are entitled to based on the slot leader election. Naturally, stake pools with greater control of stake will be entitled to a larger number of blocks.

In the transition between _**Epoch N+2 and Epoch N+3,**_ a new snapshot registers the collected rewards. The protocol uses the stake distribution recorded at the end of _**Epoch N**_  to calculate how much of the rewards belong to each stake key.

Finally, during the transition between _**Epoch N+3**_ and _**Epoch N+4**_, the protocol distributes rewards to all stake keys. These rewards appear in wallets at the beginning of _**Epoch N+4**_, are credited to the wallet's balance, and are considered part of the stake for the snapshot taken at the final slot of _**Epoch N+4**_.

<figure><img src="../.gitbook/assets/delegationcycle.gif" alt=""><figcaption></figcaption></figure>
