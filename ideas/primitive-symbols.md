# Goal
The goal of this document is to explore, in depth, how we find/create Atomic Data's "Property Definitions" (PD for short throughout this document). (This could also be generalized and re-stated as: How do we find/create widely understood Symbol->Semantic Meaning mappings in the context of the distributed web?)

We first need to agree on the requirements, and then we can explore the known options for how to meet those requirements and accomplish our goal.

# Define Requirements
These initial requirements were from a discussion between me and @joepio.

- Immutable — Symbol -> Definition mappings cannot be updated directly.
- Discoverable — Developers need to be able to easily find ALL definitions created by ANYONE. This is needed for maximum symbol reuse. (Ideally we could find 'updates' to older symbols easily as well. Call these 'Synonyms'.)
- Replicated — Definition data is still available if the author's server goes offline.
- Addressable — Must be able to dereference the Resource to view the Property Definition.
- Permissionless — Anyone must be able to create new Definitions

Immutability can be done with content hashing. We can extend content hashing to content addressing for replication, indirectly covering the addressability requirement as well. (There are other ways to handle these, so I'm not too worried about them.) Discoverable/Replicated/Addressable/Permissionless are all intertwined based on *how* we build the system. 

The hardest requirements to reconcile (in my opinion), are Discoverability (max reuse of existing Definitions are essential) vs Permissionlessness (new symbols must share addressability for compatibility within the larger system).

# Permissionless vs. Discoverable
The easiest way to make something permissionless is to put it behind a namespace. Let's make the assumption that global namespaces (users) will have to be permissionless. If this is the case, any data created within a user namespace will also be permissionless.

Let's logically walk through our options and, for now, only be concerned with: Permissionlessness and Discoverability.
## One vs The Web
### Web Crawling
RDF is built around the idea that anyone can say anything about anything else. The URIs are namespaced (prefixed) using domain names. The problem with discoverability (in the PD usecase) is that not *every* domain will create PDs, but the only way to know who created new ones, is to ask *every* domain if they have. And since you don't know *when* they might create one, you need to poll *everyone*. This can be done. We use the current internet, nearly exclusively, through this method: by way of search engines.

While permissionless, it is usually not feasible for any single individual to undertake such a monumental task of crawling *every* domain. 
## One vs Subset of the Web (Author Registration)
If we knew *which* subset of people *have* published PDs, then we could just poll this much smaller subset for new or updated PDs. Much more feasible. So where would these people 'register' so someone/anyone could build an index of all PDs themselves?

This registration could be *within* someone's namespace. But because it is *their* namespace, they could choose to ignore you. In this context, namespaces **would be permissioned**. Put another way, you would have to *trust* that such a registrar is *fair*, but they would have the option to unilaterally delist you from the search index. 

This doesn't have to be a problem. Anyone can create their own index list. But now we are devolving back to Web Crawling...

Another problem is trying to prevent everyone from registering, especially if they have no intention of creating any new PDs. Keeping the list small is important because we still need to poll everyone on the list to discover new state. Polling is very wasteful of resources. 

Can we make a system where authors *push* data, so we don't have to constantly poll them?

## The Web vs One
On creation of a new PD the author could contact *everyone* in the namespace. This is the inverse of crawling the web. But how do we *require* the author to expend all the necessary effort to widely disseminate their new PD? We can't, and if they don't share, it will never get reused and data will never be interoperable. 

Authors of normal websites generally *want* to be heard, and so are incentivized to make their content found. However, with something like our PDs, no one really benefits directly, or immediately. A PD might be created, and even if we had a *pefect* system, it might take a while for someone else to *need* the same things. We don't know *who* or *when*. The person who doesn't *know* they need it yet, cannot predict the future, and the person who created it *already has what they need*. So who is responsible for making this data discoverable? I would argue, it is *everyone* using the system. The problem is sharing the cost. It cannot be entirely on the author to disseminate nor on the consumer to crawl the entire web.

If we could *know* intermediate people would gossip reliably, then the burden of effort would not be so incredibly high on either party.

## The Web + Gossip
Gossipping feels like the right way to spread the cost amongst all participants of the system. Let's *assume* for now that there are enough honest actors that we can fully disseminate through gossip.

Okay, but what if someone is offline when a new PD is gossiped — how do they learn of it when they come back online? How can you *know* you have the complete set of *all* PDs? Remember, when coming back online, anyone we ask directly *might* only have a partial set (they may *also* have been offline during a different time period). The only way I can figure, is that when coming back online, you ask everyone for the hashes you don't have (so we need an efficient way to find the symmetric difference between two sets of hashes). If you are receiving gossip while coming back online, then once you have contacted *everyone* you can know you have the complete set. So we still have to crawl the web *once per restart*. This is better than polling constantly.

Can we make restarts more efficient? If we could know who was online (and fully up to date) while we were offline, we would really only need to ask that *one* person. If we have a way for everyone online to keep a log of those who have gone offline, then this could work.

## Gossiping Members
We can use a membership protocol such as [Fireflies](http://www.cs.cornell.edu/home/rvr/papers/Fireflies.pdf) to have some guarantees and a way to reason about gossiping. Everyone participating basically needs to constantly expend (some minimal) resources to know who is online (i.e. who is part of the set). Fireflies has the added benefit of having guaranteed dissemination, as long as the number of bad actors (non-gossipers) are below a configured threshold. This is great, because up to now, we were *assuming* a way to gossip reliably. 

Can everyone on the internet be a member? I doubt Fireflies or any other membership protocol could reach internet scale in a *single* set, so not really. Is this okay? I think it is fine. We just need a diverse set of people replicating and keeping track of the set of all PDs. Use of any membership protocol puts our system back to being sort of like 'registered author' mode, but now it is permissionless, with the tradeoff being bounded in size. 

So instead of putting the restart cost on any particular member, we spread the cost to *everyone* in the system. Good, feels like we are making progress. When we come back online, we would need to keep track of 'notes' that are in the *same epoch* as when we went offline. This is the subset of all people who were online *since we went offline*. We could perhaps extend the 'note' format to include a flag for whether they are 'synched' or not as well. Then we can detect everything we need from the note.

## Recap thus far
We have created a single set of people capturing a single (complete) set of data. Using a hash structure (presumably tries) we can create symmetric differences quickly to aid in syncing. This is *technically* a DHT, but as described everyone holds the entire hash table. So it is technically a Zero-Hop DHT. If the state gets too large, we can shard using a One-Hop DHT. Then, if the membership gets too large, we would need to move to a Multi-Hop DHT (Read Section 7 toward the end of the Fireflies paper).

Before moving on, I do think we need to *really* understand the ramifications of designing the system as outlined above. In essence, the network *as a whole* has authority through the shared, common, ruleset; no single individual has authority to make decisions that do not conform to the network's ruleset. So, I would argue, the *network* is its own namespace. 

In this 'namespace', the shared ruleset acts broadly like a *type* of permissioning. However, permissionless *content* creation is what we care about, and the ruleset does not *have* to place restrictions on content creation. 

## First problem with our construction: Spam
Since we are permissionless, we still need to solve a key problem: Spam. Why do we have spam now, when we didn't worry about it before? In our membership construction, we are requiring all effort be shared across the whole network... This creates a new lever for bad actors to pull.

If the *network* owns the namespace, how does it collectively police the usage of itself? For a start, honest actors would not gossip about PDs that do not conform to the schema defined in the network protocol. 

Someone could still play by all the rules but *lie* to people coming back online: If we add a 'synced' flag on the note, and someone coming online contacts the liar, the liar can inject unlimited state to the syncher. There is no reason the network would know about this; the liar is a bad actor and does not have to gossip their spam PDs and make themselves obvious to the wider network. How can a sycher detect the false state? If they can't, then they will repeat the injected state to anyone who contacts them for all the PDs.

This seems bad. However we are not the first distributed system to have this problem. 

### Potential Solution: Proof of Work
It is common to use Proof of Work to make an attack more expensive. If we required *every* PD to include a proof that some threshold of work was done to it, then an attacker would be limited to the amount of *resources* they were willing to spend to attack the network. On syncing, or gossiping, you would simply drop all PDs that didn't meet the work threshold. 

We really don't care about authorship, since the network itself is the true namespace. Since we are permissionless, using PoW is the only real mechanism to put up boundaries in this context. The upside, is that verifying PoW is *much* faster than verifying a signature.

### Extending the Solution: Pending PDs
Let's make some adjustments to our 'protocol'. Instead of dropping messages that *only* fail on the PoW metric, we could hold on to them for some time. We could also create *two* PoW thresholds: a lower one to get a PD accepted as 'Pending', and a much higher one to make it 'Official'. If a PD does not reach the 'Official' threshold by some protocol-defined 'Time To Live', then it will be dropped and not added to the set of all PDs.

In the same spirit, we could also set some sort of threshold of time that a PD *must* be pending before it is added. If there was some sort of 'voting' period, then we could even have people supply PoW as a Nay vote, to counter the Yea votes. A chat system (since we are building this on a gossip protocol already) would enable direct discussion — we could create one chat room per pending PD. People that want to join the chat rooms could put up some Proof of Work to enter, and this 'entry fee' PoW could eventually be assigned to either Yea or Nay (so the work would not be 'wasted'). These are all just options to think about.

Is using PoW to vote Yea or Nay permissionless? That is an interesting question. Because the core protocol is going to decide based on the winning PoW total, and the PoW function *itself* is permissionless, then someone who wanted a PD to be official over others' disagreement would simply need to expend *more* resources than all of the Nays combined. Because the *protocol* doesn't care about anything but doing the math on the 'votes', then I would argue this is still permissionless. It is *much more difficult* to swim upstream, but it is by no means impossible. I haven't thought through how all of this voting might work, but I think something like this *might* work. The discussion would be whether to explore this or not.

#### Latency
If we went with some sort of voting time period, this would greatly slow down the speed at which a developer could build a system that requires a new PD. This is bad. However, we don't want our PDs to grow too fast or have duplicates. So the voting period might be a way to nudge a developer to make sure they *really* need to create a new PD. 

The downside is, at the beginning of the system, there won't be many properties (obviously we can try to put as many well known properties in the system ahead of time as possible). This will create the worst possible developer on-boarding experience when the project is most nascent and *needs* a good experience to grow and become adopted.

While a concern, I don't *believe* this is ultimately a problem. If we have symbol discovery/search, and if we build out PDs for the most likely early usecases developers might be interested in, then *those* developers building those *particular* usecases will have a (hopefully) magical experience with the system. The earliest devs that believe in the vision of this system will be laying the foundation that will rapidly improve developer experience as time progresses.

As we get more and more PDs in the system, the voting latency will become more important. If we have a million PDs, there is a pretty good likelihood that new PDs will be duplicates.

## Another problem with our construction: State Alteration
How do latecomers to this set of all PDs *know* that all the rules were followed? 

So far we have a series of votes for PDs, but we have no way for the network as a whole, to 'remember' the outcomes reliably. (For example: a bad actor could still alter the state in their set, and disseminate their false state to anyone who asks them directly for it.)

If we can't have a way to reliably know the past decisions, then making the decisions in the first place is pointless. 

This means we will need a log of all things that were gossiped and voted on. And, as is hopefully self-evident, there needs to be only *one* log for the entire network.

### Consensus
To achieve a single log of events, we need the computers within the system to unanimously and quickly resolve conflicting states (i.e. achieve consensus).

Computers are controlled by people, and those people can decide whether to contribute to or attack the system with their computers.

We cannot simply go by the *number* of computers (that is, we cannot hope that good actors outnumber bad actors), because we have no way to know how many computers might be controlled by a single person ([Sybil Attack](https://gitcoin.co/blog/a-community-based-roadmap-for-sybil-detection-across-web-3/#:~:text=What%20is%20a%20Sybil%20attack%3F%C2%A0)). If we did rely on the number of computers, it would be trivial for an attacker to programmatically create hundreds of nodes on AWS, to increase their 'voting' power in the system.

We need a better proxy for a single voter, something that increases the cost for being a bad actor. (In other words, we want honest actors and bad actors playing the *same game*.) This would mean requiring both to expend resources in a manner that cannot be easily faked, but that remains digitally verifiable.

Like spam, this is not an uncommon problem in distributed systems. We used PoW for spam protection, and we can reuse PoW here (as a slightly different mechanism). We integrate this PoW *into* some form of consensus algorithm; through the combination, we can build a system that reaches a singular decision (i.e. achieve consensus).

### Voting on PDs
Now that we have a single *true* log for the network, we can discuss if we want to add additional curation mechanisms: for example, the voting idea introduced above.

We can use such a Yea/Nay voting system as a way to filter what is allowed on the official log. In essence, we could have humans coming to consensus on each PD through Yea/Nay votes, and if a PD is successfully voted in, it would then be added to the singular log (kept in check by the low level consensus).

If we do end up adding this curation layer, the effect would be two systems working together. They all use PoW, in various ways, as it cannot be faked, is digitally verifiable (self evidently), and the method to produce (CPUs) is ubiquitous and accessible to all participants.

## Full Recap
The system described above calls for a single, permissionless, append-only log that is sybil-resistant and has spam protection. This is achieved by way of a membership protocol (low level validation nodes which have their own 'weight' for sybil resistance), whose members run consensus on a single true (to the protocol) log, and a high level protocol for 'voting' on what can be appended to the one log. Newcomers or outsiders can ask any of the low level validation nodes for a copy of the log, and all of them will give the same answer (hence *one true* log). Newcomers can replay all the votes to know that the protocol was indeed followed, and the final state is indeed correct.



