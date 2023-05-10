---
cover: ../.gitbook/assets/CABAL (1).png
coverY: 0
---

# Running the node and connecting to a network

{% hint style="success" %}
This tutorial has been updated to work with [cardano-node 8.0.0](https://github.com/input-output-hk/cardano-node/releases/tag/8.0.0)
{% endhint %}

### Public networks

The latest supported networks can be found at [https://book.world.dev.cardano.org/environments.html](https://book.world.dev.cardano.org/environments.html)

### Get configuration files for Preview Testnet

Lets create a directory for our node files:

```bash
mkdir preview
cd preview
```

Download the configuration files&#x20;

```bash
wget https://book.world.dev.cardano.org/environments/preview/config.json
wget https://book.world.dev.cardano.org/environments/preview/topology.json
wget https://book.world.dev.cardano.org/environments/preview/byron-genesis.json
wget https://book.world.dev.cardano.org/environments/preview/shelley-genesis.json
wget https://book.world.dev.cardano.org/environments/preview/alonzo-genesis.json
wget https://book.world.dev.cardano.org/environments/preview/conway-genesis.json
```

```bash
cardano-node run --help
```

```
Usage: cardano-node run [--topology FILEPATH]
                          [--database-path FILEPATH]
                          [--socket-path FILEPATH]
                          [ --tracer-socket-path-accept FILEPATH
                          | --tracer-socket-path-connect FILEPATH
                          ]
                          [--byron-delegation-certificate FILEPATH]
                          [--byron-signing-key FILEPATH]
                          [--shelley-kes-key FILEPATH]
                          [--shelley-vrf-key FILEPATH]
                          [--shelley-operational-certificate FILEPATH]
                          [--bulk-credentials-file FILEPATH]
                          [--host-addr IPV4]
                          [--host-ipv6-addr IPV6]
                          [--port PORT]
                          [--config NODE-CONFIGURATION]
                          [--snapshot-interval SNAPSHOTINTERVAL]
                          [--validate-db]
                          [ --mempool-capacity-override BYTES
                          | --no-mempool-capacity-override
                          ]
```

### Running a passive node on Preview Testnet

```bash
cardano-node run --topology topology.json \
--database-path db \
--socket-path node.socket \
--port 3001 \
--config config.json 
```

Open a new terminal and Create the CARDANO\_NODE\_SOCKET\_PATH environment variable

```bash
export CARDANO_NODE_SOCKET_PATH="$HOME/preview/node.socket"
```

Check that the node is syncing

```
cardano-cli query tip --testnet-magic 2
```

### Running a mainnet node

Create a directory for the mainnet node

```
mkdir mainnet
cd mainnet
```

Get the configuration files

```
wget https://book.world.dev.cardano.org/environments/mainnet/config.json
wget https://book.world.dev.cardano.org/environments/mainnet/topology.json
wget https://book.world.dev.cardano.org/environments/mainnet/byron-genesis.json
wget https://book.world.dev.cardano.org/environments/mainnet/shelley-genesis.json
wget https://book.world.dev.cardano.org/environments/mainnet/alonzo-genesis.json
wget https://book.world.dev.cardano.org/environments/mainnet/conway-genesis.json
```

```
cardano-node run --topology topology.json \
--database-path db \
--socket-path node.socket \
--port 3001 \
--config config.json 
```

On a new terminal

```
export CARDANO_NODE_SOCKET_PATH="$HOME/mainnet/node.socket"
```

