# The setup

A stake pool requires a number of artifacts:&#x20;

* **Cold keys**: used to register a stake pool,  to control the stake pool parameters, issue operational certificate and  retiring a stake pool.
* **Operational keys**, also called **KES key**s: used together with the operational certificate to sign blocks.
* **VRF keys**: used in the slot leader election process, and to contribute to the evolving nonce.&#x20;
* **Operational certificate:** to transfer stake rights from Cold keys to KES keys. The certificate is included in the block header.&#x20;
* **Operational certificate counter:** To determine precendece, a certificate with a higher counter overrides one with a lower counter.&#x20;

When running a stake pool we will have a core node (with the job of producing blocks), and at least one relay node responsible for propagating blocks.  Ideally we will have more than one relay to guarantee that our blocks can still be broadcasted in case of failure of one relay or its connections. Each node must be running on a separate machine.&#x20;

The **block producer** node will hold the KES signing key, the VRF signing key and the Operational Certificate

**Relay** nodes do not need any keys.&#x20;

In addition, we will need an **air gapped machine** (specially when running a pool on mainnet) **** to keep Cold keys and wallet keys secure and never exposed to the internet.&#x20;

The **block producer** must be well secured and **only** connect to its own relay nodes.&#x20;

**Relay nodes** will connect to its own block producer and to 15-20 other relays on the network.&#x20;

P2P ensures duplex connections, using classic topology requires other relays to include our relays on their topology. Some agreements with other stake pool operators might be needed.&#x20;

<figure><img src="../.gitbook/assets/Screen Shot 2023-02-24 at 12.33.39.png" alt=""><figcaption></figcaption></figure>
