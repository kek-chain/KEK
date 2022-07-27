---
# Table of Comparison
| Parameter | Ethereum | Solana | KeK Chain |
|-----------|----------|--------|-----------|
| TPS| ~ 7 tps | ~ 65K tps | ~75k tps  |
| Consensus | PoW | PoSH | PoSA |
| Nodes | 14K | 1250 | 21 |
| Block Time | ~15s  | ~2s | ~2.7s |
| TXn Fee | ~30usd-120usd | ~0.00001usd | ~0.00001usd |

___

# Consensus
---
## Consensus of KeK Chain

### Abstract

We target to design the consensus engine of KEK (KeK Chain) to achieve the following goals:
1. Wait a few blocks to confirm (should be less than Ethereum 1.0), better no fork in most cases.
1. Blocking time should be shorter than Ethereum 1.0, i.e. 3 seconds on KeK Chain
1. No inflation, the block reward is transaction gas fees.

1. As much as compatible as Ethereum.
1. With staking and governance as powerful as Cosmos SDK.

pBFT (Practical Byzantine Fault Tolerance) consensus algorithm allows a distributed system to reach a consensus even when a small amount of nodes demonstrate malicious behavior (such as falsifying information). During information transmission, PBFT uses cryptographic algorithms such as signature, signature verification, and hash to ensure that everything stays irrevocable, unforgeable, and indisputable. It also optimizes the BFT algorithm, reducing its complexity from exponential to polynomial. In a distributed system that constitutes 3f+1 nodes (f represents the number of byzantine nodes), a consensus can be reached as long as no less than 2f+1 non-byzantine nodes are functioning normally.
___

