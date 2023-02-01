---
description: >-
  Objective:  Understand generation of certificates and creating transactions
  that include certificates
cover: .gitbook/assets/RUNNING (1).png
coverY: 361
---

# 1.5 Register stake address and delegate

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
--tx-in $(cardano-cli query utxo --address $(cat paymentwithstake.addr) --testnet-magic 1 --out-file  /dev/stdout | jq -r 'keys[1]') \
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

```
cardano-cli transaction build --babbage-era \
--testnet-magic 2 \
--witness-override 2 \
--tx-in $(cardano-cli query utxo --address $(cat paymentwithstake.addr) --testnet-magic 1 --out-file  /dev/stdout | jq -r 'keys[1]') \
--change-address $(cat paymentwithstake.addr) \
--certificate-file delegation.cert \
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
