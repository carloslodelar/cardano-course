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

####

{% hint style="info" %}
WE WILL GENERATE KEYS IN OUR AIR-GAPPED MACHINE.&#x20;
{% endhint %}

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
Generate a recovery phrase and save it to a file&#x20;

```
cardano-address recovery-phrase generate > recoveryphrase.txt
```

Get the root private key

```
cat recoveryphrase.txt | cardano-address key from-recovery-phrase Shelley > root.prv
```

Generate the private and public keys for the index 0 address

```
cardano-address key child 1852H/1815H/0H/0/0    < root.prv > payment-0.prv
cardano-address key public --without-chain-code < payment-0.prv > payment-0.pub
```

Convert the private key so that it can be used within the cardano-cli

```
cardano-cli key convert-cardano-address-key \
--shelley-payment-key \
--signing-key-file payment-0.prv \
--out-file payment-0.skey
```

Get the public (verification) key compatible with cardano-cli

```
cardano-cli key verification-key \
--signing-key-file payment-0.skey \
--verification-key-file payment-0.vkey
```

&#x20;Generate the stake keys

```
cardano-address key child 1852H/1815H/0H/2/0 < root.prv > stake.prv
cardano-address key public --without-chain-code < stake.prv > stake.pub
```

Convert it&#x20;

```
cardano-cli key convert-cardano-address-key \
--shelley-stake-key \
--signing-key-file stake.prv \
--out-file stake.skey
```

Get the extended verification key

```
cardano-cli key verification-key \
--signing-key-file stake.skey \
--verification-key-file stake.evkey
```

And the non-extended verification key&#x20;

```
cardano-cli key non-extended-key \
--extended-verification-key-file stake.evkey \
--verification-key-file stake.vkey
```

Finally, generate payment address

```
cardano-cli address build --testnet-magic 2
--payment-verification-key $(cat payment-0.pub)
--stake-verification-key $(cat stake.pub)
--out-file payment-0.address
```

And the stake address

```
cardano-cli stake-address build --testnet-magic 2 \
--stake-verification-key-file stake.pub \
--out-file stake.address
```


{% endtab %}
{% endtabs %}
