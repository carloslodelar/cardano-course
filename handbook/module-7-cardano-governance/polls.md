# Polls

On cardano-node 8.0.0 we introduced a new subset of commands to create polls among stake pool operators. A poll is official when it is signed using a signature from the genesis delegate keys.&#x20;

{% hint style="info" %}
To practice creating and answering polls we will create a local cluster
{% endhint %}

### Create a poll

```
cardano-cli governance create-poll \
--question "Best name for a dog?" \
--answer "Cheeto" \
--answer "Ham" \
--answer "Rocco" \
--out-file poll.cbor > poll.json
```

{% code overflow="wrap" %}
```
Poll created successfully.
Please submit a transaction using the resulting metadata.

{
    "94": {
        "map": [
            {
                "k": {
                    "int": 0
                },
                "v": {
                    "list": [
                        {
                            "string": "Best name for a dog?"
                        }
                    ]
                }
            },
            {
                "k": {
                    "int": 1
                },
                "v": {
                    "list": [
                        {
                            "list": [
                                {
                                    "string": "Cheeto"
                                }
                            ]
                        },
                        {
                            "list": [
                                {
                                    "string": "Ham"
                                }
                            ]
                        },
                        {
                            "list": [
                                {
                                    "string": "Rocco"
                                }
                            ]
                        }
                    ]
                }
            }
        ]
    }
}

Hint (1): Use '--json-metadata-detailed-schema' and '--metadata-json-file' from the build or build-raw commands.
Hint (2): You can redirect the standard output of this command to a JSON file to capture metadata.

Note: A serialized version of the poll suitable for sharing with participants has been generated at 'poll.cbor'.
```
{% endcode %}

Let's take a look to the serialized version of the poll

```
cat poll.cbor

{
    "type": "GovernancePoll",
    "description": "An on-chain poll for SPOs: Best name for a dog?",
    "cborHex": "a1185ea200817442657374206e616d6520666f72206120646f673f0183816643686565746f816348616d8165526f63636f"
}
```

