---
cover: .gitbook/assets/config files.png
coverY: 0
---

# 2.1 Protocol parameters

We can think of the protocol parameters as the set of rules that govern the Cardano.

Let's explore the protocol parameters with the Cardano-CLI. Note that we will use mainnet this time as testnets might have different values on the parameters. Also note that we are simplifying the out by colapsing the cost models.&#x20;

```
$ cardano-cli query protocol-parameters --mainnet

> {
    "collateralPercentage": 150,
    "costModels": {
        "PlutusScriptV1": {...},
        "PlutusScriptV2": {...}
    },
    "decentralization": null,
    "executionUnitPrices": {
        "priceMemory": 5.77e-2,
        "priceSteps": 7.21e-5
    },
    "extraPraosEntropy": null,
    "maxBlockBodySize": 90112,
    "maxBlockExecutionUnits": {
        "memory": 62000000,
        "steps": 40000000000
    },
    "maxBlockHeaderSize": 1100,
    "maxCollateralInputs": 3,
    "maxTxExecutionUnits": {
        "memory": 14000000,
        "steps": 10000000000
    },
    "maxTxSize": 16384,
    "maxValueSize": 5000,
    "minPoolCost": 340000000,
    "minUTxOValue": null,
    "monetaryExpansion": 3.0e-3,
    "poolPledgeInfluence": 0.3,
    "poolRetireMaxEpoch": 18,
    "protocolVersion": {
        "major": 7,
        "minor": 0
    },
    "stakeAddressDeposit": 2000000,
    "stakePoolDeposit": 500000000,
    "stakePoolTargetNum": 500,
    "treasuryCut": 0.2,
    "txFeeFixed": 155381,
    "txFeePerByte": 44,
    "utxoCostPerByte": 4310,
    "utxoCostPerWord": null
}
```

So, the CLI is getting the protocol parameters from the Ledger state, this means that these are the currently active parameters and their active values. Note that those that are set to null are parameters that have been deprecated.&#x20;

Protocol parameters can be updated, via an update proposal.&#x20;

In addtion to **Protocol parameters** we have **Global constants**. Unlike protocol parameters, Global constants can only be updated with a Softfork or a Hardfork. &#x20;





* Cardano-SL [https://github.com/serokell/cardano-sl/blob/master/docs/configuration.md](https://github.com/serokell/cardano-sl/blob/master/docs/configuration.md)&#x20;
* Cardano-ledger repository contains the formal specifications, executable models, and implementations of the Cardano Ledger.

{% embed url="https://github.com/input-output-hk/cardano-ledger" %}

{% embed url="https://cips.cardano.org/cips/cip9/" %}
CIP9&#x20;
{% endembed %}
