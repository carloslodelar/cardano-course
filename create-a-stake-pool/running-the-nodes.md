# Running the nodes

## Prepare topology file

Block producer

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
         "valency":1
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

Relay

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
         "valency":1
      }
   ],
   "publicRoots":[
      {
         "accessPoints":[
            {
               "address":"relays-new.cardano-mainnet.iohk.io",
               "port":3001
            }
         ],
         "advertise":false
      }
   ],
   "useLedgerAfterSlot":322000
}
```

## Prepare startup script

```
#! /bin/bash

cardano-node run --topology configuration/topology.json \
--database-path db \
--socket-path socket/node.socket \
--host-addr 0.0.0.0 \
--port 3000 \
--config configuration/config.json \
+RTS -N2 -A16m -qg -qb --disable-delayed-os-memory-return -RTS
```

### Setup cardano-node to run as systemd service

To run cardano-node as a systemd service, make sure that cardano-node and cardano-cli binaries are installed on:

```
/usr/local/bin/
```

Then, create the cardano-node.service file. We will save on `/etc/systemd/system/`

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
User              = clr
Type              = simple
WorkingDirectory  = /home/clr
ExecStart         = /home/clr/startnode.sh
KillSignal        = SIGINT
RestartKillSignal = SIGINT
TimeoutStopSec    = 300
LimitNOFILE       = 32768
Restart           = always
RestartSec        = 5
SyslogIdentifier  = cardano-node

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

```
journalctl --unit=cardano-node --follow
```

```
journalctl --unit=cardano-node --since=today
```

{% hint style="info" %}
Useful resources: \


[https://www.man7.org/linux/man-pages/man1/systemctl.1.html](https://www.man7.org/linux/man-pages/man1/systemctl.1.html)

[https://www.man7.org/linux/man-pages/man1/journalctl.1.html](https://www.man7.org/linux/man-pages/man1/journalctl.1.html)
{% endhint %}
