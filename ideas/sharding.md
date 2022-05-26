# Context
(This document is still a work in progress.)
I think there are some aspects of this system that require the use of a blockchain. The two core aspects are the digital identity PKI system, and the timestamping of merkle roots to ensure non-repudiation of public statements.

With regards to blockchains, I see two large obstacles:

First, my largest concern is the friction required to acquire a token in order to use the system. I think this friction would destroy any hope of widespread adoption. At a minimum I feel that transaction fees **must** be Proof of Work. That allows gives us a mechanism for spam protection.

Second, we need to scale far beyond the capabilities of a single chain. If we can remove a token from this completely, I *believe* we have more sharding options, as we don't have the double spend constraint.

# Difference From Token-Blockchains
## PKI
To make the multi-key identity PKI system work, we only need to have a diverse set of actors observing key rotations. We only need to ensure that keys are invalidated by currently active keys. 

In a monolithic blockchain, the interweaving of many parties updating their key sets, sort of build upon each other and prevents a single user from rolling back the chain. 

For sharding, this is not ideal. We have no way to scale the shards up and down, as the users are cryptographically intertwined in the monolithic chain.

I *believe* that we want to have users each have their own signature chain. That way, we can shuffle a users PKI history between shards as the network grows and shrinks. 

## Non-Repudiable Data (NRD)
This setting requires immense scale. The current idea, is to simply have a widely viewed publication of a merkle root. It is the responsibility of the author to ensure all past states are persisted. 

(Technically, this is not *truly* repudiable data. If an author were to lose their past states, those particular statements would become unknown. This is okay, because this author would *look* like they are trying to hide something. So it sort of falls into a reputation metric. Someone who can't prove many old roots, should be thought of as untrustworthy. I think this is the best we can do, else our NRD scaling problem becomes even more extreme: record *every* NRD statement, on-chain. This would make shard scaling very expensive. Each user's chain would be a huge amount of state to move during shard transitions, and the number of users on a shard would be greatly diminished. It also has no flexibility when it comes to graceful degradation. When publishing a root, the user can make N updates in a **single** txn. Recording every statement *requires* N **txns**. With the merkle root method, a user can simply space out the frequency of the merkle root submissions. Recording every state degrades the latency on the *creation of* the data, whereas the merkle root method only degrades the latency in which created data becomes *provable* (or 'Published').)

This would also be within a user namespace (some has to author this...). Like the identity problem, I think we need to keep a user's publication chain, self contained.

# Similarities Between PKI & NRD
It looks like we keep things chunked by user. (NRD is technically namespaced *by the PKI* system, so the PKI system is a dependency for our NRD system.) The reason given above for keeping chains self-contained, is that way we can 'move' these chains between shards. This is necessary, because as the system expands (the set of diverse 'validators' increases) we need to split existing shards, and determine which chains the validators are validating. This random (but deterministic) assignment fulfills the 'diverse set of actors' requirement. A validator cannot pick *which* users they are validating. This is necessary to stop any potential collusion.

This is conceptually similar to a DHT. In a DHT, a node keeps the hashes closest to it's NodeID. For us, the node would be keeping *chains* with user namespace hashes closest to it's NodeID. Unlike a DHT, when data is added by a particular user it must be collectively validated by *all nodes keeping that chain*.

Note: The hand-off of the chain data needs to be handled carefully, to ensure the new shard users have the data before the old shard members remove it to make room on-disk. This means that at any point a shard must be able to hold two shards worth of state. However, chains will be non-equal and ever growing in size. So this will need some thought... More on this later.

# Problem?: Global State
## Validators
We still need a top level (singular) chain, so *all* validators know who else is validating (to derive *all* members of their shard). This should be easily handled by a single chain. The validators will *not* be everyone on the internet, and we can force a minimum validation period, thus we reducing the number of 'Add Validator' txns on the 'Validator Registration Chain' (VRC).
## Users
This is the hard part. All validators must know about all users, so they know *who* to validate. I think this means we need a *single* chain for the *creation* of an identity. If *all* validators validated this 'User Registration Chain' (URC) then they would know the complete set of all Validators and the complete set of all Users. The URC differs from the VRC, in that the VRC must record one txn per validator per validation period. The URC is simply a log of only 'new' identities. The key rotations would happen within the shards.