Participants (SPO's) will use `poll.cbor`file to create and submit their responses&#x20;

The Delegate key holder proposing the poll will publish the poll in a transaction, to build the transaction we do:

{% code overflow="wrap" %}
```
cardano-cli transaction build \
--babbage-era \
--testnet-magic 42 \
--tx-in $(cardano-cli query utxo --address $(cat utxo-keys/utxo1.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -r 'keys[0]') \
--change-address $(cat utxo-keys/utxo1.addr) \
--metadata-json-file poll.json \
--json-metadata-detailed-schema \
--required-signer-hash $(cat delegate-keys/delegate1.hash) \
--out-file question.tx
```
{% endcode %}

Note that when building the transaction can use `--required-signer-hash` (as opposed to `--required-signer`) since the signing keys are on cold storage and build requires access to a live node.\
\
Sign the transaction with the delegate signing key and with a payment signing key.&#x20;

```
cardano-cli transaction sign \
--tx-body-file question.tx \
--signing-key-file delegate-keys/delegate1.skey \
--signing-key-file utxo-keys/utxo1.skey \
--testnet-magic 42 \
--out-file question.tx.signed
```

When we inspect the transaction (question.tx.signed)&#x20;

```
cardano-cli transaction view --tx-file question.signed
```

we should see something like this: &#x20;

{% code overflow="wrap" %}
```

certificates: null
collateral inputs: []
era: Babbage
fee: 173729 Lovelace
inputs:
- 14bd4fe1f04829ed6f55350f21f243774e96f0ea9278ac8905e1a0e53dce0e01#0
metadata:
  '94':
  - - 0
    - - Best name for a dog?
  - - 1
    - - - Cheeto
      - - Ham
      - - Rocco
mint: null
outputs:
- address: addr_test1vzm3wju8c3u5w4vdq2ydes73vqz9kfx40j99cwjjvpefpmcg36av4
  address era: Shelley
  amount:
    lovelace: 299999826271
  datum: null
  network: Testnet
  payment credential key hash: b7174b87c47947558d0288dcc3d160045b24d57c8a5c3a52607290ef
  reference script: null
  stake reference: null
reference inputs: []
required signers (payment key hashes needed for scripts):
- 186c97a1194d2c62e3bfafef7b471cb0bb4ce09322ca44a908e81bee
return collateral: null
total collateral: null
update proposal: null
validity range:
  lower bound: null
  upper bound: null
withdrawals: null
witnesses:
- key: VKey (VerKeyEd25519DSIGN "240a1e444b9ed4d2080dd8ac731510037ea887dc1be26e8ab99d8c09b13864d9")
  signature: SignedDSIGN (SigEd25519DSIGN "8c92614f0fafd716fbdbb76ba83e6ac8e93d0c593acdc500328d8b0b7595d70f5a8cbbde499d9c0969ec41162f505a93e35d7dc3965f9a18d05a934b428de806")
- key: VKey (VerKeyEd25519DSIGN "da962fd3e59db09e724d698914a83bf7eacfbdccae07a78957c03b617f06e26e")
  signature: SignedDSIGN (SigEd25519DSIGN "83feb9bbd588d5933b25bad3da341043af088b5ab99e4f332e89f3e525c18a780909f1bc6da7d63372dda47ae901c63ecf23a62082efba62f35ecc424f5db20f")
```
{% endcode %}

And finally submit the transaction and usual:

```
cardano-cli transaction submit \
--testnet-magic 42 \
--tx-file question.signed
```

We can use _**dbsync**_ to check how it was registered online:

```
cexplorer=# SELECT * FROM tx_metadata WHERE key = 94;
```

```
 id | key |     json                                                              | bytes                                                                                                | tx_id 
----+-----+------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------+-------
 1  | 94  | {"0": ["Best name for a dog?"], "1": [["Cheeto"], ["Ham"], ["Rocco"]]} | \xa1185ea200817442657374206e616d6520666f72206120646f673f0183816643686565746f816348616d8165526f63636f | 11 
 (1 row)
```

```
cexplorer=# SELECT * FROM extra_key_witness;
```

```
 id |                            hash                            | tx_id
----+------------------------------------------------------------+-------
  1 | \x186c97a1194d2c62e3bfafef7b471cb0bb4ce09322ca44a908e81bee |    11
(1 row)
```

Of course, the hash matches the hash of the delegate key. This way, when SPOs see a poll signed with any of the delegate keys (verified by the delegate key hashes) they know this is an official poll.&#x20;

### Answering the poll

Use `answer-poll` to create a response. You can use the `--answer`option to record your answer right away providing the index of your response, or omit it to access the interactive method.&#x20;

Note that in any case we are redirecting the output to `poll-answer.json`

#### Using --answer

```
cardano-cli governance answer-poll \
--poll-file poll.json \
--answer 0 > poll-answer.json
```

{% code overflow="wrap" %}
```
Best name for a dog?
→ Cheeto

Poll answer created successfully.
Please submit a transaction using the resulting metadata.
To be valid, the transaction must also be signed using a valid key
identifying your stake pool (e.g. your cold key).

Hint (1): Use '--json-metadata-detailed-schema' and '--metadata-json-file' from the build or build-raw commands.
Hint (2): You can redirect the standard output of this command to a JSON file to capture metadata.
```
{% endcode %}

#### Responding interactively

```
cardano-cli governance answer-poll \
--poll-file poll.json > poll-answer.json
```

{% code overflow="wrap" %}
```
Best name for a dog?
[0] Cheeto
[1] Ham
[2] Rocco

Please indicate an answer (by index): 0

Poll answer created successfully.
Please submit a transaction using the resulting metadata.
To be valid, the transaction must also be signed using a valid key
identifying your stake pool (e.g. your cold key).


Hint (1): Use '--json-metadata-detailed-schema' and '--metadata-json-file' from the build or build-raw commands.
Hint (2): You can redirect the standard output of this command to a JSON file to capture metadata.
```
{% endcode %}

#### Submit the response in a transaction

```
cardano-cli transaction build \
    --babbage-era \
    --testnet-magic 42 \
    --tx-in $(cardano-cli query utxo --address $(cat utxo-keys/utxo1.addr) --testnet-magic 42 --out-file  /dev/stdout | jq -r 'keys[0]') \
    --change-address $(cat utxo-keys/utxo1.addr) \
    --metadata-json-file poll-answer.json \
    --json-metadata-detailed-schema \
    --required-signer pools/cold1.skey \
    --out-file answer.tx
```

```
cardano-cli transaction sign \
--tx-body-file answer.tx \
--signing-key-file pools/cold1.skey \
--signing-key-file utxo-keys/utxo1.skey \
--testnet-magic 42 \
--out-file answer.signed
```

```
cardano-cli transaction submit \
--testnet-magic 42 \
--tx-file answer.signed

Transaction successfully submitted.
```

#### Verifying Answers <a href="#verifying-answers" id="verifying-answers"></a>

Finally, it’s possible to verify answers seen on-chain using the `governance verify-poll` command. What ‘verify’ means here is two-folds:

* It checks that an answer is valid within the context of a given survey
* It returns the list of signatories key hashes found in the transaction;\
  in the case of a valid submission, one key hash will correspond to a known\
  stake pool id.

```
cardano-cli governance verify-poll \
--poll-file poll.json \
--signed-tx-file answer.signed

Found valid poll answer, signed by:
[
    "bea662a2aaefd5b4c7b92aaa5d8b59d9d22fb7034e5b309d56d54084"
]
```
