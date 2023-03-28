---
cover: .gitbook/assets/p2p.png
coverY: 0
---

# Module 4 Peer-to-peer (P2P) networking

{% embed url="https://youtu.be/zOTfhcK-Wf4" %}

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

<figure><img src=".gitbook/assets/Screen Shot 2023-03-10 at 13.40.58.png" alt=""><figcaption></figcaption></figure>

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

* As a node that is not involved in block production/propagation we don't need local roots, we can put IOG relays under **Public Roots**.&#x20;
* **Advertise** field does not have any effect at the moment. It is intended for later when peer sharing is released.&#x20;
* Note that we are using    `"useLedgerAfterSlot":0` This will disable Ledger peers the first time we run the node, i.e. joining the network for the first time, the next time we run the node, it will pick-up ledger peers.&#x20;

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
               "address":"preview-node.world.dev.cardano.org",
               "port":30002
            }
         ],
         "advertise":false
      }
   ],
   "useLedgerAfterSlot":0
}
```

### Example of topology files for a stakepool

### Block Producer

* The block-producer node includes it's own relays (`x.x.x.x` and `y.y.y.y`) under **Local Roots.**
* **Advertise** is not used, this is for use when peer sharing is released.&#x20;
* The value of **valency** must equal the number of local roots in that group.&#x20;
* Do not use **Public Roots** in a block producer.
* Note that we use `"useLedgerAfterSlot": -1` to indicate that it should never use LedgerPeers.

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
         "valency":2
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

### Relays

* The relay `x.x.x.x`  includes its own block producer node (`z.z.z.z`) and the other relay (`y.y.y.y`) under local roots.&#x20;
* Again,  value of valency must equal the number of local roots in that group, so it's 2. &#x20;
* We can have other peers under public roots.&#x20;
* On relays we do want to use LedgerPeers, thus we use `"useLedgerAfterSlot": 10000000`

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
         "valency":2
      }
   ],
   "publicRoots":[
      {
         "accessPoints":[
            {
               "address":"preview-node.world.dev.cardano.org",
               "port":30002
            }
         ],
         "advertise":false
      }
   ],
   "useLedgerAfterSlot":1000000
}
```

### Relays with more than one group in local roots

Assume a.a.a.a is a DNS of a partner pool to which we always want to have a connection with. Let's say this DNS can resolve to 2 IPs. We want to have 1 "hot" connection to any of the resolved IPs without compromising our connections to our block producer z.z.z.z and our relay y.y.y.y; in that case we put them on separate groups:

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
         "valency":2
      },
      {
         "accessPoints":[
            {
               "address":"a.a.a.a",
               "port":3000
            },
         ],
         "advertise":false,
         "valency":1
      }
   ],
   "publicRoots":[
      {
         "accessPoints":[
            {
               "address":"preview-node.world.dev.cardano.org",
               "port":30002
            }
         ],
         "advertise":false
      }
   ],
   "useLedgerAfterSlot":1000000
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
