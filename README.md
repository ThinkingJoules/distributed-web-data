# linked-data-thoughts
Exploration of ideas on how to merge concetps from GunDB, AtomicData, IPFS, and blockchain(s) into a single system. The goal is to allow for addressable, retreievable, and interoperable data that enables applications to be reduced to simply a UI for interacting with and creating the data.

Working Name for this concept (not a specific implementation): Digital Reality (DR)

**WIP**

# 10,000length view
* Build a [tokenless](https://github.com/ThinkingJoules/linked-data-thoughts/wiki/'Tokenless'-Proof-of-Stake-Blockchain-(PoSW-=-Proof-of-Staked-Work)) Layer0 Platform to enable the building of addressable semantic symbols using subnets.
* First L1 should be a PKI system that will allow many keys to be associated with a Long Lived Identifier (LLI), your Identity.
  * Other L1's can integrate the LLI into their operation for an understand of authorship.
* Non-blockchain systems can be built that use the LLI to create a resolvable IP address
* One or more L1's can be built to generate a Semantic Ontology (SO) to define common data types, understanding of properties, units, conversions between units, encodings, etc.
* Leverage the LLI-> IP resolver to build a system to retrieve/cache/replicate SO data between Identities.

The user of this system should never have to acquire any tokens*. In theory, if a user isn't creating any new foundational symbols, they would only need to expend some Proof of Work to create an identity or update their keys on the PKI chain.

(* No tokens, but if they want to host their own SO data they might have to pay for hosting, self hosting is of course always an option)

# My current vision
The problem with blockchains, is that everything on a single chain must be communicated to every participant. Avalanche and Polkadot solve this problem with creating 'subnets' and 'parachains' respectively. The main idea is to not encumber validators in state they don't care about. If someone wants a micro transaction chain with thousands of TPS they can define their own chain with proper capacity to keep fees low and their system working as designed, and they can do that without being constrained to primitives in a generic one-size-fits-all VM. Monolithic designs like Ethereum are clearly struggling with that design pattern.

## Chain of Chains
If all of these [PoSW](https://github.com/ThinkingJoules/linked-data-thoughts/wiki/'Tokenless'-Proof-of-Stake-Blockchain-(PoSW-=-Proof-of-Staked-Work)) systems were registered on some top level PoSW chain (sort of like the P-Chain in Avalanche) then that would let everyone know what chains exist and other info about them.
### Namespace and addressability
If there is a single registration point, all subnets can be in relation to them. They could have their own sort of URL convention 

```[scheme]://[acronym for this overall system].[Registration TxnID for Subnet].[subnet? scheme? specific suffix]```

Needs work, but I'm sure you see where I'm going here. I would put the URL path in order of high level to low level. The subnet could be thought of as a TLD, but for prefix reasons, I would probably move it's position for this URL usecase.
Instead of TxnID (hash) could we use to unsigned integers? Like (BlockNumber,TxnNumberInBlock)? It would be way better than hashes for compactness. As long as we can know the transaction has reached a state of irreversibility, I don't see why this wouldn't work. Need to explore failure modes, but two uints would be slick.

## Distributed PKI
My personal goal is building this aspect of the Digital Reality. I would really like a static identifier that remains unchanged between key rotations. Allowing more than one key per logical identity also could allow ranking of keys to allow devices to sign stuff for other applications, but for example, not be able to add more keys to the identity.

Creating a long lived identifier (LLI) could allow any application to key information to a specific user, and not a specific public key.

## Low Level Primitives
I could imagine one of the early chains would define specifications for low level primitives that could be used and referenced in other projects (blockchain or not). The txn fee would probably be a crazy high amount of work. Something like days to weeks (maybe even a month?) of PoW to avoid unnecessary or useless primitives to be added (of course, someone still needs to implement it in software in some sort of client somewhere). If you could use the txn coordinates instead of hash, you could use this as a prefix on all the binary encodings so all data would be identifiable regardless of context. With sufficient work requirement you could easily end up with only a 2-3 byte prefix for all identifier tags.

