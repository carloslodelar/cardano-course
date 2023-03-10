---
cover: .gitbook/assets/p2p.png
coverY: 0
---

# P2P topology

{% embed url="https://www.youtube.com/watch?v=wnv7VCa79eo" %}

{% embed url="https://docs.cardano.org/explore-cardano/cardano-network/p2p-networking" %}

To Enable P2P, we do it from the configuration file, take for example the Preview testnet [configuration file](https://book.world.dev.cardano.org/environments/preview/config.json), it contains the field  `"EnableP2P".` It can be set to `false` or `true`

For example, on Preview testnet the default is `true` since this network is already running with P2P.&#x20;

<pre><code>{
...
<strong>  "EnableP2P": true,
</strong>...
  "TargetNumberOfActivePeers": 20,
  "TargetNumberOfEstablishedPeers": 40,
  "TargetNumberOfKnownPeers": 100,
  "TargetNumberOfRootPeers": 100,
  "TestEnableDevelopmentNetworkProtocols": true,
}
</code></pre>

#### The P2P topology file&#x20;

{% embed url="https://github.com/input-output-hk/cardano-node/blob/master/doc/getting-started/understanding-config-files.md" %}

#### New P2P topology file format (please use it on node 1.35.6 Single Relay)

{% embed url="https://github.com/input-output-hk/cardano-node/issues/4559" %}
