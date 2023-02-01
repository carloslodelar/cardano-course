---
description: 'Objective: Learn how to run cardano-node and connect to different networks'
cover: .gitbook/assets/RUNNING (1).png
coverY: 410
---

# 1.2 Running the node and connecting to a network

### Public networks

The latest supported networks can be found at [https://book.world.dev.cardano.org/environments.html](https://book.world.dev.cardano.org/environments.html)

### Get configuration files for Pre-Production Testnet

Lets create a directory for our node files:

```bash
mkdir preprod
cd preprod
```

Download the configuration files&#x20;

```bash
wget https://book.world.dev.cardano.org/environments/preprod/config.json
wget https://book.world.dev.cardano.org/environments/preprod/topology.json
wget https://book.world.dev.cardano.org/environments/preprod/byron-genesis.json
wget https://book.world.dev.cardano.org/environments/preprod/shelley-genesis.json
wget https://book.world.dev.cardano.org/environments/preprod/alonzo-genesis.json
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
export CARDANO_NODE_SOCKET_PATH="$HOME/preprod/node.socket"
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

