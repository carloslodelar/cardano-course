---
cover: ../.gitbook/assets/cluster (1).png
coverY: 0
---

# Creating a local cluster

{% hint style="success" %}
This tutorial has been updated to work with [cardano-node v.8.0.0](https://github.com/input-output-hk/cardano-node/releases/tag/8.0.0).
{% endhint %}

In certain situations, it may be necessary to set up a local cluster. To simplify this process, there are user-friendly tools that require minimal effort to deploy. However, to help with a deeper comprehension of the automated tools and the underlying processes involved, this tutorial will begin by setting up the cluster manually. This hands-on approach will provide valuable insights into the steps required.

The following example will initiate a cluster during the Byron era and subsequently upgrade it progressively until it reaches the Babbage era. This step-by-step process will demonstrate the evolution and advancements of the cluster in its transition through different eras.

## Configuring the network and nodes

### Requirements:

* A Linux or macOS machine, see the [Requirements](https://github.com/input-output-hk/cardano-node-wiki/blob/main/docs/getting-started/install.md) for details
* `cardano-node` and `cardano-cli` in your $PATH

{% hint style="info" %}
Note: This section of the course will minimize the use of scripts. The intention is to make the process as transparent and clear as possible.  
{% endhint %}

The `cardano-cli genesis create-cardano` command will be used to simplify the process. It will create most of the necessary files for the local cluster. You only need to supply the following template files as a starting point:

1. Configuration
2. Byron genesis template
3. Shelley genesis template
4. Alonzo genesis template
5. Conway genesis template  

### 1. The configuration files

[Cardano World testnet templates](https://github.com/input-output-hk/iohk-nix/tree/master/cardano-lib/testnet-template) is the source for the template files. These are the templates that IOG uses to deploy a new testnet.  

Create a directory for the project:

```bash
mkdir -p cluster/template
cd cluster
```

Download the template files and save them in the _template_ folder you just created:

<pre class="language-bash" data-overflow="wrap"><code class="lang-bash"><strong>wget -P template/ https://raw.githubusercontent.com/input-output-hk/iohk-nix/master/cardano-lib/testnet-template/alonzo.json
</strong>wget -P template/ https://raw.githubusercontent.com/input-output-hk/iohk-nix/master/cardano-lib/testnet-template/byron.json
wget -P template/ https://raw.githubusercontent.com/input-output-hk/iohk-nix/master/cardano-lib/testnet-template/config.json
wget -P template/ https://raw.githubusercontent.com/input-output-hk/iohk-nix/master/cardano-lib/testnet-template/shelley.json
wget -P template/ https://raw.githubusercontent.com/input-output-hk/iohk-nix/master/cardano-lib/testnet-template/conway.json
</code></pre>

The system will start in the Byron era and will have only two block-producing nodes. You will add two stake pools later when you transition to the Shelley era. Make a directory for each of the nodes and a configuration folder for your configuration files:  

```bash
mkdir -p bft0 bft1 configuration
```

The nodes must connect to each other. Each of the nodes requires a topology file that tells it which nodes to connect to.  

The node bft0 will run on port 3000, and node bft1 – on port 3001. For now, use the classic topology (not P2P).

Create the topology file for bft0 with:

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

### 2. The genesis files

Use the `byron.json` template file together with `cardano-cli` to generate a `byron-genesis.json` file. The template looks like this:

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

Note that this template uses 200 millisecond slots in the Byron era. Mainnet slots lasted 20 seconds during the Byron era and one second during Shelley and later eras.  

Slots were 20 times longer in the Byron era because there was a block in every slot. This was necessary to ensure that each block had enough time to travel the world and that everybody received the block before forging the next block. In Shelley and later eras, slots are shorter but not all slots have a block. On mainnet, only 5% of the slots have a block; this is the active slot coefficient. This selection of parameters allows for Byron and Shelley epochs to have approximately the same number of blocks.  

The local cluster uses two-second slots in Byron and 0.1-second slots in the following eras. The `cardano-cli genesis create-cardano` command automatically generates configuration files with this 20:1 ratio.  

The rest of the parameters will match the mainnet ones. For detailed information about the parameters, see:

1. [Byron genesis data format](https://github.com/input-output-hk/cardano-node-wiki/blob/main/docs/reference/byron-genesis.md)
2. [Shelley era genesis](https://github.com/input-output-hk/cardano-node-wiki/blob/main/docs/getting-started/understanding-config-files.md)

You can now make a few changes to the `shelley.json` template. Since there are only two nodes, bring `updateQuorum` down to 2, reduce the epoch length to 9000, the security parameter k to 45, and  the Shelley era slots will last 1/10th of a second. To help the cluster match the progression of mainnet protocol versions, set major (protocol version) to 2. On mainnet, Shelley era is protocol version 2.0:  

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

A few changes are also needed to the `config.json` template. For now, disable P2P topology and disable the `EnableDevelopment` options so you can update your cluster using proper update proposals. Initially, state that your nodes are ready to move to protocol version 1.0.0 (PBFT). Logs will come very fast, so keep the `minSeverity` in info:  

{% tabs %}
{% tab title="Linux" %}
```
 sed -i template/config.json \
 -e 's/"ExperimentalProtocolsEnabled": true/"ExperimentalProtocolsEnabled": false/' \
 -e 's/"ExperimentalHardForksEnabled": true/"ExperimentalHardForksEnabled": false/' \
 -e 's/"LastKnownBlockVersion-Major": 3/"LastKnownBlockVersion-Major": 0/' \
 -e 's/"LastKnownBlockVersion-Minor": 1/"LastKnownBlockVersion-Minor": 0/' \
 -e 's/""minSeverity": "Debug"/"minSeverity": "Info"/' 
```
{% endtab %}

{% tab title="macOS" %}
```
 gsed -i template/config.json \
 -e 's/"ExperimentalProtocolsEnabled": true/"ExperimentalProtocolsEnabled": false/' \
 -e 's/"ExperimentalHardForksEnabled": true/"ExperimentalHardForksEnabled": false/' \
 -e 's/"LastKnownBlockVersion-Major": 3/"LastKnownBlockVersion-Major": 0/' \
 -e 's/"LastKnownBlockVersion-Minor": 1/"LastKnownBlockVersion-Minor": 0/' \
 -e 's/""minSeverity": "Debug"/"minSeverity": "Info"/' 
```
{% endtab %}
{% endtabs %}

Also, change a couple of Byron parameters for your local cluster. Change `minThd` to `"1000000000000000"`  so that update proposals need positive votes from both genesis keys to be approved. And change `"updateImplicit"` to 450, so that update proposals expire if they have not accumulated enough votes after 450 slots:  

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

Now you can use the magic of `cardano-cli genesis create-cardano`:

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

Move genesis files to the configuration directory to keep things in order:

```bash
mv node-config.json configuration/config.json
mv shelley-genesis.json byron-genesis.json alonzo-genesis.json conway-genesis.json configuration/
```

And move your genesis delegate keys to their corresponding delegate nodes:

```bash
mv delegate-keys/byron.000* delegate-keys/shelley.000* bft0/
mv delegate-keys/byron.001* delegate-keys/shelley.001* bft1/
```

To simplify the process, create bash scripts to start your nodes. Bft0 will run on port 3000:

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

And bft1 on port 3001:  

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

Now give your scripts executable permission:

```bash
chmod +x bft0/startnode.sh bft1/startnode.sh 
```

Open a new terminal for each of the nodes. Go to the corresponding bft folder, and run the script from there:

```bash
cd bft0
./startnode.sh
```

Repeat for bft1. The nodes will idle until the start time is reached, and then they will start producing blocks.  

Now you can set the environment variable `CARDANO_NODE_SOCKET_PATH`:

```bash
export CARDANO_NODE_SOCKET_PATH=~/cluster/bft0/bft0.socket
```

Next, query the tip of the chain to make sure everything is working as expected:  

```bash
cardano-cli query tip --testnet-magic 42
```

Great, you have a network on the Byron era running locally!  
