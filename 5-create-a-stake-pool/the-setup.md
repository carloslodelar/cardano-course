# The setup

A stake pool requires a number of keys:&#x20;

* Cold keys: used to register and register a stake pool,  to control the stake pool parameters, issue operational certificate and  retiring a stake pool.
* KES keys: used together with the operational certificate to sign blocks.
* VRF keys: used in the slot leader election process, and to contribute to the evolving nonce.&#x20;

When running a stake pool we will have a block producer node and at least one relay node, ideally more than one to guarantee that our blocks can still be broadcasted in case of failure on one relay or in its connections. &#x20;

The block producer will hold&#x20;

<figure><img src="../.gitbook/assets/Screen Shot 2023-02-24 at 12.33.39.png" alt=""><figcaption></figcaption></figure>
