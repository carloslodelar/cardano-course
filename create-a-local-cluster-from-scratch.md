# Create a local cluster from scratch

Often you will find useful to deploy a local cluster. We provide tools that allow you to deploy a local cluster with little effort. We will use those tools later.  For now we will do it manually. Hopefully this will help having a better understanding of what happens behind the scenes when using the automated tools.

We will start our cluster in Byron era, and upgrade it all the way to Babbage.&#x20;

### Configure the network and nodes

#### Requirements:

* A linux or macOS machine, see the [Requirements](https://github.com/input-output-hk/cardano-node/blob/master/doc/getting-started/install.md) for details.&#x20;
* cardano-node and cardano-cli in your $PATH

{% hint style="info" %}
Note: It would be more efficient to have a script for most of the tasks that we will perform, since our intention is making the process as transparent and clear as possible, we will limit the use of scripts.
{% endhint %}

To deploy a cluster we need:&#x20;

1. configuration file
2. topology files&#x20;
3. genesis files

#### 1. The configuration files

We will use the[ Cardano World Testnet Templates ](https://github.com/input-output-hk/cardano-world/tree/master/nix/cardano/environments/testnet-template)to generate our genesis files but first, let's create a directory for our project

```bash
mkdir -p local-cluster/template
cd local-cluster
```

Download the template files and save it in the template folder we just created.&#x20;

<pre class="language-bash" data-overflow="wrap"><code class="lang-bash"><strong>wget -P template/ https://raw.githubusercontent.com/input-output-hk/cardano-world/master/nix/cardano/environments/testnet-template/alonzo.json
</strong>wget -P template/ https://raw.githubusercontent.com/input-output-hk/cardano-world/master/nix/cardano/environments/testnet-template/byron.json
wget -P template/ https://raw.githubusercontent.com/input-output-hk/cardano-world/master/nix/cardano/environments/testnet-template/config.json
wget -P template/ https://raw.githubusercontent.com/input-output-hk/cardano-world/master/nix/cardano/environments/testnet-template/shelley.json</code></pre>

Our system will start in Byron era. We will start with 2 BFT nodes we will add 2 Stake Pools when we transition to Shelley.  Let's make a directory for each of the nodes and a configuration folder for our configuration files.&#x20;

```bash
mkdir -p bft0 bft1 configuration
```

We want every node to connect to each other,&#x20;

When running the nodes I will use port 3000 for node bft0, and port 3001 for node bft1. Now we can create a topology file for each of our nodes. We will use classic topology (not P2P).

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

And for bft2

#### 3. The genesis file

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

Note that this template uses 200 millisecond slots on Byron era. Mainnet used 20 second slots during the Byron era because blocks need to travel the world and 20 seconds ensured that everybody had received the previous block before forging a new one.  In our local cluster we will use 1 second slots in Byron and 0.5 seconds slots on the following eras.  &#x20;

```
sed -i template/byron.json -e 's/"updateImplicit": "10000"/"updateImplicit": "900"/'
```

The rest of the parameters match mainnet ones. For detailed information about the parameters, see:[ Byron genesis data format ](https://github.com/input-output-hk/cardano-node/blob/master/doc/reference/byron-genesis.md)

Now we need to decide a few things:&#x20;

* **Protocol magic:** the magic number that identifies our network. Let's use 42.
* **Start time:** The time that the system will start producing blocks, this sometime in the future, but not much. Let's give ourselves 20 minutes.&#x20;
* **k:** The security parameter that tells us that any transaction that is more than k blocks deep on the chain is considered stable. We will use k=60  Note that on Byron era epochLength = 10K so our Byron epochs will last  600 slots of 1 second or 10 minutes. &#x20;
* **Number of delegate nodes:** Number of block producer nodes. In Byron we don't have stake pools, but we have block producers that take turns to produce blocks in a round-robin fashion. We will use 2. We will transition to Shelley soon enough and we will add more stake pools.
* **Total Balance:** How much tokens will there be in our system. Let's have 45 million.&#x20;
* **Delegate-share:** The portion of the stake that will be controlled by the block producers (delegates). We will distribute 80% of the total between the 2 delegates.&#x20;
* **Number of avvm keys:** People who purchased Ada at a pre-sale were issued a certificate at the Ada Voucher Vending Machine (AVVM) during the pre-sale period. We can use this to have other addresses with funds in our system. Lets make it 0 in our system
* **avvm entry balance:**  How much will each aavm key gets. Zero in our case.&#x20;

Now we can use cardano-cli to create the genesis file from the `byron.json`

```bash
cardano-cli byron genesis genesis \
--protocol-magic 42 \
--start-time $(date -d "now + 3 minutes" +%s) \
--k 108 \
--n-poor-addresses 0 \
--n-delegate-addresses 2 \
--total-balance 45000000000000 \
--delegate-share 0.80 \
--avvm-entry-count 0 \
--avvm-entry-balance 0  \
--protocol-parameters-file template/byron.json \
--genesis-output-dir byron
```

This will create a `/byron` directory, a `byron-genesis.json` file, keys and delegation certificates, let's see:&#x20;

```bash
tree byron
>
byron
├── delegate-keys.000.key
├── delegate-keys.001.key
├── delegate-keys.002.key
├── delegation-cert.000.json
├── delegation-cert.001.json
├── delegation-cert.002.json
├── genesis.json
├── genesis-keys.000.key
├── genesis-keys.001.key
└── genesis-keys.002.key
```

Let's move our genesis.json file to the configuration folder. We will not transition to Shelley era right away, but the cardano-cli will ask for the Shelley and Alonzo genesis files, so for now we will just use the templates to please the cli.&#x20;

```bash
mv byron/genesis.json configuration/byron-genesis.json
cp template/config.json configuration/
cp template/shelley.json configuration/shelley-genesis.json
cp template/alonzo.json configuration/alonzo-genesis.json
```

We will tweak the config.json file

```bash
 sed -i configuration/config.json \
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

And let's also move our delegate keys and certificates to their corresponding node:&#x20;

```bash
mv -t bft0/ byron/delegation-cert.000.json byron/delegate-keys.000.key
mv -t bft1/ byron/delegation-cert.001.json byron/delegate-keys.001.key
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
--delegation-certificate delegation-cert.000.json \
--signing-key delegate-keys.000.key 
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
--delegation-certificate delegation-cert.001.json \
--signing-key delegate-keys.001.key 
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
export CARDANO_NODE_SOCKET_PATH=~/local-cluster/bft0/bft0.socket
```

Let's query the tip of the chain to make sure everything is working as expected:&#x20;

```bash
cardano-cli query tip --testnet-magic 42
```

Great we have a network on Byron era running locally!&#x20;
