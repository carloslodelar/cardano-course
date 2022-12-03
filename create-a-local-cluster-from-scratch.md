# Create a local cluster

Often you will find useful to deploy a local cluster. We provide tools that allow you to deploy a local cluster with little effort. We will use those tools later.  For now we will do it manually.  Hopefully this will help having a better understanding of what happens behind the scenes when using the automated tools.&#x20;

We will start our cluster in Byron era, and upgrade it all the way to Babbage.&#x20;

### Configure the network and nodes

#### Requirements:

* A linux or macOS machine, see the [Requirements](https://github.com/input-output-hk/cardano-node/blob/master/doc/getting-started/install.md) for details.&#x20;
* cardano-node and cardano-cli in your $PATH

{% hint style="info" %}
Note: It would be more efficient to have a script for most of the tasks that we will perform, since our intention is making the process as transparent and clear as possible, we will limit the use of scripts.
{% endhint %}

To deploy a local cluster we need:&#x20;

1. configuration file
2. topology files&#x20;
3. Byron genesis file
4. Shelley genesis file&#x20;
5. Alonzo genesis file&#x20;

#### 1. The configuration files

We will use the[ Cardano World Testnet Templates ](https://github.com/input-output-hk/cardano-world/tree/master/nix/cardano/environments/testnet-template)to generate our genesis files but first, let's create a directory for our project

```bash
mkdir -p cluster/template
cd cluster
```

Download the template files and save it in the template folder we just created.&#x20;

<pre class="language-bash" data-overflow="wrap"><code class="lang-bash"><strong>wget -P template/ https://raw.githubusercontent.com/input-output-hk/cardano-world/master/nix/cardano/environments/testnet-template/alonzo.json
</strong>wget -P template/ https://raw.githubusercontent.com/input-output-hk/cardano-world/master/nix/cardano/environments/testnet-template/byron.json
wget -P template/ https://raw.githubusercontent.com/input-output-hk/cardano-world/master/nix/cardano/environments/testnet-template/config.json
wget -P template/ https://raw.githubusercontent.com/input-output-hk/cardano-world/master/nix/cardano/environments/testnet-template/shelley.json
</code></pre>

Our system will start in Byron era. We will start with 2 BFT nodes. We will add 2 Stake Pools when we transition to Shelley era.  Let's make a directory for each of the nodes and a configuration folder for our configuration files.&#x20;

```bash
mkdir -p bft0 bft1 configuration
```

We want our bft nodes to connect to each other. Leet's use port 3000 for node bft0, and port 3001 for node bft1. Now we can create a topology file for each of our nodes. For now we will use classic topology (not P2P).

So we create the topology file for bft0 with

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

For bft1

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

Our network will start in Byron era and we will upgrade it all the way up to Babbage era, the `byron.json` file that we downloaded is our template, we will use it together with cardano-cli to generate our `byron-genesis.json` file.&#x20;

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

Note that this template uses 200 millisecond slots on Byron era. Mainnet used 20 second slots during the Byron era because blocks need to travel the world and 20 seconds ensured that everybody had received the previous block before forging a new one.  In our local cluster we will use 4 second slots in Byron and 0.2 seconds slots on the following eras.  &#x20;



The rest of the parameters match mainnet ones. For detailed information about the parameters, see:[ Byron genesis data format ](https://github.com/input-output-hk/cardano-node/blob/master/doc/reference/byron-genesis.md)

Let's make a few changes to our shelley.json template

```
sed -i template/shelley.json \
-e 's/"updateQuorum": 3/"updateQuorum": 2/' \
-e 's/"maxLovelaceSupply": 45000000000000000/"maxLovelaceSupply": 45000000000000/' \
-e 's/"epochLength": 432000/"epochLength": 9000/' \
-e 's/"securityParam": 108/"securityParam": 45/' \
-e 's/"slotLength": 1/"slotLength": 0.20/'
```

And a few changes to the config.json template

```
 sed -i template/config.json \
 -e 's/"EnableP2P": true/"EnableP2P": false/' \
 -e 's/"TestEnableDevelopmentNetworkProtocols": true/"TestEnableDevelopmentNetworkProtocols": false/' \
 -e 's/"TestEnableDevelopmentHardForkEras": true/"TestEnableDevelopmentHardForkEras": false/' \
 -e 's/"LastKnownBlockVersion-Major": 3/"LastKnownBlockVersion-Major": 1/' \
 -e 's/"LastKnownBlockVersion-Minor": 1/"LastKnownBlockVersion-Minor": 0/' \
 -e 's/"TestShelleyHardForkAtEpoch": 0/"TestShelleyHardForkAtEpoch": /' \
 -e 's/"TestAllegraHardForkAtEpoch": 0/"TestAllegraHardForkAtEpoch": /' \
 -e 's/"TestMaryHardForkAtEpoch": 0/"TestMaryHardForkAtEpoch": /' \
 -e 's/"TestAlonzoHardForkAtEpoch": 0/"TestAlonzoHardForkAtEpoch": /' \
 -e 's/""minSeverity": "Debug"/"minSeverity": "Info"/' 
```

Now we can use the magic of cardano-cli genesis create-cardano

```
cardano-cli genesis create-cardano \
--genesis-dir ./ \
--gen-genesis-keys 2 \
--start-time $(date -u -d "now + 5 minutes" +%FT%Tz) \
--supply 30000000000000 \
--security-param 45 \
--slot-length 200 \
--slot-coefficient 5/100 \
--testnet-magic 42 \
--byron-template template/byron.json \
--shelley-template template/shelley.json \
--alonzo-template template/alonzo.json \
--node-config-template template/config.json
```

Let's move our genesis files to configuration directory to keep tings in orther

```
mv node-config.json configuration/config.json
mv shelley-genesis.json configuration/
mv byron-genesis.json configuration/
mv alonzo-genesis.json configuration/
```

And let's move our delegate keys to their corresponging delegate nodes:

```
mv -t bft0/ delegate-keys/byron.000* delegate-keys/shelley.000* 
mv -t bft1/ delegate-keys/byron.001* delegate-keys/shelley.001*
```

To make our lives easier we will create scripts to start our nodes.&#x20;

For bft0:

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

For bft1:

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

Now let's give them executable permission:

```bash
chmod +x bft0/startnode.sh bft1/startnode.sh 
```

Now open a new terminal for each of the nodes, go to the corresponding bft folder and run the script from there:

```bash
cd bft0
./startnode.sh
```

Repeat for bft1 and bft2, The nodes will idle until start time is reached, then they will start producing blocks.&#x20;

Now you can set the environment variable  `CARDANO_NODE_SOCKET_PATH`

```bash
export CARDANO_NODE_SOCKET_PATH=~/cluster/bft0/bft0.socket
```

Let's query the tip of the chain to make sure everything is working as expected:&#x20;

```bash
cardano-cli query tip --testnet-magic 42
```

Great we have a network on Byron era running locally!&#x20;
