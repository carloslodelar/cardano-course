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
--out-file names.poll
```

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

Note: A serialized version of the poll suitable for sharing with participants has been generated at 'names.poll'.
```

Let's take a look to the serialized version of the poll

```
cat names.poll

{
    "type": "GovernancePoll",
    "description": "An on-chain poll for SPOs: Best name for a dog?",
    "cborHex": "a1185ea200817442657374206e616d6520666f72206120646f673f0183816643686565746f816348616d8165526f63636f"
}
```

Participants (SPO's) will use `names.poll` file to create and submit their responses&#x20;

### Answering the poll

Use `answer-poll` to create a response. You can use the `--answer`option to record your answer right away, or omit it to access the interactive method.&#x20;

Note that in any case we are redirecting the output to `poll-answer.json`

#### Using --answer

```
cardano-cli governance answer-poll \
--poll-file names.poll \
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
--poll-file names.poll > poll-answer.json
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

#### Submit the response

