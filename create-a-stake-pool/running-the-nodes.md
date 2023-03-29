# Running the nodes

## Prepare topology files

#### Block Producer

A basic setup where x.x.x.x and y.y.y.y are IP addresses of our own relays.&#x20;

* `Valency` determines the total number of connections to local roots, i.e. if we want a hot connection with both x.x.x.x and y.y.y.y relays, we use `Valency 2.`
* Don't use PublicRoots in our Block Producer. &#x20;
* `UseLedgerAfterSlot` is set to -1  so that we never use ledger peers on the block producer.&#x20;

```
{
   "localRoots":[
      {
         "accessPoints":[
            {
               "address":"x.x.x.x",
               "port":3000
            },
            {
               "address":"y.y.y.y",
               "port":3000
            }
         ],
         "advertise":false,
         "valency":2
      }
   ],
   "publicRoots":[
      {
         "accessPoints":[
            
         ],
         "advertise":false
      }
   ],
   "useLedgerAfterSlot":-1
}
```

#### Relays

* Our relays will connect to the block producer z.z.z.z and to our other relay y.y.y.y.&#x20;
* We use Valency 2 to tell the node maintain a hot connection with both nodes.&#x20;
* Here we can use IOG relays (and any other trusted peer) under Public Roots&#x20;
* Set `useLedgerAfterSlot: 1000000.` We just need to make sure that the target slot is not too old, in particular if we are syncing the chain for the first time.&#x20;

```
{
   "localRoots":[
      {
         "accessPoints":[
            {
               "address":"z.z.z.z",
               "port":3000
            },
            {
               "address":"y.y.y.y",
               "port":3000
            }
         ],
         "advertise":false,
         "valency":2
      }
   ],
   "publicRoots":[
      {
         "accessPoints":[
            {
               "address":"preview-node.world.dev.cardano.org",
               "port":30002
            }         
         ],
         "advertise":false
      }
   ],
   "useLedgerAfterSlot":1000000
}
```

## Prepare startup scripts

For the block producer

```bash
#!/bin/bash

# Configuration variables
TOPOLOGY_FILE="configuration/topology.json"
CONFIG_FILE="configuration/config.json"
DATABASE_PATH="db"
SOCKET_PATH="socket/node.socket"
KES_PATH="keys/kes.skey"
VRF_PATH="keys/vrf.skey"
OPCERT_PATH="keys/opcert.cert"
HOST_ADDR="0.0.0.0"
PORT="3000"

cardano-node run \
    --topology "${TOPOLOGY_FILE}" \
    --database-path "${DATABASE_PATH}" \
    --socket-path "${SOCKET_PATH}" \
    --host-addr "${HOST_ADDR}" \
    --shelley-kes-key "${KES_PATH}"  \ 
    --shelley-vrf-key "${VRF_PATH}" \
    --shelley-operational-certificate "${OPCERT_PATH}" \
    --port "${PORT}" \
    --config "${CONFIG_FILE}" 
```

For the Relays

```bash
#!/bin/bash

# Configuration variables
TOPOLOGY_FILE="configuration/topology.json"
CONFIG_FILE="configuration/config.json"
DATABASE_PATH="db"
SOCKET_PATH="socket/node.socket"
HOST_ADDR="0.0.0.0"
PORT="3000"

cardano-node run \
    --topology "${TOPOLOGY_FILE}" \
    --database-path "${DATABASE_PATH}" \
    --socket-path "${SOCKET_PATH}" \
    --host-addr "${HOST_ADDR}" \
    --port "${PORT}" \
    --config "${CONFIG_FILE}"
```

{% hint style="info" %}
**If you get an error about VRF keys:**\
****\
**VRF private key file /keylocation/vrf.skey has “other” file permissions. Please remove all “other” file permissions.**\
**``**\
**`Fix it with`**&#x20;

**``**

**chmod 0400 vrf.skey**

**`OR`**

**`chmod og-rwx vrf.skey`**
{% endhint %}

### Setup cardano-node to run as systemd service

It will be usefull to set your time zone to UTC

```
sudo timedatectl set-timezone UTC
```

Create the cardano-node.service file. We will save on `/etc/systemd/system/`

```
sudo nano /etc/systemd/system/cardano-node.service
```

```
# The Cardano Node service (part of systemd)
# file: /etc/systemd/system/cardano-node.service

 [Unit]
Description       = Cardano Node Service
Wants             = network-online.target
After             = network-online.target

[Service]
User=clr
Type=simple
WorkingDirectory=/home/clr
ExecStart=/home/clr/startnode.sh
KillSignal=SIGINT
RestartKillSignal=SIGINT
TimeoutStopSec=300
LimitNOFILE=32768
Restart=always
RestartSec=5
SyslogIdentifier=cardano-node

 [Install]
WantedBy          = multi-user.target
```

```
sudo systemctl daemon-reload
sudo systemctl enable cardano-node.service
```

Useful systemctl commands:

```
systemctl --help
systemctl [OPTIONS...] COMMAND ...


Query or send control commands to the system manager.

Unit Commands:
  list-units [PATTERN...]             List units currently in memory
  list-sockets [PATTERN...]           List socket units currently in memory,
                                      ordered by address
  list-timers [PATTERN...]            List timer units currently in memory,
                                      ordered by next elapse
  start UNIT...                       Start (activate) one or more units
  stop UNIT...                        Stop (deactivate) one or more units
  reload UNIT...                      Reload one or more units
  restart UNIT...                     Start or restart one or more units
  try-restart UNIT...                 Restart one or more units if active
  reload-or-restart UNIT...           Reload one or more units if possible,
                                      otherwise start or restart
  try-reload-or-restart UNIT...       If active, reload one or more units,
                                      if supported, otherwise restart
  isolate UNIT                        Start one unit and stop all others
  kill UNIT...                        Send signal to processes of a unit
  clean UNIT...                       Clean runtime, cache, state, logs or
                                      configuration of unit
  is-active PATTERN...                Check whether units are active
  is-failed PATTERN...                Check whether units are failed
  status [PATTERN...|PID...]          Show runtime status of one or more units
```

For example:

```
sudo systemctl stop cardano-node.service
```

```
sudo systemctl start cardano-node.service
```

**Use journalctl to inspect you node logs:**

Follow the logs in real time

```
journalctl --unit=cardano-node --follow
```

Show only the actual message (pure node logs) without any metadata from journalctl

```
journalctl --unit=cardano-node --follow --output=cat
```

Filter logs from a specific date

```
journalctl --unit=cardano-node --since=today
```

Delete old logs

```
sudo journalctl --vacuum-time=5d
```

{% hint style="info" %}
Useful resources: \


[https://www.man7.org/linux/man-pages/man1/systemctl.1.html](https://www.man7.org/linux/man-pages/man1/systemctl.1.html)

[https://www.man7.org/linux/man-pages/man1/journalctl.1.html](https://www.man7.org/linux/man-pages/man1/journalctl.1.html)
{% endhint %}
