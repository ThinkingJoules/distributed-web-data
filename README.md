# linked-data-thoughts
Exploration of ideas on how to merge concetps from [GunDB](https://github.com/amark/gun), [Atomic Data](https://docs.atomicdata.dev/atomic-data-overview.html), [IPFS](https://github.com/ipfs/ipfs), and [blockchain(s)](https://github.com/ThinkingJoules/linked-data-thoughts/wiki/Blockchains) into a single system. The goal is to allow for addressable, retrievable, and interoperable data that enables applications to be reduced to simply a UI for interacting with and creating the data.

Working name for this concept (not a specific implementation, and I'm open to suggestions on better names): Digital Reality (DR)

**WIP**

# Overview
* Build a [tokenless](https://github.com/ThinkingJoules/linked-data-thoughts/wiki/'Tokenless'-Proof-of-Stake-Blockchain-(PoSW-=-Proof-of-Staked-Work)) Layer0 Platform to enable the building of addressable semantic symbols using subchains.
* First L1 should be a PKI system that will allow many keys to be associated with a Long Lived Identifier (LLI), your Identity.
  * Other L1's (or non-chain apps) can integrate the LLI into their operation for an understanding of authorship.
* Non-blockchain systems can be built that use the LLI to create a resolvable IP address to user's server(s); Example: [Fireflies](http://www.cs.cornell.edu/home/rvr/papers/Fireflies.pdf)
* One or more L1's can be built to generate a semantic ontology to define common data types, understanding of properties, units, conversions between units, encodings, etc. This will act as the data model for databases (Semantic Object Database - SOD) that are namespaced behind the LLI.
* Leverage the LLI-> IP resolver to build a system to retrieve/cache/replicate SOD data between Identities.
  * Goal is to avoid a DHT so users running their own system can cache/replicate exactly the data they want from others.
  * Hashing and encryption will be used for deduplication and replication privacy.

The user of this system should never have to acquire any tokens*. In theory, if a user isn't creating any new global namespace symbols, they would only need to expend some Proof of Work to create an identity or update their keys on the PKI chain.

(* No tokens, but if they want to host their own SOD they might have to pay for hosting/internet/etc in fiat money, self hosting is of course always an option)

# My current vision
The problem with blockchains, is that everything on a single chain must be communicated to every participant. Avalanche and Polkadot solve this problem with creating 'subnets' and 'parachains' respectively. The main idea is to not encumber validators in state they don't care about. If someone wants a micro transaction chain with thousands of TPS they can define their own chain with proper capacity to keep fees low and their system working as designed, and they can do that without being constrained to primitives in a generic one-size-fits-all VM. Monolithic designs like Ethereum are clearly struggling with that design pattern. Seperation of concerns is important. Once you have multiple chains, why not a chain for (most) everything?

## Chain of Chains
If all of these [PoSW](https://github.com/ThinkingJoules/linked-data-thoughts/wiki/'Tokenless'-Proof-of-Stake-Blockchain-(PoSW-=-Proof-of-Staked-Work)) systems were registered on some top level PoSW chain (sort of like the P-Chain in Avalanche) then that would let everyone know what chains exist and other info about them. A concrete example of an extremely simple subchain; you could have each block in the chain simply be a single transaction (given the chain is low velocity) and the payload simply be a JSON object that follows the schema rules for that chain (VM is really just a schema validator). Each block, in essence becomes just a row in a table. Obviously the VM can be as complex or simple as you want it. I'm just trying to illustrate how simple a subchain as database could be. 

Would need to think about all the various ways a chain might be configured and what requirements the registration chain might have. (Side thought: In theory, there might be a use case where a particular subchain VM might have it's own form of registration for a sub-subchain. I suppose if it made sense for their use case, it is perfectly fine to do that.)

### Namespace and addressability
If there is a single registration point, all subnets can be in relation to them. They could have their own sort of URL convention: 

```dr://[Registration TxnID for Subnet]/[subnet specific syntax]```

I would put the URL path in order of high level to low level (opposite of a DNS URL that has the TLD at the end). The subchain could be thought of as a TLD, but for prefix reasons, I would probably move it's position for this URL usecase. I think the ```dr``` scheme would handle a default way to negotiate transport protocols to allow http, websockets, quic, etc.

Instead of TxnID (hash) could we use two unsigned integers? Like (BlockNumber,TxnNumberInBlock)? It would be way better than hashes for compactness. As long as we can know the transaction has reached a state of irreversibility, I don't see why this wouldn't work. Need to explore failure modes, but two uints would be slick. Could perhaps allow these chains to be configured so a block=txn, so it is a single uint. I think that would be fine for chains where you don't want state growth. High throughput chains would still want to batch things. This is something that could be put in the chain registration block, so things know how to address locations. If we can make these types of chains self-similar we can reuse tooling and encodings and make things really simple to develop and understand.

## Distributed PKI
My personal goal is building this aspect of the Digital Reality. I would really like a static identifier that remains unchanged between key rotations. Allowing more than one key per logical identity could also allow ranking of keys to allow devices to sign stuff for other applications, for example, but not be able to add more keys to the identity.

Creating a long lived identifier (LLI) could allow any application to key information to a specific user, and allow cross-ecosystem verification of authenticity of data.

# Semantic/Linked Data
To read about my (lengthy) thoughts and musings on data and how to make it useful, read: [Semantic Objects](https://github.com/ThinkingJoules/linked-data-thoughts/blob/main/semantic-objects.md) which will discuss all things about the Semantic Object Database (SOD)

