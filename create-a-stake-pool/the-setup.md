# The setup

{% hint style="info" %}
**This tutorial guides you through the process of creating a stake pool on the preview testnet (testnet-magic 2).** &#x20;

**Make sure to use the appropriate genesis files from**: [**https://book.world.dev.cardano.org/environments.html**](https://book.world.dev.cardano.org/environments.html).\
\
**For pre-production testnet, just replace `--testnet-magic 2` with `--testnet-magic 1`.**&#x20;

**For mainnet, use `--mainnet`**
{% endhint %}

A stake pool requires several key components:&#x20;

* **Cold keys**: used for registering a stake pool, controlling stake pool parameters, issuing operational certificates, and retiring a stake pool
* **VRF keys**: play a crucial role in the slot leader election process and contribute to the evolving nonce&#x20;
* **Operational keys**, also known as **KES key**s: used in conjunction with operational certificates to sign blocks
* **Operational certificate:** facilitates the transfer of stake rights from cold keys to KES keys and is included in the block header&#x20;
* **Operational certificate counter:** to determine precedence, a certificate with a higher counter takes precedence over one with a lower counter&#x20;

When operating a stake pool, there should be a core node responsible for block production and at least one relay node responsible for block propagation. Ideally, multiple relays are advisable to ensure continuous block broadcasting, even in the event of a relay or its connections failing. Each node should run on a separate machine.&#x20;

The **block-producing** node holds the KES signing key, the VRF signing key, and the operational certificate. **Relay** nodes do not need any keys.&#x20;

Additionally, an air-gapped machine is essential, especially when running a pool on mainnet. It ensures the security of cold keys and wallet keys by keeping them isolated from the internet, preventing any exposure.&#x20;

The **block producer** must be well secured and **only** connect to its own relay nodes.&#x20;

**Relay nodes** should connect to their own block producer and to 15-20 other relays on the network.&#x20;

P2P ensures duplex connections, whereas employing a classic topology demands that other relays include your relays in their topology. This may necessitate agreements with other stake pool operators.&#x20;

<figure><img src="../.gitbook/assets/Screen Shot 2023-02-24 at 12.33.39.png" alt=""><figcaption></figcaption></figure>
