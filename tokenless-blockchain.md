# Isn't this impossible?
Depends what properties you think separate a blockchain from an append-only database? I feel that permissionless = blockchain, with or without a tradeable token.

# Purpose
This would be for non-monetary chains. Something closer to blockchain as a database. There is no 'wallet' as there is no exchange of tokens allowed. 

These chains would be used as a central, permissionless database to describe labels we use in the data model for this overall project. These labels don't *need* a blockchain, but the 'central' (known network, with a way to find the entire set of participants) creation avoids the need to crawl the web and 'find' other labels. This has the added benefit of being *able to* reject things that have a namespace collision (if desired — however, I think we probably need to allow collisions of 'shortnames'). It also allows an easy way to follow updates to previous descriptions.

I think something like this could also be used for the PKI chain, as well as for dealing with non-repudiable data. These two usecases would need to scale to high throughput. The PKI chain is particularly interesting, as each users PKI is effectively a signature chain that simply needs to be widely observed to ensure the signature chain is following rules for proper key updates.

# Tokenless
Given the purpose above, I would want the core protocol permissionless and accessible. Acquiring tokens to pay txn fees or use the network is NOT accessible enough. It creates too much friction for onboarding.
## Other Projects
There are several projects/services that you can put your 'state' into a merklized tree and put the root on various (token-based) blockchains. I do not consider these acceptable. These services are donation based, and generally have higher (or at least out of our control) latency for publishing state. Something like this adds a dependency that we cannot control, and indirectly does require **someone** to have a token. We *could* do this to prototype the data model without building any of these chains, but I think you could simply test the data model independently with 'fake' symbols (shared amongst early devs, to create interoperable test data).

# Validators and Stake
I would use a fair and accessible Proof of Work (PoW) function such as RandomX for giving validators 'stake' to avoid any sort of Sybil attacks. Basically, the consensus would be Proof of Stake (PoS), but the ***stake is generated*** by way of PoW! This gets around the need for acquiring tokens directly, but can operate using PoS rules. 
## Normal PoS Systems
Validators 'put stake' in a validation period and earn a reward if they meet some sort of uptime or other protocol defined requirement during said time/block period. 
## The Twist
I would envision the mining of your stake happening before you start validating. You then 'reward' the entire stake to their NodeID if they met the uptime. Reading through the chain, you could then determine their total 'stake' and get a distribution of 'heavier' nodes. This would be the Sybil protection. Of course, each staking period would need a minimum to help limit nodes to 'serious' nodes.
## Losing Stake
We would need a 'slashing' rule if you fail to meet the validation period requirements (uptime, etc.). 

We could also add some sort of decay function so early nodes don't always end up with disproportionate stake, since they have more time to accrue work.
## Gaining Stake
Another way for validators to gain 'weight' would be to potentially charge PoW in exchange for answering queries (spam protection of a different kind?). This allows people who use the system to (indirectly) help protect from a single bad actor gaining a large amount of stake and overwhelming the system. 

Another option to increase the total stake of the system (protection) without requiring everyone to run a full node, would be to allow users to delegate their stake (PoW) to a node operator they perhaps trust (eg a friend). (With a Digital Identity (LLI) in the system, this could perhaps be the mechansim for non-node operators to add weight to their identity, while also helping protect the network.)

# Spam protection
Since we don't have a token to pay txn fees, we could use a PoW function and some protocol-defined threshold to act as a txn fee (could be dynamic based on demand?). Not sure on a PoW algo for this, if we want to allow phones to generate work (for txns) in a reasonable time (I think RandomX would work on phones too? Have to check). In the shared database mode (eg PropertyDefs), the transaction fee would probably be really high, as we don't want unnecessary additions. In the PKI setting this is probably where some sort of dynamic fee, that is set around the actual limitations of the network/consensus, would ratchet up as the system reaches its limit.

# Security
Security can be measured by way of opportunity cost associated with not mining Monero or anything else that does have a token and uses the RandomX PoW. Basically, the only attack vector is someone burning money just to mess with the system. Having a really slow decay rate can help here, as there will be a couple of high stake (trust) anchors for whomever started the system. 

## Consensus
I would currently adapt the SnowBall Consensus (from the Avalanche project) since they have the most scalable protocol for number of validators (therefore most accessible if you wanted to actively validate) and highest throughput. 

A pure PoW (Bitcoin) consensus would suffer from block-reorgs which would mess up our symbols that are already created. 

PoS (SnowBall) has very fast finality times and irreversabilty properties that are ideal for our usecase. 
### Irreversability
I still need to fully understand *how* irreversible state is in SnowBall, and how to 'restart' or 'recover' from a hostile attack (bad actor controls > ~30% of the 'stake'). I believe that the worst a bad actor could do is cause the chain to 'stop'? Since there is no money at stake, attacks are not world-ending for this system. We just need to make sure that there is no 'block re-org' amongst honest nodes.

I still don't fully understand all the nuances of SnowBall consensus, as it is first in a new class of consensus protocols (leader-less). If anyone has a handle on it and can answer my questions, please reach out!

# Scaling
By removing the token/transfers, we no longer need to worry about the double spend problem. We only need to worry about colluding chain re-writes. We are using this form of blockchain for:
- Getting a diverse set of actors to enforce the rules/observe
- Acting as some form of permissionless Registrar (central log of core data)

I believe that doing things of this nature can give us more flexibility in how we shard the network to gain horizontal scaling. We don't need *every* observer validating everything. We just need *enough* observers to reduce the chance that an entire shard could collude to allow some subset of info to be rewritten. The game theory changes quite a bit when there is no liquid/extractable value. All that I know about sharding blockchains is in the context of token-based chains. I know that it basically never works out well. In short, it is very difficult. Since we using blockchains slightly differently I think we need to re-examine sharding techniques through a different lens. I don't have any particular approach in mind, but I think we just need to be careful to not write-off techniques that have been discarded by the token-based chains.
