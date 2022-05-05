# Unsiloing Data
Exploration of ideas on how to merge concetps from [GunDB](https://github.com/amark/gun), [Atomic Data](https://docs.atomicdata.dev/atomic-data-overview.html), [IPFS](https://github.com/ipfs/ipfs), and [blockchain(s)](https://github.com/ThinkingJoules/linked-data-thoughts/wiki/Blockchains) into a single system. The goal is to allow for addressable, retrievable, and interoperable data that enables applications to be reduced to simply a UI for interacting with and creating the data.

Working name for this concept (not a specific implementation, and I'm open to suggestions on better names): Digital Reality (DR)

This is WIP. Open issues to start discussions!

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

# Layers
I feel there are a few core aspects to build this system.
1) The low level stuff. This is the blockchain(s), and core semantic symbols. [Read more here](https://github.com/ThinkingJoules/distributed-web-data/blob/main/identifiers.md)
2) The mid level stuff. This is building up semantic symbols in some sort of extensible and easier-than-RDF system to make the SOD. [Read more here](https://github.com/ThinkingJoules/distributed-web-data/blob/main/semantic-objects.md)
3) [Using 1 and 2 together](https://github.com/ThinkingJoules/distributed-web-data/blob/main/all-at-once.md)
4) Queries (need to figure out #2 first so we know what the 'triple' looks like)
5) UI Builders/components. Since Apps should now be UI's, how can we make building UI's easier/simpler.

