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
--out-file poll.json
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

Note: A serialized version of the poll suitable for sharing with participants has been generated at 'poll.json'.
```
{% endcode %}

Let's take a look to the serialized version of the poll

```
cat poll.json

{
    "type": "GovernancePoll",
    "description": "An on-chain poll for SPOs: Best name for a dog?",
    "cborHex": "a1185ea200817442657374206e616d6520666f72206120646f673f0183816643686565746f816348616d8165526f63636f"
}
```

Participants (SPO's) will use `poll.json`file to create and submit their responses&#x20;

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