Can a single chain have enough throughput to handle user registration? According to this [chart](https://www.statista.com/forecasts/1146844/internet-users-in-the-world) we had about 1.8 billion users in 2010, and today we have about 5 billion users. That is (an average) rate of:
- 5b-1.8b = 3.2b / 12 years = 267 Million per year
- 520m / 365 = 732,000 per day
- 732,000 / 24 / 60 / 60 = ~ 8.5 Users per second.

Do we think users will create identities at the internet adoption rate? If this system contains a single 'killer app' then we should compare it to some app *on* the internet. I will use TikTok as it took the world by storm and is later in release date than many other large social media apps. TikTok was released in September of 2016, so let's just say 2017. We will use todays total app downloads as a proxy for 'sign ups' (2.6 billion at time of writing). This should over-exaggerate new users. We will consider this to have happened over 5 years (instead of 5 years 5 mo at time of writing). All numbers are rounded up.
- 2.6b / 5 years = 267 Million per year
- 520m / 365 = ~1.5 Million per day
- 1.5M / 24 / 60 / 60 = ~ 18 Users per second.

To get a reference, the highest number of transactions in a single day on Ethereum was 1.5 Million. What we are doing is far more simple, and there are newer consensus algorithms that can scale much better. So even with older technology, a blockchain network *can* handle this.

The problem, is that these validators *also* need to validate their shard. However, this would be running a second *parallel* consensus system within the validator's shard. So the throughput here, comes down to the performance *of the validators machine*. This would require a healthy enough internet connection, as well as processing power to run all the code (check signatures), as well as a fast enough (and large enough) disk to write all these states to disk (SSD).

Between consensus algorithms that are much more scalable than Ethereum, and computer hardware continuing (for now) with Moore's Law, the real analysis would probably need to be on the pure bandwidth required. I have no doubt that these validators would need to be ran in the cloud to achieve the necessary throughput.

# Problem: Who pays for this?
By removing a token, we have created a frictionless way for users to create new identities or publish roots. However, if we are pushing the limit of blockchains, then surely the hardware to run all this will be considerable. We could require the validators to use a token but keep transaction fees tokenless, but this removes any mechanism for validators to make money. Adding a token is only useful if everyone uses it, to allow the transfer of value from consumers of the network to the people running the network.
## Need a token?
The reason to not use a token is to remove the friction for initial user on-boarding. In the tokenless form, validators would need to burn PoW (or some other Proof Of [Anti-Sybil]?) to ***generate*** a sybil resistant metric. My original vision for keeping this tokenless, was that most users would already be running a node for themselves, and could also use their node for validating. 

This donation of resources would be at a marginal cost and if paired with scaling (more nodes = more sharding), then if people want the network to be faster, then *more people* share their resources to allow more sharding (and therefore more throughput). There is no coordination mechanism here. People that want a faster network than the current one, *should* donate, and if others don't notice any throughput problem they probably wouldn't donate their resources.

It sounds like donations work if we can keep things in the 'marginal cost' regime. As soon as special purpose, high performance nodes need to be put online *just to validate*, then we can no longer rely on donated resources to ensure a stable system.

### Node Performance
If we parameterize sharding properly, then we could probably spread the validation load out far enough that a users node could be dual-use (marginal donation cost). However, in order to participate in the sharding, a validator *must* be able to handle the throughput of the monolithic User Registration Chain.

Let's think about the nature of the URC. It should be an S-shaped adoption curve. This means that on either end of the S-curve we won't have any problems. We would not expect new user registration to continue forever at the peak growth rate (mid S-curve), even if people are creating multiple identities. 

Because the URC is a single chain, adding more validators adds no throughput benefit. Adding validators only increases the decentralization and security of the chain (more honest actors makes it harder for bad actors to attack). 

We really only need to understand the relationship of: resource requirements vs consensus throughput. Since the PKI and NRD shards are technically independent, then a node *must* run the VRC & URC and then can run one or both of the PKI & NRD validation. Since the shards will be the same consensus as the VRC/URC chains, just ran in parallel, we can just use the resource calculations over again to add up the total resources needed for different throughput combinations of URC+VRC+[Shard(s)].

# How to Shard?
As noted earlier in this document, we need a way to automatically coordinate the scaling up and down on the number of shards.

Naively, I want to combine concepts from two existing computer science ideas: B Trees and Epoch Based Reclamation (EBR).

I like the idea of setting thresholds for the division and merging of the B Tree Leaves. These leaves are like our shards.

EBR is used in non-garbage collected languages as away to reclaim (free) memory.

We need to scale shards up or down, but we also want to do this seamlessly while the network is operating. We also need to make sure that attackers cannot simply generate NodeIDs that can allow them to 'pick' their shard.

To get around the attacker picking the NodeID, we can use the entropy from their 'Add Validator' txn on the VRC to derive an ValidationNodeID. We would then map that to a particular shard for the entirety of their validation period. The issue here, is that they may need to download a considerable amount of state in order to start validating any shard. This is where an attacker costs us significant overhead: we cannot have a static or easily predictable NodeID -> Shard mapping.

Since we can know when their validation ends, the entire network can be watching for some sort of 'running' average for the total number of validators. If this average falls below a certain threshold for too long, it will lower the number of shards, and if it is above for so long, it will increase the number of shards. I think this could be modelled like something akin to a [PID Controller](https://en.wikipedia.org/wiki/PID_controller). Allowing a scale up/down of multiple shards per adjustment period.

If the correction (change to the number of shards) is applied at some delay point in the future, then it can give the nodes time to perform shard migration in the background. Shard migration would basically be syncing up all the state for the nodes new shard (and begin receiving updates) so that when the time comes to 'switch' it can begin instantly validating, thus completing the handoff.

We would want to update often enough to make the system responsive, but not too often, as all the shard migrations are wasted effort (overhead). If we made the validation period fixed at, say, 14 days. Then we would probably want the PID algorithm to loop and adjust about once every 30 days (>2x validation period).

# Next Steps
I think it is hard to say if a tokenless model will work until we can reason about the actual cost for an individual to contribute to network throughput. In order to contribute they *must* run two monolithic chains (VRC+URC) but at least one shard (either PKI updates, or NRD root timestamping)

## Needed Metrics
What is the overhead to run our chosen consensus algorithm at various throughput levels (8-30tps). We would need to know (approximate):
- Bandwidth Speed
- CPU Usage
- RAM
- Min Read/Write Speed for SSD
- SSD size/growth rate

With this info, we can determine how much overhead is required for someone to contribute to adding throughput (sharding). If these stats are not outside many of the common/affordable cloud provider nodes then we might be able to do this in a tokenless manner. If validating *just* the URC is substantial then we need to think more about how to design this system.

From the user perspective, we also need to understand how txn fees work. If they are tokenless, then it is probably going to be PoW. How long of a delay (doing enough work) might a user experience in creating a new identity (on the URC) given various amounts of demand (throughput) on the network?

## Current Design
### Consensus Algorithm
I think we *must* use the [Snow Consensus](https://assets.website-files.com/5d80307810123f5ffbb34d6e/6009805681b416f34dcae012_Avalanche%20Consensus%20Whitepaper.pdf) algorithm, as it can allow for (in theory) an unbounded number of validators. This is required, because if all of the shard validators also validate the VRC & URC chains, then we could potentially have hundreds of thousands or potentially a million or more validators (since we then divide them in to shards). Many consensus algorithms have message complexities too high, and have a sharp upper bound on max number of validators.
### Sharding
We will have a many -> many relationship between validators and chains. We need to think about the number of validators per shard that is acceptable, and also how to programmatically coordinate growing or shrinking the number of shards.



