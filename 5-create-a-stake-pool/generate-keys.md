# Generate keys

So we will need the following:

| `payment.vkey`   | payment verification key         |
| ---------------- | -------------------------------- |
| `payment.skey`   | payment signing key              |
| `payment.addr`   | funded address linked to `stake` |
| `stake.vkey`     | staking verification key         |
| `stake.skey`     | staking signing key              |
| `stake.addr`     | registered stake address         |
| `cold.skey`      | cold signing key                 |
| `cold.vkey`      | cold verification key            |
| `kes.skey`       | KES signing key                  |
| `kes.vkey`       | KES verification key             |
| `vrf.skey`       | VRF signing key                  |
| `vrf.vkey`       | VRF verification key             |
| `opcert.cert`    | operational certificate          |
| `opcert.counter` | issue counter                    |

#### Payment and stake keys

{% tabs %}
{% tab title="With the  cardano CLI" %}
Generate payment  keys

```bash
cardano-cli address key-gen \
--verification-key-file payment.vkey \
--signing-key-file payment.skey
```

Generate stake keys

```bash
cardano-cli stake-address key-gen \
--verification-key-file stake.vkey \
--signing-key-file stake.skey
```

Generate payment address&#x20;

```bash
cardano-cli address build \
--payment-verification-key-file payment.vkey \
--stake-verification-key-file stake.vkey \
--testnet-magic 2 \
--out-file payment.addr
```
{% endtab %}

{% tab title="From recovery phrase" %}
```
```
{% endtab %}
{% endtabs %}
