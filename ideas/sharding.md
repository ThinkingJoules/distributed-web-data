# Context
I think there are some aspects of this system that require the use of a blockchain. The two core aspects are the digital identity PKI system, and the timestamping of merkle roots to ensure non-repudiation of public statements.

With regards to blockchains, I see two large obstacles:

First, my largest concern is the friction required for a user to acquire a token in order to use the system. I think this friction would destroy any hope of widespread adoption. At a minimum I feel that transaction fees **must** be Proof of Work. That gives us a mechanism for spam protection.

Second, we need to scale far beyond the capabilities of a single chain. If we can remove a token from this completely, I *believe* we have more sharding options, as we don't have the double spend constraint.

This document is a naive first guess on how to do sharding, given the avoidance of an integrated token. I have very little knowledge of conventional (token) blockchain sharding, nor conventional database sharding. The approach I take here is more along the lines of a DHT, but for chains. A 'Distributed Chain Table' or DCT if you will.

# Difference From Token-Blockchains
## PKI
To make the multi-key identity PKI system work, we only need to have a diverse set of actors observing key rotations. We only need to ensure that keys are invalidated by currently active keys. 

In a monolithic blockchain, the interweaving of many parties updating their key sets, sort of build upon each other and prevents a single user from rolling back the chain. 

For sharding, this is not ideal. We have no way to scale the shards up and down, as the users are cryptographically intertwined in the monolithic chain.

I *believe* that we want to have users each have their own signature chain. That way, we can shuffle a users PKI history between shards as the network grows and shrinks. This also allows any observer to obtain a single users chain, without having to download or replay interweaved state transitions from other users.

## Non-Repudiable Data (NRD)
This setting requires immense scale. The current idea, is to simply have a widely viewed publication of a merkle root. It is the responsibility of the author to ensure all past states are persisted. 

(Technically, this is not *truly* non-repudiable data. If an author were to lose their past states, those particular statements would become unknown. While not ideal, I think this is okay. The author would *look* like they are trying to hide something. So it sort of falls into a reputation metric. Someone who can't prove many old roots, should be thought of as untrustworthy. I think this is the best we can do, else our NRD scaling problem becomes even more extreme: record *every* NRD statement, on-chain. This would make shard scaling very expensive. Each user's chain would be a huge amount of state to move during shard transitions, and the number of users on a shard would be greatly diminished. It also has no flexibility when it comes to graceful degradation. When publishing a root, the user can make N updates in a **single** txn. Recording every statement *requires* N **txns**. With the merkle root method, a user can simply space out the frequency of the merkle root submissions as the network becomes congested. Recording every state degrades the latency on the *creation of* the data, whereas the merkle root method only degrades the latency in which created data becomes *provable* (or 'Published').)

This would also be within a user namespace (someone has to author this...). Like the identity/PKI problem, I think we need to keep a user's publication chain, self contained.

# Similarities Between PKI & NRD
It looks like we keep things chunked by user. (NRD is technically namespaced *by the PKI* system, so the PKI system is a dependency for our NRD system.) The reason given above for keeping chains self-contained, is that way we can 'move' these chains between shards. This is necessary, because as the system expands (the set of diverse 'validators' increases) we need to split existing shards, and determine which chains the validators are validating. This random (but deterministic) assignment fulfills the 'diverse set of actors' requirement. A validator cannot pick *which* users they are validating. This is necessary to stop any potential collusion.

This is conceptually similar to a DHT. In a DHT, a node keeps the hashes closest to it's NodeID. For us, the node would be keeping *chains* with user namespace hashes closest to it's NodeID. Unlike a DHT, when data is added by a particular user it must be collectively validated by *all nodes keeping that chain*.

# Problem?: Global State
## Validators
We still need a top level (singular) chain, so *all* validators know who else is validating (to derive *all* members of their shard). This should be easily handled by a single chain. The validators will *not* be everyone on the internet, and we can force a minimum validation period, thus reducing the number of 'Add Validator' txns on the 'Validator Registration Chain' (VRC).
## Users
This is the hard part. All validators must know about *all* users, so they know which chains are part of their shard. I think this means we need a top level (singular) chain for the *creation* of an identity. If *all* validators validated this 'User Registration Chain' (URC) then they would know the complete set of all Validators *and* the complete set of all Users. The URC differs from the VRC, in that the VRC must record one txn per validator *per* validation period. The URC is simply a log of only 'new' identities. The key rotations would happen within the shards. (Think of the txn on the URC as simply a special 'Genesis Block' that might have a different structure than the following 'update' blocks.)

