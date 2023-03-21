---
cover: ../.gitbook/assets/p2p.png
coverY: 0
---

# Peer-to-peer (P2P) networking

### What do we achieve with P2P

* With automatic P2P, (registered) nodes can discover and establish connections with each other&#x20;
* Nodes can establish full duplex  connections (simultaneous server and client side of the mini-protocols)
* Nodes can maintain some static topology i.e. its own relays and bp, and trusted peers with which we always want to keep a connection.&#x20;
* RootPeers reload with SIGHUP signal, no need to restart.&#x20;
*   The node dynamically manages the connections:  Each node maintains a set of peers mapped into three categories:

    * **cold peers** ‒ existing (known) peers without an established network connection
    * **warm peers** ‒ peers with an established bearer connection, which is only used for network measurements without implementing any of the node-to-node mini-protocols (not active).
    * **hot peers** ‒ peers that have a connection, which is being used by all three node-to-node mini-protocols (active peers)

    Newly discovered peers are initially added to the cold peer set. The P2P governor is then responsible for peer connection management.
* Maintaining diversity in hop distances contributes to better block distribution times across the globally distributed network.
* In the case of adversarial behavior, the peer can be immediately demoted from the hot, warm, or cold sets.

We classify upstream peers in 3 nested categories: Known/Established/Active

<figure><img src="../.gitbook/assets/Screen Shot 2023-03-10 at 13.40.58.png" alt=""><figcaption></figcaption></figure>

### What is next

* Peer sharing&#x20;
* Release Ouroboros Genesis

{% hint style="info" %}
Learn more:&#x20;

[Peer-to-peer (P2P) networking](https://docs.cardano.org/explore-cardano/cardano-network/p2p-networking)

[Interview with Networking team lead ](https://youtu.be/wnv7VCa79eo)
{% endhint %}

###

#### The P2P topology file&#x20;

{% embed url="https://github.com/input-output-hk/cardano-node/blob/master/doc/getting-started/understanding-config-files.md" %}

#### New P2P topology file format (please use it on node 1.35.6 Single Relay)

{% embed url="https://github.com/input-output-hk/cardano-node/issues/4559" %}

### Example of a topology file for a node not involved in block production or block propagation

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

### Example of topology files for a stakepool

The block-producer node includes it's own relays (`x.x.x.x` and `y.y.y.y`) under local roots. Note that we use `"useLedgerAfterSlot": -1` to indicate that it should never use LedgerPeers.

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

The relay `x.x.x.x`  inlcudes its own block producer node (`z.z.z.z`) and the other relay (`y.y.y.y`) under local roots, and a few other root peers under public roots. Note that this time we do want to use LedgerPeers, thus we use `"useLedgerAfterSlot": 10000000`

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

### Configuring the node to use P2P

To Enable P2P, we do it from the configuration file, take for example the Preview testnet [configuration file](https://book.world.dev.cardano.org/environments/preview/config.json), it contains the field  `"EnableP2P".` It can be set to `false` or `true`

For example, on Preview testnet the default is `true` since this network is already running with P2P.  We also need to configure the Target number of _Active_, _Established_ and _Known_ Peers, together with the target of _Root_ Peers

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
