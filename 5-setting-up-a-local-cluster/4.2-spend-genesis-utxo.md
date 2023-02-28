---
cover: ../.gitbook/assets/cluster (1).png
coverY: 0
---

# 4.2 Spend genesis UTXO

Create keys and addresses to withdraw the initial UTxO

```
cardano-cli keygen --secret utxo-keys/payment.000.key
```

```
cardano-cli signing-key-address --testnet-magic 42 \
--secret utxo-keys/payment.000.key > utxo-keys/payment.000.addr
```

Write genesis addresses to files&#x20;

```
cardano-cli signing-key-address \
    --testnet-magic 42 \
    --secret utxo-keys/byron.000.key > utxo-keys/byron.000.addr
```

It is time to spend the genesis utxos and send the funds to the regular byron addresses we created above. We send funds from their genesis addresses `byron.000.addr` to our newly created regular `payment.000.addr`

{% code overflow="wrap" %}
```bash
cat utxo-keys/byron.000.addr

>
2657WMsDfac6JrXMvC5KQuYhCFfhoS5c1jfBPje9vn2D86PkugFZa5oMWcBJo1nrt
VerKey address with root 79ed4e30d7707c20a8c958fe89146fd21e1536e3d29a29e0131deabf, attributes: AddrAttributes { derivation path: {} }
```
{% endcode %}

Create a directory for our transaction files

```
mkdir transactions
```

And issue a genesis utxo expenditure, note that for simplicity we are being generous with transaction fees. We will pay 1 ADA or 1000000 lovelace.&#x20;

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
