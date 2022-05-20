# Unsiloing Data
Exploration of ideas on how to merge concepts from [GunDB](https://github.com/amark/gun), [Atomic Data](https://docs.atomicdata.dev/atomic-data-overview.html), [IPFS](https://github.com/ipfs/ipfs), and [blockchain(s)](https://github.com/ThinkingJoules/linked-data-thoughts/wiki/Blockchains) into a single system. 

## Goal
Create a system that allows people to:
- Own their own data (portability) 
- Have all the data in a unified data model (interoperability)
- Create digital identities to help make the data addressable, retrievable, and verifiable between users

This should enable unsiloing of data and allow applications to be reduced to only a UI.

## System Requirements
Below are some requirements ***I believe*** need to be upheld in order to get widespread adoption. These are philosophical/ideological in nature, so if you disagree on some points, then the rest of the ideas in this document will probably not land well with you. These could also be called [Assumptions or Premises](https://github.com/ThinkingJoules/distributed-web-data/blob/main/accessible.md):
- Federated: The core difference between my approach and some other dApp approaches, is I prefer to gain privacy through a federated model, instead of *requiring* encryption (DHT/widely replicated). This requires a way to set permissions on who can access what. 
  - I think content addressing and encryption can and probably will be used, but first and foremost, data sits at end points (like an IP Address).
- Accessible: No acquiring tokens in order to use the system. Ideally little to no money should be needed.
- Permissionless: This system does not aggregate any content, so there is no reason to censor or curate content creation at this low level. Systems built on top of this have to deal with it. Adding new semantic symbols also needs to be available to anyone.
- Discoverable: Underlying core primitives are easily found. These core primitives then enable universal queries for p2p discovery of content. 
- Verifiable: Digital Signatures to verify authorship.

# TL;DR
I will try to lay out a proposal that meets the requirements listed above better than the current alternatives, by combining some underlying components together:
- Shared semantic symbols for interoperable data
- Query language built from core semantic symbols
- Digital Identity (PKI system) used as a namespace to store user data, and build a DNS-like endpoint resolver
## Quick Compare To Other Systems
- The data model is based on a lineage of projects RDF -> Solid -> Atomic Data
  - The data model (like the ones listed above) is conceptually based on a 'Triple'
  - A Triple is the result of decomposing tables. A single Triple is basically a 'cell'.
- You could probably fit the data model into GunDB, but GunDB isn't 'easy' to set up in a federated manner.
- IPFS is content addressed. We need static 'keys' in which to compose objects and with which to associate permissions. The 'triple' has three parts, Subject-Property-Value. The 'Value' portion might be used within a content addressed network.
- Blockchains, of some sort, seem unavoidable to get truly non-repudiable public statements. I expect almost all data for social-like apps will have to go on some sort of chain. The challenge is how to do this in a manner that can handle social media (internet) scale. All private data is not on any 'chain'. Any blockchains within this new system would be of the [tokenless design](https://github.com/ThinkingJoules/distributed-web-data/blob/main/tokenless-blockchain.md).

The data model is the unifying mechanism. Regardless of how or where the data is authored, it should always be thought of as Triples. This allows data authored on a blockchain to share symbols with non-blockchain data. This unification allows a single query language across everything. This single query language becomes the universal API in which apps (now simply UIs) operate.

# Quick Summary (11 minute read)
There has always been a large amount of interest in taking back control of our digital lives; this is only going to grow. I do not think I am necessarily the right person to solve this problem, but I have been searching for and reading many discussions on how to 're-decentralize the web'. So I wanted to try to reconcile some common ideas and approaches and try my hand at positing a complete system. This is a quick run through of my why and how. 

[Find a detailed explanation of the architecture here](https://github.com/ThinkingJoules/distributed-web-data/blob/main/architecture.md).

## Defining the Problem
Current (centralized) apps are authoritative, in that they control your data, and the questions you can ask about it (API). If all data becomes unsiloed (user-owned/portable), should apps still be authoritative in the symbols and semantics they use to describe the data relevant to their app? If they are not authoritative, how will developers have any way to understand data created and labeled by other developers through different 'apps' (UIs)?

I'm going to make the assumption that readers of this document do not want decentralized apps (dApps) to be authoritative in any way (else what are we doing?). But without any authority — chaos. (If you can't find something you own, even though you **know** you own it, do you actually have it at all?). 

## Recent Attempts: No Central Authority
So, the thinking goes, if involving a central authority is a non-starter, then we need to take the no-authority approach and decrease the chaos. To decrease chaos, we need a data model that is simultaneously as simple and as expressive as possible. Let's look at some examples and their underlying data models:

- GunDB is a graph-like data model, where anyone can make any sort of chaining of symbols that terminates in a value. There is no built-in way for other developers to easily understand another app's data (graph structure) without extensive documentation. It also does not have any sort of query language to ask questions about your data. You either know what you are looking for and can find it, or you have no easy way to 'learn' about the data. In addition, you cannot easily have a UI load your data from **your** server. Data is highly replicated and the only privacy you get is through encryption. The data is often persisted with where the UI is loaded from, and is more ephemeral in the rest of the system. To 'own' your own data the UI builder must enable your browser to store your data. The upside from the highly replicated method is the use of CRDTs to merge mutable data at great efficiency. GunDB is often used as a web socket synchronization mechanism that can operate at graphics capable performance.
- IPFS has a Hash-Based File-Like data model. It uses Content IDentifiers (CIDs) to give hints to allow the creation of hash-based graphs and data structures. This is helpful as the system is pseudo-self documenting. However, because it is strictly content addressed, it is difficult to work with when updating data while maintaining widely disseminated identifying symbols. It provides mechanisms to work around this (IPNS) but at its core, it was built around immutable hashes. A lot of things were added on to make IPFS more user friendly, but I feel, at its core, it is built around an incorrect primitive for our goal. I know content addressing is needed in any decentralized system, but I think we need to use it in a different place. IPFS has become complicated in order to work around the purely hash-based addressing. I think you could configure the system to do whatever you might want, but that ability comes at the expense of chaos when trying to integrate different apps' data structures.
  - The focus on the content addressed primitives has its upsides. IPFS is the only system listed here that does not rely directly on DNS to properly retrieve data. It uses a DHT to handle the routing algorithmically. That is why their identifiers are not, and cannot, be (directly) human readable.
- Atomic Data uses a graph-based model using an RDF-like 'Triple' as its core primitive. Everything is a triple. Through reification it gains a solid model for learning about the data and what the labels mean (semantics). Even as a nascent project, I feel it is on the best footing with regards to an easy to understand data model, that also allows great expressivity. The reason I think a 'triple' is the correct model, is that the Property symbol has great semantic value. In order to leverage this semantic value, we need to focus on finding ways to make Property Definitions widely reusable, understandable, findable, etc. So far, making Property Definitions reusable and understandable has been accomplished, but making them findable/searchable (as well as widely replicated) remains a key shortcoming. The integration of IPFS (which has been talked about) to make property definitions immutable would be helpful, but there would still need to be a mechanism built in either IPFS or the project itself to find 'updated' versions of properties. It uses public key technology to verify authorship of data. However, that is really the only distributed system technology built into it at the moment. Currently all of your data lives on a single server, which you control. There is currently no synchronization mechanism for multiple servers for your own data. (Again this project is quite young and is already actively working on most of the problems listed.)

Out of the projects listed above, we get the cool properties of mergeable data, content addressing, and semantic symbols. But the thing they all have in common is that each, in some way (and some more than others), so far lacks mechanisms for giving developers a prevailing, shared, vision of how to use the system to build interoperable data/apps. Without giving developers a clear, shared vision, we will forever be stuck with authoritative apps and data structures. We will then have only moved siloed data from separate servers to 'owned' data that is still logically siloed.

In trying to solve the problem of central authoritative apps, it feels like we created the same problem in a different form (while still having varying success on managing the chaos). So what to do?

Perhaps we need to think of authority differently. 

## Authority Reconsidered
Let's take a look at blockchains and how they fit into this puzzle. 

The data model of a blockchain is simply an append-only log database. It can be much more complex than a database, but in short, it is a state machine. What is contained within the log (state transitions) is dictated by the use-case of the chain. Almost all blockchains today focus around exchanging of tokens (e.g. Bitcoin) and/or allowing complex and custom rules that govern state transitions (e.g. Ethereum, smart contracting). 

So basically, this is an ultra authoritative system. However, in most configurations it is considered permissionless. The system is authoritative, but there is no single agent/entity that has "Authority", as in, special powers of control *within* the system.

I feel this meets the requirement for a non-authoritative system. 

So let's list reasons often cited for **not** using blockchains:
- Tokens are both high friction to acquire and enable scams.
- Generally not considered "scalable".
- Can't delete old data. (Sensitive information cannot be 'on-chain'.)

So we can't, and don't want to, simply put all the data on a blockchain. This is a terrible idea. Does this mean we cannot use blockchains at all? Let's look at the above critiques and see what they are really criticizing:
- Tokens are usually used to incentivize securing the blockchain. But we only need this security to secure...the tokens. Tokens can only have an exchange rate, if you can actually exchange them, on-chain. If you don't have tokens (or tokens that can't be traded...which aren't really tokens then, are they?) then you have no extractable value (nothing to secure). Scams become impossible and we have removed an onboarding step for participants (i.e. acquiring the token).
  - If you remove the ability to trade tokens from the blockchain construction, is it still a blockchain? Without going into detail here, you could call it a Blockchain (-Token) or an Append-Only Log (+Permissionless +Txn Fees +Consensus +Sybil Resistant...etc.). 'Tokenless Blockchain' still seems the most apt descriptor.
- "Not scalable": This complaint originated in the context of internet-scale payment networks (token-based blockchains). If we continue to consider a tokenless construction, then how much throughput we need is based on the usecase of any particular chain.
  - Additionally, is it possible that getting rid of the token allows more flexibility in scaling blockchains?
- "Can't delete old data/Sensitive info can't be on-chain": This is true. But if we are not trading tokens, and are instead creating semantic symbols...would we ever want old data deleted?

If we could temporarily buy the idea that many aspects of blockchain don't have to be a detriment, and other aspects could be beneficial, then we could perhaps consider the usage of them as *part* of a new system. What if we put *some* data on a blockchain, without a token? And then combine that blockchain(s) with parts of the other systems listed above?

## Frankenstein System
### Data
If we think the 'triple' is a powerful primitive, let's see what happens if we put Atomic Data's Property Definitions on a blockchain:

- Property Definitions should not contain sensitive information, so they could be put on a blockchain.
- We gain a central place in which to enforce a known format for wide consumption and understanding.
- Through centralized (but permissionless) creation, we know exactly how and where to find all known definitions. This allows easy discoverability and reuse, no web-crawling needed.
- All participants of the Property Definition blockchain would replicate the state, to ensure that definitions are always reachable (online).
- We could point backwards to updates on old symbols to create a clear lineage of changes. Reading through the chain would yield all symbols, as well as related/updated symbols.

Scalability is not a problem. For example: let's say a language has ~1,000,000 words (average adult knows up to 35,000 words in their native language). If each of these need to be used as a 'property' then even with the worst performing blockchain (Bitcoin) with a throughput of about 4 transactions per second, you could put all of those on chain in about 70 hours. Even with combinatorics and nuance, the slowest of all the blockchains would still have enough throughput to add definitions faster than people could use them. 

For our ends, does that mean blockchains are *too* scalable, because they are permissionless?? I would argue that we would absolutely **need** to slow things down immensely, at least for the Property Definition use case. We cannot simply detect and reject semantically equivalent properties, so we would still need a mechanism to nudge people to search for and attempt to understand/reuse existing definitions. Generally blockchains have transaction fees (denominated by the token) to prevent excessive state-growth to the chain. A tokenless transaction fee could be done through a fair proof of work algorithm. We could create a pseudo voting system where new transactions expire if they don't meet enough PoW threshold by a certain time. This could allow many developers to observe, read, and discuss new symbols and add work (vote) in order to get them accepted into the canonical list of all properties. Symbols could still be added unilaterally but would require a single person to run enough computers (expense) to 'force' the transaction through alone. This is still permissionless, but here we are *purposefully adding friction* to help curate the list of symbols.

Conversely, in a usecase requiring non-repudiable data (e.g. social-like apps), we need more throughput than any blockchain system in existence currently offers. Are these usecases hopeless in a distributed web environment? Not necessarily: all blockchains in existence are built around the token. I think removing the token from a blockchain can make it much easier to shard, and gain horizontal scaling to meet higher throughputs. We don't need the double-spend property that token-based chains require. But we would still have the guarantee of a single history provided by a diverse set of actors recording snapshots of data.

So assuming even the hardest usecase might be possible to handle, let's go back to the Property Definitions. If we now have addressable semantic symbols from this Properties Definition chain, then we could use these symbols in other, **non-blockchain** systems. Now two different developers have a known, common 'dictionary' of properties to aid in searching for data or integrating data to a synthetic UI. These symbols could in fact be used to augment existing dApps, such as GunDB, to improve interoperability/composability.

(Note: There are obviously many nuanced issues that still need to be solved, but I think the pattern above could be re-used to add other primitive symbols that the entire system relies on. Thus creating a self-similar developer experience. Read the [detailed architecture write up](https://github.com/ThinkingJoules/distributed-web-data/blob/main/architecture.md) if you want to dive into the deep end.)


### User Namespace: A place for data to reside
Given that we want the ability to fully control our data, we need a way to make sure systems know which data is ours and where to find it. 

The original web is built around DNS and Domain Names. The domain name acts as a namespace prefix as well as a static identifier to find the current IP Address (where the data for that Domain resides). This is a permissioned and centralized system that has worked well: Human Friendly and Unique (there is only **one** google.com). The problem is that requiring a user to purchase a domain name seems no different than needing a user to purchase a token to use a blockchain. (The combination of human friendly and unique, is exactly what characterizes the digital tokens called NFTs.) 

(Hypothetically IPFS is an alternative to DNS, however, we do not consider IPFS at this level, because the content does not have a location OR a namespace, it is purely a hash. An IPFS-like approach should be used lower in this system. We are using a federated system at the high level.)

To create a low friction and accessible User Namespace (Domain), I propose that it be tied to the one known mechanism that gives users the ability to verify themselves digitally: Public Keys.

Both GunDB and Atomic Data use a single key-pair to prove authorship of data. I think it is essential to both allow more than one key, but also to keep some sort of Long Lived Identifier (LLI) that remains constant across key changes. To do this we will need a special ([tokenless](https://github.com/ThinkingJoules/distributed-web-data/blob/main/tokenless-blockchain.md)) blockchain that deals with all the PKI key rotations. The LLI would be derived from the transaction that creates a new identity. This will create a unique, but not-human-friendly symbol. (I think using a blockchain here is inevitable. Even the BlueSky ADX proposal uses a [permissioned append only log for their PKI](https://github.com/bluesky-social/adx/blob/main/architecture.md#the-did-consortium).)

Leveraging this new primitive (LLI) we can now create a system that can act as a way to resolve IP addresses (or equivalent). We can extend this system to allow Human-Friendly-Alias(es). However, to avoid making things token-like, we must allow collisions of names. This has the potential for confusion, but I think there are ways to help minimize that.

(Again, read the [detailed architecture write up](https://github.com/ThinkingJoules/distributed-web-data/blob/main/architecture.md) if you want more info.)

## Bringing it all together
We use tokenless blockchains to generate both LLIs (user address/Domains) and core semantic symbols. We make these tokenless blockchains addressable through [chain coordinates](https://github.com/ThinkingJoules/distributed-web-data/issues/2). Putting these bits together we can make a new non-DNS resolved URL. These URLs are used in a Triple-like data model, as they were in RDF/AD. 

We gain immutability of the core symbols through the append-only nature of blockchains. We gain addressability of these symbols through chain coordinates. We gain discoverability due to the centralized (but permissionless) nature of using a blockchain.

Leveraging the discoverability of the core symbols, we should gain maximum reuse to ensure a widely understood set of symbols. These core symbols can then be integrated into a query language that will act as a universal API for all 'apps' (UIs). We have now gained interoperability 🎉. 

(Important aside: The query language can be flexible and there is no worry of lock-in, because the underlying symbols are in a shared, yet addressable format, independent of anything else. I think the URLs will be http resolved, so we could simply use GET requests similar to how AD operates currently. I think a more complex option will be needed as well, but I expect several simultaneous methods to ask about data to be available at once.)

We can further extend this system to attempt to solve the non-repudiable data problem. I personally feel that blockchains are required here as well. I go into my reasoning in great length in the full architecture writeup (linked below). I believe this is the hardest part of the system, and would love to have a discussion exploring ways of solving it that differ from my own. 

(Of course if you see anywhere my reasoning is faulty, please open an issue to discuss! I know I'm not an expert and welcome feedback!)

[Find the full Architecture outline here](https://github.com/ThinkingJoules/distributed-web-data/blob/main/architecture.md).