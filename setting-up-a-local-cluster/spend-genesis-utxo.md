---
cover: ../.gitbook/assets/cluster (1).png
coverY: 0
---

# Spending the genesis UTXO

Create keys and addresses to withdraw the initial UTXO

```
cardano-cli keygen --secret utxo-keys/payment.000.key
```

```
cardano-cli signing-key-address --testnet-magic 42 \
--secret utxo-keys/payment.000.key > utxo-keys/payment.000.addr
```

Write genesis addresses to files  

```
cardano-cli signing-key-address \
    --testnet-magic 42 \
    --secret utxo-keys/byron.000.key > utxo-keys/byron.000.addr
```

It is time to spend the genesis UTXOs and send the funds to the regular Byron addresses created above. You will send funds from their genesis addresses `byron.000.addr` to your newly created regular `payment.000.addr`

{% code overflow="wrap" %}
```bash
cat utxo-keys/byron.000.addr

>
2657WMsDfac6JrXMvC5KQuYhCFfhoS5c1jfBPje9vn2D86PkugFZa5oMWcBJo1nrt
VerKey address with root 79ed4e30d7707c20a8c958fe89146fd21e1536e3d29a29e0131deabf, attributes: AddrAttributes { derivation path: {} }
```
{% endcode %}

Create a directory for your transaction files

```
mkdir transactions
```

And issue a genesis UTXO expenditure. Note that for simplicity the example is being generous with transaction fees. It will pay 1 ada or 1000000 lovelace.  

```
cardano-cli issue-genesis-utxo-expenditure \
--genesis-json configuration/byron-genesis.json \
--testnet-magic 42 \
--tx transactions/tx0.tx \
--wallet-key utxo-keys/byron.000.key \
--rich-addr-from $(head -n 1 utxo-keys/byron.000.addr) \
--txout "(\"$(head -n 1 utxo-keys/payment.000.addr)\", 29999999999000000)"
```

Submit the transaction:

```
cardano-cli submit-tx \
            --testnet-magic 42 \
            --tx transactions/tx0.tx
```
