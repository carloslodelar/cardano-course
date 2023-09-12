---
cover: ../.gitbook/assets/cluster (1).png
coverY: 0
---

# From Byron to Shelley

You don't want to stay on Byron era forever, right? Of course not. Then move on to the Shelley era, but before you can do that you need to go to protocol version 1.0.0 (Ouroboros BFT). This was referred to as the Byron Reboot and was the only real hard fork in the traditional sense.  

## Upgrade to Ouroboros BFT 

Update your scripts that run the nodes to include Shelley keys:

Create, submit, and vote an update proposal.  

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

And submit the update proposal

<pre class="language-bash"><code class="lang-bash"><strong>cardano-cli byron submit-update-proposal \
</strong><strong>--testnet-magic 42 \
</strong><strong>--filepath transactions/updateprotov1.proposal
</strong></code></pre>

Monitor the log files from the node, you should see:  

{% code overflow="wrap" %}
```
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate = ByronUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateVersion = 1.0.0, protocolUpdateState = UpdateRegistered (SlotNo )}]}))
```
{% endcode %}

Now create the votes for both of your genesis keys

<pre class="language-bash"><code class="lang-bash"><strong>cardano-cli byron governance create-proposal-vote \
</strong>--proposal-filepath transactions/updateprotov1.proposal \
--testnet-magic 42 \
--signing-key bft0/byron.000.key \
--vote-yes \
--output-filepath transactions/updateprotov1.000.vote
</code></pre>

Submit the first vote  

```bash
cardano-cli byron submit-proposal-vote  \
--testnet-magic 42 \
--filepath transactions/updateprotov1.000.vote
```

You should see `UpdateActive`

{% code overflow="wrap" %}
```
...
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate = ByronUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateVersion = 1.0.0, protocolUpdateState = UpdateActive (fromList [KeyHash {unKeyHash = }])}]}))
...
```
{% endcode %}

Create and submit the second vote:

<pre class="language-bash"><code class="lang-bash"><strong>cardano-cli byron governance create-proposal-vote \
</strong>--proposal-filepath transactions/updateprotov1.proposal \
--testnet-magic 42 \
--signing-key bft1/byron.001.key \
--vote-yes \
--output-filepath transactions/updateprotov1.001.vote
</code></pre>

```bash
cardano-cli byron submit-proposal-vote  \
--testnet-magic 42 \
--filepath transactions/updateprotov1.001.vote
```

This vote completes the vote threshold; the update proposal is confirmed

{% code overflow="wrap" %}
```
...
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate = ByronUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateVersion = 1.0.0, protocolUpdateState = UpdateConfirmed (SlotNo )}]}))
...
```
{% endcode %}

It's time to endorse the proposal. Update your config file to indicate that you are ready to move to protocol `LastKnownBlockVersion-Major`: 1,

{% tabs %}
{% tab title="Linux" %}
```bash
sed -i configuration/config.json \
-e 's/"LastKnownBlockVersion-Major":0/"LastKnownBlockVersion-Major":1/'
```
{% endtab %}

{% tab title="macOS" %}
```bash
gsed -i configuration/config.json \
-e 's/"LastKnownBlockVersion-Major":0/"LastKnownBlockVersion-Major":1/'
```
{% endtab %}
{% endtabs %}

IMPORTANT: You need to restart the nodes to pick up the new configuration.   


As soon as you restart the second node and produce a block with it, the update proposal has received the required endorsements. It becomes a candidate, and the `epochNo` is set at which the transition will take place if it becomes a stable candidate.  

{% code overflow="wrap" %}
```
...
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate = ByronUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateVersion = 1.0.0, protocolUpdateState = UpdateCandidate (SlotNo ) (EpochNo )}]}))
...
```
{% endcode %}

2k slots later, it becomes a stable candidate.   

{% code overflow="wrap" %}
```
...
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate = ByronUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateVersion = 1.0.0, protocolUpdateState = UpdateStableCandidate (EpochNo )}]}))
...
```
{% endcode %}

The update proposal is adopted at the epoch transition:

