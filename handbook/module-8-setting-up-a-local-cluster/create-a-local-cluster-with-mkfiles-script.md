# Create a local cluster with mkfiles script

Now that you know how the internals of a local cluster work, you can benefit from some automation. We have a script that can help you start a cluster that forks into Babbage era at epoch 0. It is located here:&#x20;

{% embed url="https://github.com/input-output-hk/cardano-node/blob/master/scripts/babbage/mkfiles.sh" %}

{% hint style="info" %}
Make sure to use cardano-node 8.0.0

```
cardano-node --version
cardano-node 8.0.0 - linux-x86_64 - ghc-8.10
git rev 69a117b7be3db0f4ce6d9fc5cd4c16a2a409dcb8
```
{% endhint %}

To create a local-cluster using mkfiles.sh,  please clone cardano-node repo and checkout tagged version 8.0.0&#x20;

```bash
git colne https://github.com/input-output-hk/cardano-node.git
git checkout tags/8.0.0
```

`mkfiles.sh` script is currently set to start the cluster on Conway era, but Conway development is still in progress and most of the functionalities of cardano-node will not work just yet. &#x20;

We need to edit the file [scripts/babbage/mkfiles.sh](https://github.com/input-output-hk/cardano-node/blob/master/scripts/babbage/mkfiles.sh) file to ensure that our cluster starts in Babbage era.  All we have to do is comment (#) this line on the script:&#x20;

```
# echo "TestConwayHardForkAtEpoch: 0" >> "${ROOT}/configuration.yaml" 
```

Save your changes,&#x20;

It is recommended to have 3 additional terminals ready, one for each node.  When you are ready, run the `mkfiles.sh` script.&#x20;

```
./scripts/babbage/mkfiles.sh
generated genesis with: 3 genesis keys, 3 non-delegating UTxO keys, 3 stake pools, 3 delegating UTxO keys, 3 delegation map entries,
example/node-spo1.sh
example/node-spo2.sh
example/node-spo3.sh
CARDANO_NODE_SOCKET_PATH=example/node-spo1/node.sock
```

Then, run a node on each terminal.  For example:

Terminal 1

```
example/node-spo1.sh
```

Terminal 2

```
example/node-spo2.sh
```

Terminal 3

```
example/node-spo3.sh
```

{% hint style="info" %}
Note that you have 30 seconds to start the 3 nodes. To give yourself more tim, you can change this line on the mkfiles.sh script:

```shellscript
START_TIME="$(${DATE} -d "now + 30 seconds" +%s)"

```
{% endhint %}

After starting your nodes, make sure to set the CARDANO\_NODE\_SOCKET\_PATH on your working terminal:&#x20;

```
CARDANO_NODE_SOCKET_PATH=example/node-spo1/node.sock
```

and voilà! We have a cluster running on Babbage era with very little effort:

```
cardano-cli query tip --testnet-magic 42
{
    "block": 3519,
    "epoch": 70,
    "era": "Babbage",
    "hash": "dd38083ef4392502218e71c6364bcfefdef1f9e6563f27fa1db2196e1c65b786",
    "slot": 35361,
    "slotInEpoch": 361,
    "slotsToEpochEnd": 139,
    "syncProgress": "100.00"
}
```
