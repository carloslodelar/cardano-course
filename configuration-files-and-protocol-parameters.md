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

In addition to **protocol parameters** we have **global constants**. Unlike protocol parameters, global constants can only be updated with a software update which always include increasing the protocol version. An increase in the major version indicates a hard fork, and the minor version a soft fork (meaning old software can validate but not produce new blocks).&#x20;

```
SlotsPerEpoch
SlotsPerKESPeriod
MaxKESEvolutions
StabilityWindow
RandomnessStabilisationWindow
Quorum
MaxMajorPV
ActiveSlotCoeff
NetworkId
MaxLovelaceSupply
securityParam
slotLength
```

Below you will find a description of the protocol parameters and global constants:

|                                                         | Description                                                                                                                                                                             | Value                                                                                                                                                                                                                    |
| ------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| NetworkId                                               | Determines what network, either mainnet or testnet, is expected                                                                                                                         | mainnet                                                                                                                                                                                                                  |
| MaxLovelaceSupply                                       | The total number of lovelace in the system                                                                                                                                              | 45000000000000000                                                                                                                                                                                                        |
| k                                                       | Security parameter, after how many blocks is the blockchain considered to be final, and thus can no longer be rolled back (i.e. what is the maximum allowable length of any chain fork) | 2160                                                                                                                                                                                                                     |
| ActiveSlotCoeff                                         | Fraction of slots that will contain a block (on average). Value f from the Praos paper                                                                                                  | 0.05                                                                                                                                                                                                                     |
| SlotsPerEpoch                                           | The number of slots in an epoch                                                                                                                                                         | 432000                                                                                                                                                                                                                   |
| slotLength                                              | The length of each slot (in seconds)                                                                                                                                                    | 1                                                                                                                                                                                                                        |
| Quorum                                                  | How many of the genesis delegate keys must endorse an update proposal.                                                                                                                  | 5                                                                                                                                                                                                                        |
| <p>protocolVersion<br>major<br>minor</p>                | Current protocol version                                                                                                                                                                | 7.0                                                                                                                                                                                                                      |
| MaxMajorPV                                              | <p>If the major component of the protocol version is larger than MaxMajorPV, there is a</p><p>ObsoleteNode failure</p>                                                                  | Halts below 7.0 nodes                                                                                                                                                                                                    |
| StabilityWindow                                         | The number of slots needed for a block to become stable from the security parameter and the active slot coefficient (3k/f)                                                              | 3k/f                                                                                                                                                                                                                     |
| RandomnessStabilisationWindow                           | <p>Intended to be used only for freezing the</p><p>candidate nonce currently used for the waiting period for creating the reward update (4k/f)</p>                                      |  4k/f                                                                                                                                                                                                                    |
| stakeAddressDeposit                                     | Deposit amount in lovelace required to register stake credential                                                                                                                        | 2000000                                                                                                                                                                                                                  |
| stakePoolDeposit                                        | Deposit amount in lovelace required to register a stake pool                                                                                                                            | 500000000                                                                                                                                                                                                                |
| stakePoolTargetNum                                      | The optimal number of saturated stake pools. Impacts saturation threshold, encouraging growth in number of stake pools                                                                  | 500                                                                                                                                                                                                                      |
| decentralization                                        | Controls how many slots are governed by the genesis nodes via OBFT and which slots are open to any registered stake pool                                                                | <p>Deprecated </p><p></p><p>Shelley - Alonzo: [0-1]</p><p>Babbage: Null </p>                                                                                                                                             |
| SlotsPerKESPeriod                                       | After how many slots will a pool's operational key pair evolve (Key Evolving Signatures)                                                                                                | 129600                                                                                                                                                                                                                   |
| MaxKESEvolutions                                        | The maximum number of times a KES key pair can evolve before a new KES key pair must be generated from the master keys                                                                  | 62                                                                                                                                                                                                                       |
| poolRetireMaxEpoch                                      | Maximum number of epochs within which a pool can be announced to retire, starting from the next epoch                                                                                   | 18                                                                                                                                                                                                                       |
| minPoolCost                                             | Minimum Pool Cost per epoch (in lovelace). Enables pledge effect                                                                                                                        | 340000000                                                                                                                                                                                                                |
| maxTxSize                                               | Maximum transaction size in bytes                                                                                                                                                       | 16384                                                                                                                                                                                                                    |
| maxBlockBodySize                                        | Maximum block size in bytes                                                                                                                                                             | 90112                                                                                                                                                                                                                    |
| maxBlockHeaderSize                                      | Maximum block header size in bytes                                                                                                                                                      | 1100                                                                                                                                                                                                                     |
| maxValueSize                                            | The maximum serialized length (in bytes) of a multi-asset value (token bundle) in a transaction output                                                                                  | 5000                                                                                                                                                                                                                     |
| minUTxOValue                                            | During the Shelley era,  minimum ada value that a transaction output must hold                                                                                                          | Deprecated                                                                                                                                                                                                               |
| utxoCostPerWord                                         | During Mary Allegra and Alonzo this parameter determined the minimum ada value that a transaction output should have to sustain native assets (Mary, Allegra) and Datum hashes (Alonzo) | <p>Deprecated</p><p></p>                                                                                                                                                                                                 |
| utxoCostPerByte                                         | <p>In the Babbage era,  transaction outputs will be required to contain at least</p><p><code>(160 + |serialized_output|) * coinsPerUTxOByte</code></p><p>many lovelace</p>              | <p>4310</p><p><a href="https://cips.cardano.org/cips/cip55/">https://cips.cardano.org/cips/cip55/</a></p>                                                                                                                |
| treasuryCut                                             | Proportion of the reward pot that will move to the treasury at each rewards distribution                                                                                                | 0.2                                                                                                                                                                                                                      |
| monetaryExpansion                                       | Every epoch, the total amount of ada in circulation is increased by monetaryExpanision % of the Ada reserves is moved to the rewards pot                                                | <p>3.0e-3   <br>(.003)</p>                                                                                                                                                                                               |
| poolPledgeInfluence                                     | Determines stake pool owner-stake influence on pool rewards                                                                                                                             | 0.3                                                                                                                                                                                                                      |
| txFeeFixed                                              | Base transaction fee (in lovelace)                                                                                                                                                      | 155381                                                                                                                                                                                                                   |
| txFeePerByte                                            | Additional transaction fee per byte of data (in lovelace)                                                                                                                               | 44                                                                                                                                                                                                                       |
| <p>costModels<br>PlutusV1<br>PlutusV2</p>               | <p>Every phase-2 scripting language converts the calculated execution cost into a number of</p><p>ExUnits using a cost model, which depends on the language</p>                         | <p>PV1 <br>PV2</p>                                                                                                                                                                                                       |
| <p>executionUnitPrices<br>priceMemory<br>priceSteps</p> | <p>Prices consists of two rational numbers that correspond to the components</p><p>of ExUnits: the price per unit of memory and price per reduction step</p>                            | <p>priceMemory: 5.77e-2</p><p>(.0577)<br>priceSteps: 7.21e-5<br>(.0000721)</p>                                                                                                                                           |
| <p>maxBlockExecutionUnits<br>memory<br>steps</p>        | Maximum ExUnits allowed in a block                                                                                                                                                      | <p>memory: 62000000<br>steps: 40000000000</p>                                                                                                                                                                            |
| collateralPercentage                                    | <p>The percentage of the total transaction fee </p><p>its collateral must (at minimum) cover</p>                                                                                        | 150                                                                                                                                                                                                                      |
| maxCollateralInputs                                     | <p>Limits the total number of collateral inputs, and thus the total number of additional</p><p>signatures that must be checked during validation</p>                                    | 3                                                                                                                                                                                                                        |
| extraPraosEntropy                                       | This provides additional certainty that the blockchain has not been compromised by the seed key holders. Redundant once the system is sufficiently decentralized                        | <p>Depracated</p><p></p><p>See <a href="https://iohk.io/en/blog/posts/2021/03/29/the-secure-transition-to-decentralization/">https://iohk.io/en/blog/posts/2021/03/29/the-secure-transition-to-decentralization/</a></p> |



#### Resources

* [Cardano-ledger ](https://github.com/input-output-hk/cardano-ledger)repository contains the formal specifications, executable models, and implementations of the Cardano Ledger.
* [Cardano improvement proposal 9](https://cips.cardano.org/cips/cip9/)
* [Cardano improvement proposal 28](https://cips.cardano.org/cips/cip28/)
* [Cardano improvement proposal 55](https://cips.cardano.org/cips/cip55/)
* Original implementation, Cardano-SL [https://github.com/serokell/cardano-sl/blob/master/docs/configuration.md](https://github.com/serokell/cardano-sl/blob/master/docs/configuration.md)&#x20;
