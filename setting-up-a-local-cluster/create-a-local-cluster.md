---
cover: ../.gitbook/assets/cluster (1).png
coverY: 0
---

# Create a local cluster

Sometimes you might need to deploy a local cluster. We provide tools that allow you to do this with little effort. We will use those tools later.  For now we will do it manually.  Hopefully this will help having a better understanding of what happens behind the scenes when using the automated tools.&#x20;

In this example we will start a cluster in Byron era, and upgrade it all the way up to Babbage era.&#x20;

### Configure the network and nodes

#### Requirements:

* A linux or macOS machine, see the [Requirements](https://github.com/input-output-hk/cardano-node/blob/master/doc/getting-started/install.md) for details.&#x20;
* cardano-node and cardano-cli in your $PATH

{% hint style="info" %}
Note: In this section of the course we will minimize the use of scripts. The intention is making the process as transparent and clear as possible.&#x20;
{% endhint %}

To keep it simple enough we will use the `cardano-cli genesis create-cardano`  command to create most of the files we will need in the local cluster. We need to start from template files:

1. Configuration
2. Byron genesis template
3. Shelley genesis template
4. Alonzo genesis template

#### 1. The configuration files

[Cardano World Testnet Templates ](https://github.com/input-output-hk/cardano-world/tree/master/nix/cardano/environments/testnet-template)is a nice starting point.&#x20;

&#x20;Let's create a directory for our project

```bash
mkdir -p cluster/template
cd cluster
```

Download the template files and save them in the template folder we just created.&#x20;

<pre class="language-bash" data-overflow="wrap"><code class="lang-bash"><strong>wget -P template/ https://raw.githubusercontent.com/input-output-hk/cardano-world/master/nix/cardano/environments/testnet-template/alonzo.json
</strong>wget -P template/ https://raw.githubusercontent.com/input-output-hk/cardano-world/master/nix/cardano/environments/testnet-template/byron.json
wget -P template/ https://raw.githubusercontent.com/input-output-hk/cardano-world/master/nix/cardano/environments/testnet-template/config.json
wget -P template/ https://raw.githubusercontent.com/input-output-hk/cardano-world/master/nix/cardano/environments/testnet-template/shelley.json
wget -P template/ https://raw.githubusercontent.com/input-output-hk/cardano-world/master/nix/cardano/environments/testnet-template/conway.json
</code></pre>

Our system will start in Byron era and will have only 2 block producing nodes. We will add 2 Stake Pools later, when we transition to Shelley era.  Let's make a directory for each of the nodes and a configuration folder for our configuration files.&#x20;

```bash
mkdir -p bft0 bft1 configuration
```

We want our nodes to connect to each other, each of the nodes requires a topology file that tells the node to which nodes to connect to.&#x20;

The node bft0 will run on port 3000, and node bft1 on port 3001 for node bft1.  For now we will use classic topology (not P2P).

So we create the topology file for bft0 with:

```json
cat > bft0/topology.json <<EOF
{
   "Producers": [
     {
       "addr": "127.0.0.1",
       "port": 3001,
       "valency": 1
     }
   ]
 }
EOF
```

And for bft1 with:

```json
cat > bft1/topology.json <<EOF
{
   "Producers": [
     {
       "addr": "127.0.0.1",
       "port": 3000,
       "valency": 1
     }
   ]
 }
EOF
```

#### 3. The genesis files

Before, we downloaded the `byron.json` template. We will use it together with cardano-cli to generate our `byron-genesis.json` file. Our template looks like this:

```json
{
  "heavyDelThd":     "300000000000",
  "maxBlockSize":    "2000000",
  "maxTxSize":       "4096",
  "maxHeaderSize":   "2000000",
  "maxProposalSize": "700",
  "mpcThd": "20000000000000",
  "scriptVersion": 0,
  "slotDuration": "200",
  "softforkRule": {
    "initThd": "900000000000000",
    "minThd": "600000000000000",
    "thdDecrement": "50000000000000"
  },
  "txFeePolicy": {
    "multiplier": "43946000000",
    "summand": "155381000000000"
  },
  "unlockStakeEpoch": "18446744073709551615",
  "updateImplicit": "10000",
  "updateProposalThd": "100000000000000",
  "updateVoteThd": "1000000000000"
}
```

Note that this template uses 200 millisecond slots on Byron era. Mainnet used 20 second slots during the Byron era and uses 1 second slots during Shelley+ eras.&#x20;

Slots were 20 times longer In Byron era because there was a block in every slot. We needed to ensure that each block had enough time to travel the world and that everybody received the block before forging a next block.  In Shelley+ eras, slots are shorter but not all slots have a block, in mainnet only 5% of the slots have a block, this is the active slot coefficient. This selection of parameters allow for Byron and Shelley epochs to have approximately the same number of blocks.&#x20;

&#x20;In our local cluster we will use 2 second slots in Byron and 0.1 seconds slots on the following eras. The `cardano-cli genesis create-cardano` automatically generates configuration files with this 20:1 ratio.&#x20;

The rest of our parameters will match mainnet ones. For detailed information about the parameters, see:

1. [Byron genesis data format ](https://github.com/input-output-hk/cardano-node/blob/master/doc/reference/byron-genesis.md)
2. [Shelley era genesis ](https://github.com/input-output-hk/cardano-node/blob/master/doc/getting-started/understanding-config-files.md)

Let's make a few changes to our shelley.json template. Since we will only have 2 nodes, we bring updateQuorum down to 2, We reduce the epoch length to 9000, the security parameter k to 45,  the shelley era slots will last 1/10th of a second; and to help our cluster to match the progression of mainnet protocol versions, we set major (protocol version) to 2. On mainnet Shelley era is protocol version 2.0&#x20;

{% tabs %}
{% tab title="Linux" %}
```
sed -i template/shelley.json \
-e 's/"major": 6/"major": 2/' \
-e 's/"updateQuorum": 3/"updateQuorum": 2/'
```
{% endtab %}

{% tab title="macOS" %}
<pre class="language-bash"><code class="lang-bash"><strong>gsed -i template/shelley.json \
</strong>-e 's/"major": 6/"major": 2/' \
-e 's/"updateQuorum": 3/"updateQuorum": 2/'
</code></pre>
{% endtab %}
{% endtabs %}

We also need a few changes to the config.json template. for now we will disable P2P topology, we will disable EnableDevelopment options because we want to update our cluster using proper update proposals. Initially we will state that our nodes are ready to move to Protocol Version 1.0.0 (PBFT). Logs will come very fast, we will keep the minSeverity in Info. &#x20;

{% tabs %}
{% tab title="Linux" %}
```
 sed -i template/config.json \
 -e 's/"EnableP2P": true/"EnableP2P": false/' \
 -e 's/"ExperimentalProtocolsEnabled": true/"ExperimentalProtocolsEnabled": false/' \
 -e 's/"ExperimentalHardForksEnabled": true/"ExperimentalHardForksEnabled": false/' \
 -e 's/"LastKnownBlockVersion-Major": 3/"LastKnownBlockVersion-Major": 0/' \
 -e 's/"LastKnownBlockVersion-Minor": 1/"LastKnownBlockVersion-Minor": 0/' \
 -e 's/"TestShelleyHardForkAtEpoch": 0/"TestShelleyHardForkAtEpoch": /' \
 -e 's/"TestAllegraHardForkAtEpoch": 0/"TestAllegraHardForkAtEpoch": /' \
 -e 's/"TestMaryHardForkAtEpoch": 0/"TestMaryHardForkAtEpoch": /' \
 -e 's/"TestAlonzoHardForkAtEpoch": 0/"TestAlonzoHardForkAtEpoch": /' \
 -e 's/""minSeverity": "Debug"/"minSeverity": "Info"/' 
```
{% endtab %}

{% tab title="macOS" %}
```
 gsed -i template/config.json \
 -e 's/"EnableP2P": true/"EnableP2P": false/' \
 -e 's/"TestEnableDevelopmentNetworkProtocols": true/"TestEnableDevelopmentNetworkProtocols": false/' \
 -e 's/"TestEnableDevelopmentHardForkEras": true/"TestEnableDevelopmentHardForkEras": false/' \
 -e 's/"LastKnownBlockVersion-Major": 3/"LastKnownBlockVersion-Major": 0/' \
 -e 's/"LastKnownBlockVersion-Minor": 1/"LastKnownBlockVersion-Minor": 0/' \
 -e 's/"TestShelleyHardForkAtEpoch": 0/"TestShelleyHardForkAtEpoch": /' \
 -e 's/"TestAllegraHardForkAtEpoch": 0/"TestAllegraHardForkAtEpoch": /' \
 -e 's/"TestMaryHardForkAtEpoch": 0/"TestMaryHardForkAtEpoch": /' \
 -e 's/"TestAlonzoHardForkAtEpoch": 0/"TestAlonzoHardForkAtEpoch": /' \
 -e 's/""minSeverity": "Debug"/"minSeverity": "Info"/' 
```
{% endtab %}
{% endtabs %}

We'll also change a couple of Byron parameters for our local cluster. Change `minThd` to `"1000000000000000"`  so that update proposals need both genesis keys positive votes to be approved. And  change `"updateImplicit"` to 450, so that update proposals expire if they have not accumulated enough votes after 450 slots. &#x20;

{% tabs %}
{% tab title="Linux" %}
```
sed -i template/byron.json \
-e 's/"minThd": "600000000000000"/"minThd": "1000000000000000"/' \
-e 's/"updateImplicit": "10000"/"updateImplicit": "450"/'
```
{% endtab %}

{% tab title="macOS" %}
<pre><code><strong>gsed -i template/byron.json \
</strong>-e 's/"minThd": "600000000000000"/"minThd": "1000000000000000"/' \
-e 's/"updateImplicit": "10000"/"updateImplicit": "450"/'
</code></pre>
{% endtab %}
{% endtabs %}

Now we can use the magic of `cardano-cli genesis create-cardano`

{% tabs %}
{% tab title="Linux" %}
```bash
cardano-cli genesis create-cardano \
--genesis-dir ./ \
--gen-genesis-keys 2 \
--gen-utxo-keys 1 \
--start-time $(date -u -d "now + 2 minutes" +%FT%Tz) \
--supply 30000000000000000 \
--security-param 45 \
--slot-length 100 \
--slot-coefficient 5/100 \
--testnet-magic 42 \
--byron-template template/byron.json \
--shelley-template template/shelley.json \
--alonzo-template template/alonzo.json \
--conway-template template/conway.json \
--node-config-template template/config.json

```
{% endtab %}

{% tab title="macOS" %}
```bash
cardano-cli genesis create-cardano \
--genesis-dir ./ \
--gen-genesis-keys 2 \
--start-time $(gdate -u -d "now + 5 minutes" +%FT%Tz) \
--supply 30000000000000000 \
--security-param 45 \
--slot-length 100 \
--slot-coefficient 5/100 \
--testnet-magic 42 \
--byron-template template/byron.json \
--shelley-template template/shelley.json \
--alonzo-template template/alonzo.json \
--node-config-template template/config.json
```
{% endtab %}
{% endtabs %}

Move genesis files to configuration directory to keep tings in order:

```bash
mv node-config.json configuration/config.json
mv shelley-genesis.json byron-genesis.json alonzo-genesis.json conway-genesis.json configuration/
```

And let's move our genesis delegate keys to their corresponding delegate nodes:

```bash
mv delegate-keys/byron.000* delegate-keys/shelley.000* bft0/
mv delegate-keys/byron.001* delegate-keys/shelley.001* bft1/
```

To make our lives easier we will create bash scripts to start our nodes.&#x20;

Bft0 will run on port 3000

```bash
cat > bft0/startnode.sh <<EOF
#!/usr/bin/env bash

cardano-node run \
--config ../configuration/config.json \
--topology topology.json \
--database-path db \
--socket-path bft0.socket \
--port 3000 \
--delegation-certificate byron.000.cert.json \
--signing-key byron.000.key
EOF
```

And BFT1 on port 3001&#x20;

```bash
cat > bft1/startnode.sh <<EOF
#!/usr/bin/env bash

cardano-node run \
--config ../configuration/config.json \
--topology topology.json \
--database-path db \
--socket-path bft1.socket \
--port 3001 \
--delegation-certificate byron.001.cert.json \
--signing-key byron.001.key 
EOF
```

Now let's give our scripts executable permission:

```bash
chmod +x bft0/startnode.sh bft1/startnode.sh 
```

Open a new terminal for each of the nodes, go to the corresponding bft folder and run the script from there:

```bash
cd bft0
./startnode.sh
```

Repeat for bft1. The nodes will idle until start time is reached, then they will start producing blocks.&#x20;

Now you can set the environment variable  `CARDANO_NODE_SOCKET_PATH`

```bash
export CARDANO_NODE_SOCKET_PATH=~/cluster/bft0/bft0.socket
```

Let's query the tip of the chain to make sure everything is working as expected:&#x20;

```bash
cardano-cli query tip --testnet-magic 42
```

Great we have a network on Byron era running locally!&#x20;
