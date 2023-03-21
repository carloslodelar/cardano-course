---
description: >-
  Objective:  Understand generation of certificates and creating transactions
  that include certificates
cover: ../../.gitbook/assets/CABAL (1).png
coverY: 0
---

# Register stake address and delegate

Let's do somehting a little bit more interesting. Let's delegate our stake so that we can participate in the protocol and get rewarded for it.&#x20;

We first need to register our stake key to the blockhain. This action incurs in a deposit of 2 ADA according to the protocol parameters&#x20;

```
cat pparams.json | grep stakeAddressDeposit
    "stakeAddressDeposit": 2000000,
```

To register our stake key, we first need to produce a registration certificate. Take a look to the `cardano-cli stake-address` command

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

We are interested in `registration-certificate`

```bash
cardano-cli stake-address registration-certificate \
--stake-verification-key-file stake.vkey \
--out-file registration.cert
```

Let's take a look of how it looks like

```bash
cat registration.cert

{
    "type": "CertificateShelley",
    "description": "Stake Address Registration Certificate",
    "cborHex": "82008200581cb6590a7ef994cc48cb4d980ee85f56d6b3a24cfd5e594cc644f761d9"
}
```

Now we need to submit our certificate to the blockchain. We will use the `build` command. This time we need to use a few more options to build the transaction. We will use `--certificate-file` to include our registration certificate, and `--witness-override` to specify that this will be signed by 2 witnesses: `payment.skey` and `stake.skey`

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

Sign it with both keys

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

Now we can delegate our stake. For that, we need to create a delegation certificate. Let's take another look to the `cardano-cli stake-address` command

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

To produce the delegation certificate we need to know the stake pool id of the pool that we will delegate to. In this case I'll delegate to the pool that processed my registration certificate.&#x20;

```
cardano-cli stake-address delegation-certificate \
--stake-verification-key-file stake.vkey \
--stake-pool-id ff7b882facd434ac990c4293aa60f3b8a8016e7ad51644939597e90c \
--out-file delegation.cert
```

~~Rinse and repeat.~~ Build, sign and submit the transaction with the certificate

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

#### The delegation cycle

The delegation cycle comprises four (4) epochs:&#x20;

The cycle starts when the stakeholder delegates his stake to a stake pool. This involves the creation of a _**Delegation Certificate**_ that is then embedded in a _**transaction**_ and _**registered**_ in the blockchain.  In our example below, this happens at any slot of _**Epoch N**_.&#x20;

Then, **** the protocol captures a stake snapshot (_**stakedist)**_ at the last slot of _**Epoch N,**_ recording 3 important pieces of information:&#x20;

1. The balance of each stake key registered in the blockchain,
2. To which stake pool each stake key is delegated,&#x20;
3. The parameters (pool cost, margin, pledge, etc) that each stake pool has set.&#x20;

This snapshot is then used at the end of _**Epoch N+1**_ to randomly select the slot leaders for _**Epoch N+2**_. This process is the essence of Ouroboros as a Proof-of-Stake consensus algorithm:  the bigger the stake, the more chances to be slot leader.  &#x20;

During _**Epoch N+2**_ the stake pools produce the blocks they are entitled to as per the slot leader election. Naturally, stake pools that control more stake will be entitled to more blocks.

In the transition between _**Epoch N+2 and Epoch N+3,**_ a new snapshot registers the rewards collected. And the protocol uses the stake distribution recorded at the end of _**Epoch N**_  to calculate how much of the rewards belongs to each stake key,

Finally, at the transition between _**Epoch N+3**_ and _**Epoch N+4**_  the protocol distributes the rewards to all stake keys. Which show up in wallets at the very start of _**Epoch N+4**_ and credited to the wallet's balance and will count as part of the stake for the snapshot to be taken at the last slot of _**Epoch N+4**_

<figure><img src="../../.gitbook/assets/delegationcycle.gif" alt=""><figcaption></figcaption></figure>