{% code overflow="wrap" %}
```
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate = ByronUpdatedProtocolUpdates []}))
```
{% endcode %}

## Shelley hard fork  

Now you can upgrade to protocol version 2.0, the Shelley Era.  

First, add the Shelley keys to your BFT nodes starting scripts:  

{% tabs %}
{% tab title="Linux" %}
{% code overflow="wrap" %}
```bash
sed -i '$ s/$/ --shelley-kes-key shelley.000.kes.skey --shelley-vrf-key shelley.000.vrf.skey --shelley-operational-certificate shelley.000.opcert.json/' bft0/startnode.sh
```
{% endcode %}

{% code overflow="wrap" %}
```bash
sed -i '$ s/$/ --shelley-kes-key shelley.001.kes.skey --shelley-vrf-key shelley.001.vrf.skey --shelley-operational-certificate shelley.001.opcert.json/' bft1/startnode.sh 
```
{% endcode %}
{% endtab %}

{% tab title="macOS" %}


{% code overflow="wrap" %}
```bash
gsed -i '$ s/$/ --shelley-kes-key shelley.000.kes.skey --shelley-vrf-key shelley.000.vrf.skey --shelley-operational-certificate shelley.000.opcert.json/' bft0/startnode.sh
```
{% endcode %}

{% code overflow="wrap" %}
```bash
gsed -i '$ s/$/ --shelley-kes-key shelley.001.kes.skey --shelley-vrf-key shelley.001.vrf.skey --shelley-operational-certificate shelley.001.opcert.json/' bft1/startnode.sh 
```
{% endcode %}
{% endtab %}
{% endtabs %}

Create the update proposal:

```bash
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

Submit the proposal:

<pre><code><strong>cardano-cli byron submit-update-proposal \
</strong><strong>--testnet-magic 42 \
</strong>--filepath transactions/updateprotov2.proposal
</code></pre>

Create the votes:

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

Submit the votes:

```
cardano-cli byron submit-proposal-vote  \
            --testnet-magic 42 \
            --filepath transactions/updateprotov2.000.vote
cardano-cli byron submit-proposal-vote  \
            --testnet-magic 42 \
            --filepath transactions/updateprotov2.001.vote
```

Then, adjust the config file again to go to protocol version 2.0

{% tabs %}
{% tab title="Linux" %}
```bash
sed -i 's/"LastKnownBlockVersion-Major":1/"LastKnownBlockVersion-Major":2/' configuration/config.json
```
{% endtab %}

{% tab title="macOS" %}
```bash
gsed -i 's/"LastKnownBlockVersion-Major":1/"LastKnownBlockVersion-Major":2/' configuration/config.json
```
{% endtab %}
{% endtabs %}

Restart the nodes to pick up the new configuration and endorse the proposal.  

{% code overflow="wrap" %}
```
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate = ByronUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateVersion = 2.0.0, protocolUpdateState = UpdateRegistered (SlotNo )}]}))
....
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate = ByronUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateVersion = 2.0.0, protocolUpdateState = UpdateConfirmed (SlotNo )}]}))
...
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate = ByronUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateVersion = 2.0.0, protocolUpdateState = UpdateStablyConfirmed (fromList [])}]}))
....
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate = ByronUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateVersion = 2.0.0, protocolUpdateState = UpdateCandidate (SlotNo ) (EpochNo )}]}))
...
Event: LedgerUpdate (HardForkUpdateInEra Z (WrapLedgerUpdate {unwrapLedgerUpdate = ByronUpdatedProtocolUpdates [ProtocolUpdate {protocolUpdateVersion = 2.0.0, protocolUpdateState = UpdateStableCandidate (EpochNo )}]}))
...
Event: LedgerUpdate (HardForkUpdateTransitionDone <EraIndex Byron> <EraIndex Shelley> (EpochNo ))

```
{% endcode %}

IMPORTANT: Take note of the epoch at which the Shelley hard fork takes place. In this case, the Shelley hard fork happened at the transition to epoch 3. The Byron epochs lasted 450 slots (10 times the security parameter k) so it happened at slot 1350.  This information will be useful later, so write it down somewhere.
