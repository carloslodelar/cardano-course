---
cover: .gitbook/assets/p2p.png
coverY: 0
---

# P2P topology

{% embed url="https://www.youtube.com/watch?v=wnv7VCa79eo" %}

To Enable P2P, we do it from the configuration file, take for example the Preview testnet [configuration file](https://book.world.dev.cardano.org/environments/preview/config.json), it contains the field  `"EnableP2P".` It can be set to `false` or `true`

On Preview testnet the default is `true` since this network is already running with P2P.&#x20;

```
{
...
"EnableP2P": true,
...
}
```