Can a single chain have enough throughput to handle user registration? According to this [chart](https://www.statista.com/forecasts/1146844/internet-users-in-the-world) we had about 1.8 billion internet users in 2010, and today we have about 5 billion. That is (an average) rate of:
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
By removing a token, we have created a frictionless way for users to create new identities or publish roots. However, if we are pushing the limit of blockchains, then surely the hardware to run all this will be considerable? We could require the validators to use a token but keep transaction fees tokenless, but this removes any mechanism for validators to make money. Adding a token is only useful if everyone uses it, to allow the transfer of value *from* consumers of the network *to* the people running the network.
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

Naively, I think of the shards as a bit like leaves in a B Tree. The idea is that you can tune when to merge or split the leaves (shards).

We need to scale shards up or down, but we also want to do this seamlessly while the network is operating. We also need to make sure that attackers cannot 'pick' their shard.s

## Avoiding Shard Selection
To ensure that an attacker cannot gain a controlling share of the nodes on any given shard, we need to randomly group validators together. The only thing they have that is constant is a NodeID (pub key, or hash of pub key). An attacker can expend effort to get a particular pub key that might be used to ensure assignment to a particular shard. So we can't use this as a source of entropy directly.
### Non-Selectable Entropy
One thing that is not easy to pick, is the entropy contained within any txn performed on a blockchain: you can't know *which* block, or know where, *within* the block, your txn might reside. You can use this uncertainty (along with the block header) to derive an unpredictable hash. This hash would then become an alias for the NodeID. 

Ah, but yes, our attackers are clever. They could simply create a bunch of new validators and some subset of them will get assigned to a common shard, that they can then execute their plan. Sybil resistance is hard enough, but as soon as we start dividing up our validators, we are also dividing up our *security*. Each shard is susceptible to a *targeted* attack that is unfeasible on the larger system. This is the fundamental issue in DHT-like deterministic sharding paradigms.

What can we do? Not much. Our solution needs to be deterministic in order to derive shards on the fly. The simplest solution, is increase our sybil resistance metric that determines the minimal threshold to become a 'Validator'. If set high enough, an attacker would not be able to expend enough to attack. The downside, is that now honest actors need to 'pay' more to contribute to the system. To prove that they are *not* bad actors.

Let us assume, given a high enough 'entry fee' to become a validator we can mitigate the attack to a degree that it is highly unlikely for a single actor to overcome a shard.

Given that assumption, we only need to shuffle our many-many relationship *when the network grows or shrinks*. If we completely reshuffled the shard validators and chains within a shard then an attacker could only attack a shard for a limited time, reducing the effect of beating the sybil resistance even further. Basically, as soon as the number of shards in the system changes, their nodes will be randomly distributed again. 

If the worst an attacker can do, is stall the shard, then they probably are trying to disrupt someones *particular* chain on that shard.

Let's think about the nature of the shards and how we migrate from one network state to the next.

## Shard Migration
When the number of shards changes, how should we adjust things? Using a B Tree idea, we would just join adjacent shards, or split a shard in to two. We want to cause some amount of change, but at the end of the day, moving state between nodes is expensive and ever-growing (given the append-only nature). This implies that, although we *can* gain throughput by sharding, this throughput results in *more state*. The more state per chain, the more we do *not* want to reshuffle the shard validators and chains within a shard.

So, we end up in a scenario where we don't/can't shuffle. This makes our system fall back to *only* our sybil resistance threshold to become a validator.

So how do we determine which validators are on which shard, and which chains are on each shard?

What do we have to work with? We can derive a NodeID alias from the first time we see an 'Add Validator' txn on the VRC. This will give us a static hash. The identities could be handled in a similar manner, a hash based on the position on the URC. We must always assign a chain to a set of validators, but validators themselves are only valid for their 'validation' period, and so, come and go. Chains can simply become inactive.

Since we can know all validators validation periods (registered on the VRC), the entire network can be watching for some sort of 'running' average for the total number of validators. If this average falls below a certain threshold for too long, it will lower the number of shards, and if it is above for so long, it will increase the number of shards. I think this could be modelled like something akin to a [PID Controller](https://en.wikipedia.org/wiki/PID_controller). Another similar method is used in Bitcoin for its [difficulty adjustment](https://en.bitcoin.it/wiki/Difficulty) to keep block times close to 10 minutes as the amount of mining increases. In Bitcoin, these are called 'Difficulty Epochs'. In our system we could call our adjustment sample 'Throughput Epochs'. Since the number of shards is how the system changes throughput.

Naively, we could go the DHT route, and say that nodes that have an alias close to a chains alias, would validate those. This gives us, effectively, a double random pairing. This close-ness matching should always give validators preference based on the static nature of both aliases. I don't think users would be advantaged to pick a shard, so the txn fee is adequate to prevent any sort of choosing on the user side. The user side is tricky, as we want to minimize the time/effort to make honest user onboarding a better experience. However, this could allow a bad actor to attempt to get a user alias close to some shard they are validating. I don't know this is much of a worry, as the validators are still the expensive part. I would rather ratchet up the validator cost even higher, if it could make user creation cheaper/faster.

Since the shards have an aggregate throughput of, effectively, a single consensus system, then we must assume that 'usage' of each users chain is also randomly distributed. Some people are power users, and most are not, so it is unlikely that *all* the power users end up on a single shard. This does indicate that the shards *could* be very out of balance. I don't know that we can control for that.

We would want to update often enough to make the system responsive, but not too often, as any state transfers caused by shard migrations are wasted effort (overhead). If we made the validation period fixed at, say, 14 days. Then we would probably want our Throughput Epoch to adjust about once every 30 days (>2x validation period). Because we are not shuffling, there should be no concern of losing data during splits and merges, as the data will be replicated on all nodes on a particular shard at any given point.

# Next Steps
I think it is hard to say if a tokenless model will work until we can reason about the actual cost for an individual to contribute to network throughput. In order to contribute they *must* run two monolithic chains (VRC+URC) but at least one shard (either PKI updates, or NRD root timestamping)

## Needed Metrics
What is the overhead to run our chosen consensus algorithm at various throughput levels (8-30tps)? We would need to know (approximate):
- Bandwidth Speed
- CPU Usage
- RAM
- Min Read/Write Speed for SSD
- SSD size/growth rate

With this info, we can determine how much overhead is required for someone to contribute to adding throughput (sharding). If these stats are not outside many of the common/affordable cloud provider nodes then we might be able to do this in a tokenless manner. If validating *just* the URC is substantial then we need to think more about how to design this system.

From the user perspective, we also need to understand how txn fees work. If this system is tokenless, then txn fees will probably be PoW. How long of a delay (doing enough work) might a user experience in creating a new identity (on the URC) given various amounts of demand (throughput) on the network?

## Current Design
### Consensus Algorithm
I think we *must* use the [Snow Consensus](https://assets.website-files.com/5d80307810123f5ffbb34d6e/6009805681b416f34dcae012_Avalanche%20Consensus%20Whitepaper.pdf) algorithm, as it can allow for (in theory) an unbounded number of validators. This is required, because if all of the shard validators also validate the VRC & URC chains, then we could potentially have hundreds of thousands or potentially a million or more validators (since we then divide them in to shards). Many consensus algorithms have message complexities too high, and have a sharp upper bound on max number of validators.
### Sharding
We will use the DHT-like approach and assume that randomly derived, yet static identifying aliases are sufficient. We would need to try various methods for determining when a split or merge would occur. I expect the number of validators per shard to be 100-300 to ensure some amount of security. PKI might be on the higher end (since it is foundational to *many* systems), whereas the NRD roots could get by with something closer to the lower end.

## Approach
First, I think we need to model the messages that are required to achieve consensus on a transaction using Snow(Ball). The paper is pretty detailed in how the sub-sampling is handled, so if we figure out what that looks like in practice, then we can estimate some of our Needed Metrics. We may need to code some subroutines in order to get some better estimates on performance.



