# Shelley hardfork

We don't want to stay on Byron era forever, right? Of course not. Let's move on to Shelley era

Create keys and addresses to withdraw the initial UTxO

```
cardano-cli keygen --secret utxo-keys/payment.000.key
cardano-cli keygen --secret utxo-keys/payment.001.key
```

```
cardano-cli signing-key-address --testnet-magic 42 \
--secret utxo-keys/payment.000.key > utxo-keys/payment.000.addr
cardano-cli signing-key-address --testnet-magic 42 \
--secret utxo-keys/payment.001.key > utxo-keys/payment.001.addr
```

Write genesis addresses to files&#x20;

```
cardano-cli signing-key-address \
    --testnet-magic 42 \
    --secret utxo-keys/byron.000.key > utxo-keys/byron.000.addr
```

Let's spend the genesis utxos and send the funds to the addresses we created above:

Now we need to send funds from their genesis addresses `genesis.000.addr`, `genesis.001.addr` and `genesis.002.addr`to our newly created regular addresses:

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

```
cardano-cli issue-genesis-utxo-expenditure \
--genesis-json configuration/byron-genesis.json \
--testnet-magic 42 \
--tx transactions/tx0.tx \
--wallet-key bft0/delegate-keys.000.key \
--rich-addr-from $(head -n 1 utxo-keys/genesis.000.addr) \
--txout "(\"$(head -n 1 byron/payment.000.addr)\", 17999999000000)"
```

```
cardano-cli issue-genesis-utxo-expenditure \
--genesis-json configuration/byron-genesis.json \
--testnet-magic 42 \
--tx transactions/tx1.tx \
--wallet-key bft1/delegate-keys.001.key \
--rich-addr-from $(head -n 1 byron/genesis.001.addr) \
--txout "(\"$(head -n 1 byron/payment.001.addr)\", 17999999000000)"
```

```
cardano-cli submit-tx \
            --testnet-magic 42 \
            --tx transactions/tx0.tx
cardano-cli submit-tx \
            --testnet-magic 42 \
            --tx transactions/tx1.tx
```

### Shelley Hardfork&#x20;

Before we can move on to Shelley, we will need a proper Shelley.genesis file&#x20;

```
mkdir shelley
cp template/shelley.json shelley/genesis.spec.json
cp template/alonzo.json shelley/genesis.alonzo.spec.json 
```

```
sed -i shelley/genesis.spec.json \
-e 's/"activeSlotsCoeff": 0.05/"activeSlotsCoeff": 0.10/' \
-e 's/"major": 6/"major": 1/' \
-e 's/"updateQuorum": 3/"updateQuorum": 2/' \
-e 's/"maxLovelaceSupply": 45000000000000000/"maxLovelaceSupply": 45000000000000/' \
-e 's/"epochLength": 432000/"epochLength": 9000/' \
-e 's/"securityParam": 108/"securityParam": 45/' \
-e 's/"slotLength": 1/"slotLength": 0.20/' 
```

```
cardano-cli genesis create \
--testnet-magic 42 \
--genesis-dir shelley \
--gen-genesis-keys 2
```

```
mv shelley/delegate-keys/delegate1* bft1/
mv shelley/delegate-keys/opcert1.cert bft1/
mv shelley/delegate-keys/delegate2* bft0/
mv shelley/delegate-keys/opcert2.cert bft0/opcert0.cert
cd bft0
rename 's/delegate2/delegate0/g' *
```

Back to local-cluster/

```
cd ..
```

Replace Shelley and Alonzo genesis files.

```
rm configuration/shelley-genesis.json configuration/alonzo-genesis.json
mv shelley/genesis.json configuration/shelley-genesis.json
mv shelley/genesis.alonzo.json configuration/alonzo-genesis.json
```

{% code overflow="wrap" %}
```
sed -i '$ s/$/ --shelley-kes-key delegate0.kes.skey --shelley-vrf-key delegate0.vrf.skey --shelley-operational-certificate opcert0.cert/' bft0/startnode.sh
```
{% endcode %}

{% code overflow="wrap" %}
```
sed -i '$ s/$/ --shelley-kes-key delegate1.kes.skey --shelley-vrf-key delegate1.vrf.skey --shelley-operational-certificate opcert1.cert/' bft1/startnode.sh   
```
{% endcode %}

We are ready to move to shelley era. So we need to Create, Submit and Vote and update proposal.&#x20;

