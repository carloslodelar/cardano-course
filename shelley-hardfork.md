# From Byron to Shelley

We don't want to stay on Byron era forever, right? Of course not. Let's move on to Shelley era

### Upgrade to Ouroboros BFT&#x20;

Let's update our scripts to run the nodes to include Shelley keys:

{% code overflow="wrap" %}
```
sed -i '$ s/$/ --shelley-kes-key shelley.000.kes.skey --shelley-vrf-key shelley.000.vrf.skey --shelley-operational-certificate shelley.000.opcert.json/' bft0/startnode.sh
```
{% endcode %}

{% code overflow="wrap" %}
```
sed -i '$ s/$/ --shelley-kes-key shelley.001.kes.skey --shelley-vrf-key shelley.001.vrf.skey --shelley-operational-certificate shelley.001.opcert.json/' bft1/startnode.sh  
```
{% endcode %}

We are almost ready to move to OBFT era. So we need to Create, Submit and Vote and update proposal.&#x20;

```bash
cardano-cli byron governance create-update-proposal \
--filepath transactions/updateprotov1.proposal \
--testnet-magic 42 \
--signing-key bft0/byron.000.key \
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
--signing-key bft0/byron.000.key \
--vote-yes \
--output-filepath transactions/updateprotov1.000.vote
</code></pre>

<pre><code><strong>cardano-cli byron governance create-proposal-vote \
</strong>--proposal-filepath transactions/updateprotov1.proposal \
--testnet-magic 42 \
--signing-key bft1/byron.001.key \
--vote-yes \
--output-filepath transactions/updateprotov1.001.vote
</code></pre>

Before we can move on and Submit the proposal and Vote let's make sure that our config file says that to we are ready to move to protocol "LastKnownBlockVersion-Major": 1,

```
jq .'"LastKnownBlockVersion-Major"' configuration/config.json 
1
```

grep&#x20;

Let's restart the nodes to pick the changes on configuration:



<pre><code><strong>cardano-cli byron submit-update-proposal \
</strong>            --testnet-magic 42 \
            --filepath transactions/updateprotov1.proposal
</code></pre>

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
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate = ByronUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateVersion = 1.0.0, protocolUpdateState = UpdateRegistered (SlotNo 457)}]}))
...
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate = ByronUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateVersion = 1.0.0, protocolUpdateState = UpdateConfirmed (SlotNo 467)}]}))
....
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate = ByronUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateVersion = 1.0.0, protocolUpdateState = UpdateCandidate (SlotNo 557) (EpochNo 2)}]}))
...
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate = ByronUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateVersion = 1.0.0, protocolUpdateState = UpdateStableCandidate (EpochNo 2)}]}))
```
{% endcode %}

Hardfork occurred

{% code overflow="wrap" %}
```
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate = ByronUpdatedProtocolUpdates []}))
```
{% endcode %}

### Shelley Hardfork&#x20;

Now lets upgrade to protocol version 2.0.0, The Shelley Era!&#x20;

Let's adjust the config file again:

```
sed -i 's/"LastKnownBlockVersion-Major":1/"LastKnownBlockVersion-Major":2/' configuration/config.json
```

```
cardano-cli byron governance create-update-proposal \
--filepath transactions/updateprotov2.proposal \
--testnet-magic 42 \
--signing-key bft0/byron.000.key \
--protocol-version-major "2" \
--protocol-version-minor "0" \
--protocol-version-alt "0" \
--application-name "cardano-sl" \
--software-version-num "1" \
--system-tag "linux" \
--installer-hash 0
```

and restart our nodes once more

```
cardano-cli byron governance create-proposal-vote \
--proposal-filepath transactions/updateprotov2.proposal \
--testnet-magic 42 \
--signing-key bft0/byron.000.key \
--vote-yes \
--output-filepath transactions/updateprotov2.000.vote
```

```
cardano-cli byron governance create-proposal-vote \
--proposal-filepath transactions/updateprotov2.proposal \
--testnet-magic 42 \
--signing-key bft1/byron.001.key \
--vote-yes \
--output-filepath transactions/updateprotov2.001.vote
```

<pre><code><strong>cardano-cli byron submit-update-proposal \
</strong>            --testnet-magic 42 \
            --filepath transactions/updateprotov2.proposal
</code></pre>

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
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate = ByronUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateVersion = 2.0.0, protocolUpdateState = UpdateRegistered (SlotNo 931)}]}))
....
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate = ByronUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateVersion = 2.0.0, protocolUpdateState = UpdateConfirmed (SlotNo 938)}]}))
...
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate = ByronUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateVersion = 2.0.0, protocolUpdateState = UpdateStablyConfirmed (fromList [])}]}))
....
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate = ByronUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateVersion = 2.0.0, protocolUpdateState = UpdateCandidate (SlotNo 1028) (EpochNo 3)}]}))
...
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate = ByronUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateVersion = 2.0.0, protocolUpdateState = UpdateStableCandidate (EpochNo 3)}]}))
...
Event: LedgerUpdate (HardForkUpdateTransitionDone <EraIndex Byron> <EraIndex Shelley> (EpochNo 3))

```
{% endcode %}
