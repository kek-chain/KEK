---
# Whitepaper 2.0 ![KeK Logo](frogp.png)
>dev@kekchain.com
---
# Genesis  File
___
This docunment explains how KeK chain is Structured

## What is a Genesis File

A genesis file is a JSON file which defines the initial state of your blockchain. It can be seen as height 0 of your blockchain. The first block, at height 1, will reference the genesis file as its parent.

___

## Explanantion

* Chain ID  
The chain ID is a property of the chain managed by the node. It is used for replay protection of transactions

  KEK CHAIN ID:

    `TBA`  


* period  
Minimum difference between two consecutive block’s timestamps. Suggested 3s for the testnet .

* epoch  
Number of blocks after which to checkpoint and reset the pending votes. Suggested 100 for testnet

* nonce  
The nonce is the cryptographically secure mining proof-of-work that proves beyond reasonable doubt that a particular amount of computation has been expended in the determination of this token value.
KeK Chain is Set to 0X0.

* timestamp   
  Must be at least the parent timestamp + BLOCK_PERIOD.
* extraData  
    *  EXTRA_VANITY: Fixed number of extra-data prefix bytes reserved for signer vanity. Suggested 32 bytes

    * Signer Info: validator address

    * EXTRA_SEAL bytes (fixed) is the signer’s signature sealing the header.

*   gasLimit  
A scalar value equal to the current chain-wide limit of Gas expenditure per block. High in our case to avoid being limited by this threshold during tests. Note: this does not indicate that we should not pay attention to the Gas consumption of our Contracts.

* difficulty  
    A scalar value corresponding to the difficulty level applied during the nonce discovering of this block. Suggested 0x1 for testnet
* mixHash   
 Reserved for fork protection logic, similar to the extra-data during the DAO. Must be filled with zeroes during normal operation.

* coinbase   
System controled address for collecting block rewards

* number   
Block height in the chain, where the height of the genesis is block 0.

* parentHash  
The Keccak 256-bit hash of the entire parent block’s header (including its nonce and mixhash). Pointer to the parent block, thus effectively building the chain of blocks. In the case of the Genesis block, and only in this case, it's 0.

___

# Account and Address
This default wallet would use a similar way to generate keys as Ethereum, i.e. use 256 bits entropy to generate a 24-word mnemonic based on BIP39, and then use the mnemonic and an empty passphrase to generate a seed; finally use the seed to generate a master key, and derive the private key using BIP32/BIP44 with HD prefix as "44'/60'/", which is the same as Ethereum's derivation path.
___

# Consensus
___
## Infrastructure Components  
**KeK Chain**. It is responsible for holding the staking function to determine validators of KEK through independent election, and the election workflow are performed via staking procedure.
___

## Consensus Protocol
The implement of the consensus engine is named as pBFT implementations of PoSA consensus. This doc will focus more on the difference and ignore the common details.

Before introducing, we would like to clarify some terms:

1. Epoch block. Consensus engine will update validatorSet from NCValidatorSet contract periodly. For now the period is 200 blocks, a block is called epoch block if the height of it is times of 200.
1. Snapshot. Snapshot is an assistant object that help to store the validators and recent signers of blocks.

___

## key Features
___
### Light client security

 Validators set changes take place at the (epoch3f+1) blocks. (f is the size of validatorset before epoch block). Considering the security of light client, we delay f+1 block to let validatorSet change take place.

Every epoch block, validator will query the validatorset from contract and fill it in the extra_data field of block header. Full node will verify it against the validatorset in contract. A light client will use it as the validatorSet for next epoch blocks, however, it can not verify it against contract, it has to believe the signer of the epoch block. If the signer of the epoch block write a wrong extra_data, the light client may just go to a wrong chain. If we delay f+1 block to let validatorSet change take place, the wrong epoch block won’t get another f+1 subsequent blocks that signed by other validators, so that the light client is free of such attack

### System transaction
The consensus engine may invoke system contracts, such transactions are called system transactions. System transactions is signed by the validator who is producing the block. For the witness node, will generate the system transactions (without signature) according to its intrinsic logic and compare them with the system transactions in the block before applying them.

### Enforce backoff