```bash
cardano-cli byron governance create-update-proposal \
--filepath transactions/updateprotov1.proposal \
--testnet-magic "42" \
--signing-key bft0/delegate-keys.000.key \
--protocol-version-major "1" \
--protocol-version-minor "0" \
--protocol-version-alt "0" \
--application-name "cardano-sl" \
--software-version-num "1" \
--system-tag "linux" \
--installer-hash 0
```

<pre><code><strong>cardano-cli byron governance create-proposal-vote \
</strong>--proposal-filepath transactions/updateprotov1.proposal \
--testnet-magic 42 \
--signing-key bft0/delegate-keys.000.key \
--vote-yes \
--output-filepath transactions/updateprotov1.000.vote</code></pre>

<pre><code><strong>cardano-cli byron governance create-proposal-vote \
</strong>--proposal-filepath transactions/updateprotov1.proposal \
--testnet-magic 42 \
--signing-key bft1/delegate-keys.001.key \
--vote-yes \
--output-filepath transactions/updateprotov1.001.vote</code></pre>

Before we can move on and Submit the proposal and Vote, we need to do another adjustment to our config&#x20;

<pre><code><strong>cardano-cli byron submit-update-proposal \
</strong>            --testnet-magic 42 \
            --filepath transactions/updateprotov1.proposal</code></pre>

```
cardano-cli byron submit-proposal-vote  \
            --testnet-magic 42 \
            --filepath transactions/updateprotov1.000.vote
cardano-cli byron submit-proposal-vote  \
            --testnet-magic 42 \
            --filepath transactions/updateprotov1.001.vote
```

Your node logs will show the&#x20;

{% code overflow="wrap" %}
```
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate = ByronUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateVersion = 1.0.0, protocolUpdateState = UpdateRegistered (SlotNo 7586)}]}))
....
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate = ByronUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateVersion = 1.0.0, protocolUpdateState = UpdateConfirmed (SlotNo 7594)}]}))
...
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate = ByronUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateVersion = 1.0.0, protocolUpdateState = UpdateCandidate (SlotNo 7810) (EpochNo 8)}]}))
...
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate = ByronUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateVersion = 1.0.0, protocolUpdateState = UpdateStableCandidate (EpochNo 8)}]}))
```
{% endcode %}

Hardfork occurred

{% code overflow="wrap" %}
```
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate = ByronUpdatedProtocolUpdates []}))
```
{% endcode %}

Now lets upgrade to protocol version 2.0.0, The Shelley Era!&#x20;

```
cardano-cli byron governance create-update-proposal \
--filepath transactions/updateprotov2.proposal \
--testnet-magic "42" \
--signing-key bft0/delegate-keys.000.key \
--protocol-version-major "2" \
--protocol-version-minor "0" \
--protocol-version-alt "0" \
--application-name "cardano-sl" \
--software-version-num "1" \
--system-tag "linux" \
--installer-hash 0
```

```
cardano-cli byron governance create-proposal-vote \
--proposal-filepath transactions/updateprotov2.proposal \
--testnet-magic 42 \
--signing-key bft0/delegate-keys.000.key \
--vote-yes \
--output-filepath transactions/updateprotov2.000.vote
```

```
cardano-cli byron governance create-proposal-vote \
--proposal-filepath transactions/updateprotov2.proposal \
--testnet-magic 42 \
--signing-key bft1/delegate-keys.001.key \
--vote-yes \
--output-filepath transactions/updateprotov2.001.vote
```

<pre><code><strong>cardano-cli byron submit-update-proposal \
</strong>            --testnet-magic 42 \
            --filepath transactions/updateprotov2.proposal</code></pre>

```
cardano-cli byron submit-proposal-vote  \
            --testnet-magic 42 \
            --filepath transactions/updateprotov2.000.vote
cardano-cli byron submit-proposal-vote  \
            --testnet-magic 42 \
            --filepath transactions/updateprotov2.001.vote
```

{% code overflow="wrap" %}
```
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate = ByronUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateVersion = 2.0.0, protocolUpdateState = UpdateRegistered (SlotNo 8681)}]}))
....
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate = ByronUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateVersion = 2.0.0, protocolUpdateState = UpdateConfirmed (SlotNo 8691)}]}))
....
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate = ByronUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateVersion = 2.0.0, protocolUpdateState = UpdateStablyConfirmed (fromList [])}]}))
....

```
{% endcode %}
