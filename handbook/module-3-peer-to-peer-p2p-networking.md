---
cover: ../.gitbook/assets/p2p.png
coverY: 0
---

# Module 4. Peer-to-peer (P2P) networking

## What do we achieve with P2P?

* Through automatic P2P, registered nodes can discover and establish connections with each other&#x20;
* Nodes can establish full-duplex connections, operating simultaneously as servers and clients for the mini-protocols
* Nodes can maintain certain static topologies, including their own relays and block producers, as well as trusted peers with which continuous connections are desired&#x20;
* Root peers can be reloaded using the SIGHUP signal, eliminating the need for a restart&#x20;
* The node can dynamically manage the connections, each node maintains a set of peers mapped into three categories:

    * **cold peers** ‒ existing (known) peers without an established network connection
    * **warm peers** ‒ peers with an established bearer connection, which is only used for network measurements without implementing any of the node-to-node mini-protocols (not active)
    * **hot peers** ‒ peers that have a connection, which is being used by all three node-to-node mini-protocols (active peers)

    Newly discovered peers are initially added to the cold peer set. The P2P governor is then responsible for peer connection management.
* Maintaining diversity in hop distances contributes to better block distribution times across the globally distributed network
* In the case of adversarial behavior, the peer can be immediately demoted from the hot, warm, or cold sets

Upstream peers are classified into three categories: known, established, and active.

<figure><img src="../.gitbook/assets/Screen Shot 2023-03-10 at 13.40.58.png" alt=""><figcaption></figcaption></figure>

## What is next?

* Peer sharing&#x20;
* Ouroboros Genesis release 

{% hint style="info" %}
Learn more:&#x20;

[Peer-to-peer (P2P) networking](https://docs.cardano.org/explore-cardano/cardano-network/p2p-networking)

[Interview with the networking team lead ](https://youtu.be/wnv7VCa79eo)
{% endhint %}

## The P2P topology file&#x20;

{% embed url="https://github.com/input-output-hk/cardano-node/blob/master/doc/getting-started/understanding-config-files.md" %}

### The new P2P topology file format (please use it with node v.1.35.6 single relay):

{% embed url="https://github.com/input-output-hk/cardano-node/issues/4559" %}

### Example of a topology file for a node not involved in block production or block propagation:

```json
{
   "localRoots":[
      {
         "accessPoints":[
            
         ],
         "advertise":false,
         "valency":1
      }
   ],
   "publicRoots":[
      {
         "accessPoints":[
            {
               "address":"relays-new.cardano-mainnet.iohk.io",
               "port":3001
            }
         ],
         "advertise":false
      }
   ],
   "useLedgerAfterSlot":322000
}
```

### Example of topology files for a stake pool

The block-producing node includes its own relays (`x.x.x.x` and `y.y.y.y`) under local roots. Note that the example uses `"useLedgerAfterSlot": -1` to indicate that it should never use `LedgerPeers`.

```json
{
   "localRoots":[
      {
         "accessPoints":[
            {
               "address":"x.x.x.x",
               "port":3000
            },
            {
               "address":"y.y.y.y",
               "port":3000
            }
         ],
         "advertise":false,
         "valency":1
      }
   ],
   "publicRoots":[
      {
         "accessPoints":[
            
         ],
         "advertise":false
      }
   ],
   "useLedgerAfterSlot":-1
}
```

The relay `x.x.x.x` includes its own block-producing node (`z.z.z.z`), the other relay (`y.y.y.y`) under local roots, and a few other root peers under public roots. Note that this time, the example does use LedgerPeers - `"useLedgerAfterSlot": 10000000`.

```json
{
   "localRoots":[
      {
         "accessPoints":[
            {
               "address":"z.z.z.z",
               "port":3000
            },
            {
               "address":"y.y.y.y",
               "port":3000
            }
         ],
         "advertise":false,
         "valency":1
      }
   ],
   "publicRoots":[
      {
         "accessPoints":[
            {
               "address":"relays-new.cardano-mainnet.iohk.io",
               "port":3001
            }
         ],
         "advertise":false
      }
   ],
   "useLedgerAfterSlot":322000
}
```

## Configuring the node to use P2P

You can enable P2P from the configuration file. For example, in the preview testnet [configuration file](https://book.world.dev.cardano.org/environments/preview/config.json), you will find the field `EnableP2P`, which can be set to either `false` or `true`.

On the preview testnet, the default is `true` since this network is already running with P2P. You will also need to configure the target number of _active_, _established_ and _known_ peers, together with the target number of _root_ peers:

<pre><code>{
...
<strong>  "EnableP2P": true,
</strong>...
  "TargetNumberOfActivePeers": 20,
  "TargetNumberOfEstablishedPeers": 40,
  "TargetNumberOfKnownPeers": 100,
  "TargetNumberOfRootPeers": 100,
}
</code></pre>