In pBFT consensus protocol, out-of-turn validators have to wait a randomized amount of time before sealing the block. It is implemented in the client-side node software and works with the assumption that validators would run the canonical version. However, given that validators would be economically incentivized to seal blocks as soon as possible, it would be possible that the validators would run a modified version of the node software to ignore such a delay. To prevent validator rushing to seal a block, every out-turn validator will get a specified time slot to seal the block. Any block with an earlier blocking time produced by an out-turn validator will be discarded by other witness node.

### How to Produce a new block  
* Step 1: Prepare  
A validator node prepares the block header of next block.
  * Load snapshot from cache or database.

  * Every epoch block, will store validators set message in extraData field of block header to facilitate the implement of light client.

  * The coinbase is the address of the validator

* Step2: FinalizeAndAssemble
  * If the validator is not the in turn validator, will call liveness slash contract to slash the expected validator and generate a slashing transaction.
  * If there is gas-fee in the block, will distribute 1/16 to system reward contract, the rest go to validator contract.

* Step3: Seal
  The final step before a validator broadcast the new block.
  * Sign all things in block header and append the signature to extraData.
  * If it is out of turn for validators to sign blocks, an honest validator it will wait for a random reasonable time.

  ### How to Validate/Replay a block

  * Step 1:  VerifyHeader  
Verify the block header when receiving a new block.
  * Verify the signature of the coinbase is in `extraData` of the `blockheader`

  * Compare the block time of the blockHeader and the expected block time that the signer suppose to use, will deny a blockerHeader that is smaller than expected. It helps to prevent a selfish validator from rushing to seal a block.

  * The `coinbase` should be the signer and the difficulty should be expected value.

* Step2: Finalize
  * If it is an epoch block, a validator node will fetch validatorSet from NCValidatorSet and compare it with extra_data.
  * If the block is not generated by inturn validatorvalidaror, will call slash contract. if there is gas-fee in the block, will distribute 1/16 to system reward contract, the rest go to validator contract.
  * The transaction generated by the consensus engine must be the same as the tx in block.

### Signature   
The signature of the coinbase is in extraData of the blockheader, the structure of extraData is: epoch block. 32 bytes of extraVanity + f*{20 bytes of validator address} + 65 bytes of signature. none epoch block. 32 bytes of extraVanity + 65 bytes of signature. The signed content is the Keccak256 of RLP encoded of the block header.

### Security and Finality
  Given there are more than 1/2*f+1 validators are honest, PoSA based networks usually work securely and properly. However, there are still cases where certain amount Byzantine validators may still manage to attack the network, e.g. through the “Clone Attack”. To secure as much as BC, KEK users are encouraged to wait until receiving blocks sealed by more than 2/3*f+1 different validators. In that way, the KEK can be trusted at a similar security level to BC and can tolerate less than 1/3*f Byzantine validators.

With 21 validators, if the block time is 5 seconds, the 2/3*f+1 different validator seals will need a time period of (2/3*21+1)*5 = 75 seconds. Any critical applications for KEK may have to wait for 2/3*f+1 to ensure a relatively secure finality. However, besides such an arrangement, KEK does introduce Slashing logic to penalize Byzantine validators for double signing or instability. This Slashing logic will expose the malicious validators in a very short time and make the Clone Attack very hard or extremely non-economic to execute. With this enhancement, 1/2*f+1 or even fewer blocks are enough as confirmation for most transactions.

### Potential Issue
#### Extending the ruling of the current validator set via temporary censorship

If the transaction that updates the validator is sent to the KEK right on the epoch period, then it is possible for the in-turn validator to censor the transaction and not change the set of validators for that epoch. While a transaction cannot be forever censored without the help of other f+1 validators, by this it can extend the time of the current validator set and gain some rewards. In general, the probability of this scheme can increase by colluding with other validators. It is relatively benign issue that a block may be approximately 5 secs and one epoch being 240 blocks, i.e. 20 mins so the validators could only be extended for another 20 mins.

___
# Key Management
This article is a guide about key management strategy on client side of your Decentralised Application on KEK Chain

## Setup Web3
`web3.js` is a javascript library that allows our client-side application to talk to the blockchain. We configure web3 to communicate via Metamask.

web3.js doc is [here](https://web3js.readthedocs.io/en/v1.2.2/getting-started.html#adding-web3-js)

## Connect KEK Chain
    TBA




