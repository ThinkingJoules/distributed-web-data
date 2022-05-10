# Merging the two concepts
We have our low level blockchain, symbol registration and logical path layer, and we have the high level semantic data. We need to figure out how and where to leverage the lower to build the upper, and how the user data is handled, synched, replicated, etc.


# Low Level Primitives
I could imagine one of the early chains would define specifications for low level primitives that could be used and referenced in other projects (blockchain or not). 
I think something like our [PropDef](https://github.com/ThinkingJoules/distributed-web-data/blob/main/semantic-objects.md#propdef) for defining the semantic symbols for 'Properties' (Predicates in RDF). Other primitives we need are FILE_EXT (universal types), and UNITs. I'm not sure if LANG_TAG should be on a chain. To make it all consistent I think it should. New human languages aren't really created often, so it seems odd, but I *suppose* it needs to exist. The other needed primtive would be the User/Agents LLI (Long Lived Identity) for their PKI.

For the non-identity chains, the txn fees would probably be a crazy high amount of work. Something like days to weeks (maybe even a month?) of PoW to avoid unnecessary or useless primitives to be added (of course, someone still needs to implement it in software in some sort of client somewhere). Assuming we can use the [chain coordinates](https://github.com/ThinkingJoules/distributed-web-data/issues/2) we can encode these tags as a prefix on all the binary encodings so all data would be identifiable regardless of context. With sufficient work requirement you could easily end up with only a 2-3 byte prefix for all identifier tags. The goal is that this would become something like a permissionless and directly referenceable version of [multiformats](https://github.com/multiformats/multiformats) which is used in IPFS Content Identifiers.

# User Owned Data
The goal of all of these primtive symbols is to build some sort of 'Semantic Object Database'. This database should be able to describe data for ANY app/usecase. The database itself could be thought of as a labeling and encoding protocol. If the low level symbols have semantic understanding, then placing those symbols in a certain, well understood, pattern would give them meaning. If this sounds very similar to literal languages, then you are indeed following along. We are trying to create SYMBOLS + SYNTAX to allow both humans and computers to 'understand' the data. If we have well understood SYMBOLS + SYNTAX then interoperability between both humans and applications will follow. 
## Nature of Unsiloing Data
When unsiloing data, the new API (between 'apps') becomes the query language of this database. The query language can also change, because the underlying semantic symbols remain. However, I think it would be good to build some rudimentarty query language that perhaps higher level languages build off of. Not sure yet. Anyway, in order to unsilo, we MUST have a common understanding of what is what. This forces developers of applications to understand the semantics so when they are looking for data for concept X they know what query should return all of that data (and not miss some of it that meets requirements for X, but is not semantically understood/labeled as such). This sounds hard, but I think it is mostly different. And if this sounds like it will never work, then please propose fixes to these ideas or data will forever be siloed and centralized! I am only positing my best guess at a design. I believe the tradeoff is this: composing (and understanding) API's from different centralized sources or understanding decentralized inputs to semantic labels. It is a wholly new paradigm for developers, but once learned is complete in its design. You would have to learn new symbols as they are generated instead of entire APIs along with their own descriptors for new apps.
## Rough Topology
I do not want a global DHT of data. It does not allow fined grain control, and if users wanted to 'distribute' their data in that manner, it is still available. I just don't want it as the core means of retrieval. Since we want to allow private data, this also pushes back against a DHT at the core. DHT's are great when all the data is public for everyone, and that is why everyone has to share in the burden of replicating/persisting it. If it is private files, now you are just externalizing costs and burdening the system with no 'public' benefit.

So how does my hopeful design work?
### Nodes, Nodes, and more Nodes
First some distinctions on the 'layers' to this topology.
#### Validator
The 'nodes' used to run/validate the core chains would simply have a single key pair and use the pub key as its ID. This is pretty standard in distributed systems. This ID will accrue the work and 'gain' weight in the network over time. If the key is lost, you lose your accrued 'stake' and have to build it back up from zero using a new key/ID. I think these nodes can be discovered through bootstrap nodes, as is common in most blockchain designs.
#### Data
User owned nodes would not necessarily have to participate in Validating. I think of these nodes as simply a computer that can answer queries to other data nodes about the data associated with a [User/Agent Identity](https://github.com/ThinkingJoules/distributed-web-data/blob/main/identifiers.md#distributed-pki) (let us use the term "**LLI**" (Long Lived Identifier) to mean an "Agent" for the rest of this document). These nodes will probably be the ones participating in a DNS-Like IP resolver layer.
#### IP Resolver Network
The goal is to build a robust network to handle updates to where to find the LLI. Here the LLI is basically a chain coorindate to the genesis transactions of a new identity. So it is basically two unsigned integers (with a 'context' integer for which chain these coordinates are for) that is equivalent to a **Domain**. I didn't say 'Domain *Name*' because 2 numbers generally aren't thought of as human friendly. However, the first billion or so identities would have fewer or equal digits when compared to a telephone number, and those are *memorizable*. So human friendly-ish. Like DNS we need a way to change the IP address associated with a LLI. We could add in 'human' names here, however we **MUST** allow collisions. The idea is that if a collision occurs, the LLI itself is needed to know for certain. I think the human names would really only be needed for discovery of or sharing of 'friends'. There is always the ability to extend the proof of work idea to allow ranking of collisions based on who expended more work to 'claim' the top search result.

I think this is purely a 'network' and not a chain. There is no real benefit to having a history of associated IPs. I am thinking something along the lines of [Fireflies](http://www.cs.cornell.edu/home/rvr/papers/Fireflies.pdf). It doesn't have to be this exact design, but I think it is compatible with all of the concepts and requirements of this new system.

# Recap so far
At this point in the design we have a way to identify a user (LLI) and resolve to an IP address to query it for data. This data is of a nature with 'universal' understanding and so can be queried to populate UI's for applications running locally on another users machine.

So things we still need to discuss:
* Permissions
* Repudiable vs Non-Repudiable Data
* Authorship
* Replication/Syncronization/Caching

# Permissions
This is going to be a hard one to solve simply. The finer grain control, the more effort for a user to set their permissions. The implementation of permissions is pretty flexible, in that it really only needs to query a set of three sets. You need to be in the intersection of a Group [of users], Capability, and [Group of] Resource in order to be able to perform the action indicated. The hardest part is making rules/ or adding new Resources to the proper set. If you had no rules, then it would push the complexity to the App to set the proper permissions on creation of new data. This forces permissions to now be part of the larger data system and is no longer 'pretty flexible', BUT is more conventional when thinking about data used in 'X' application is only shared with 'Y' people. I think there are many ways of doing permissions, but trying to optimize for user experience and ease of understanding 'who' can 'see' 'what' ('when'? 'how long'?) is difficult. Hierarchical systems are nice for inheritance, but not all data is hierarchical. The right approach is yet to be determined. I suspect leveraging some sort of query language might help. That implies that, at least for permissions, ***a*** query language will be a dependency. This could/might differ from the query language used within applications, but I assume it will be one and the same.

# Repudiable vs Non-Repudiable Data
Now that we have a way to make data private, and solely in our control, we end up with a bit of a new problem. Can someone 'retract' data and deny they ever wrote it? Siloed data at least requires collusion between a 3rd party with a wide reputation, and a single individuals wants. These generally don't align, and so the 3rd party (centralized app) is gernally trusted to treat all users equally.

Technically it is impossible in this new system to lie, since all things are authored with digital signatures. HOWEVER, in practice it can be very difficult to find and then disseminate evidence of a retraction. This goes back to our issue of [discoverability](https://github.com/ThinkingJoules/distributed-web-data/blob/main/identifiers.md#discoverability). 
## Use Case for Non-Repudiable Data
This is akin to a double spend if the data were a token. If we built a decentralized Twitter and someone tweets something that they later delete, they could claim they never tweeted it. If you cached a copy of it you can prove they did indeed author said tweet through digital signatures. However, what if you didn't have a cached copy? Maybe your server was offline temporarily? Maybe you were online, but the author **selectively disseminatated the tweet to create confusion** and get a group of people to think one thing, and another group to think another? You wouldn't even know that there was a tweet in which to scour the network for proof it actually exists. You would need to query everyone, all the time, to discover deliberate or accidental asymmetric data dissemination. This is only a problem for data that **MUST** be public, and instead is selectively shared. I think most social media/blogs/news articles/etc. falls under this umbrella. I'm not advocating that things can never be filtered or marked as deleted, merely that having a log of it allows context, and even associated reasons why the content was deleted/edited.

I don't think audit logs for companies actually fall under this umbrella. Perhaps their public statements would, but internal data audits are not for the 'public' (except of course if they are being investigated or something). Given that there is a trust/authority hierarchy in organizations, and the applications will probably be custom built, I think there are many ways to create an audit log that is permissioned like any other data a company might only want admins privy to. My comment about investigations and ensuring 'public' auditability does raise some questions. How to prevent the 'shredding' of evidence before an investigation? Might need a different sort of mechanism that is similar. This all falls under the 'Grouping Problem'.
### Grouping Problem
As discussed in the [document](https://github.com/ThinkingJoules/distributed-web-data/blob/main/identifiers.md#blockchains-as-databases) about blockchains, we want to separate state to only allow those who are interested to ***OPT-IN*** to tracking/validating the state. Many people don't use Twitter, but use Facebook, or neither. If the distinction between apps fade, then the idea of a 'tweet' or 'post' cannot be assumed. It isn't clear how much or what type of data should be on any given subchain. If you put too much, then it will take a long time for late-comers to replay all the state, and will require more resources to validate many things they don't care about.

In an ideal world you would have each subject ("tweet") have its own sub-subchain under the LLI namespace. But who validates these? How do you structure it in a way that the log is widely replicated to diverse actors so chain re-writes are detected? If you just let 'friends' validate then, it is prone to sybil like fake outs. I would prefer to keep this state separate from the PKI chain, so this would be a 'public statement' chain? The GLOBAL set of all public statements in a single chain? There are a lot of statements I don't care about. So how do we divide this up and ensure diverse actors?

So let us think about WHY we are trying to use a chain in this scenario.
1) We need diverse replication/validation
2) We need to easily detect (or simply disallow) deletion of content

#1 is really just a way to ensure #2. But validation of this type of data is really simple. It is really a schema validator like our low level validators. We don't care what they 'tweet', just so long as it follows schema rules, and putting it on a chain gives you historical values. But do we need the actual ***content*** to be on-chain? I would argue **no**. 
#### Idea #1 Detached Content
If we know tweet X was created with some state represented by hash Y. Validators simply need to verify that the binary data associated with Y is valid to accept the state. Validators need to ***see*** all the state transitions to know it's valid, but for people trying to know a previous state existed, the hash is sufficient. This is kinda similar to what Bitcoin did with [Segregated Witness](https://cointelegraph.com/explained/segwit-explained). Now if someone publishes state, the parties that did not get the content disseminated to them ***knows*** they are missing state, they just need to find the data that matches the hash. How would they find this state? First, try asking the author. If they refuse your request, then you need to find it else where. What are your options? Ask all your 'friends'. If they don't know the hash then it gets much harder. The only way to get the unknown state is to have a DHT or some other parallel system to publish content, and **hope** that someone who has seen it, publishes it there. This other network would basically act like any torrent service, and suffer from all the same shortcomiongs. If ***no one*** in the network stores the content of a hash, then it is lost forever. The base case is that you would, at a minimum, store your own transitions and respond to queries. Basically, refusing requests is like Blocking someone on Twitter. You can see their tweet if someone non-blocked shares a screenshot of it with you. This illustrates how this approach is compatible with the *idea* of permissions. This solution still doesn't solve the grouping problem.
#### Idea #2 Ethereum Like Rollups
I'm not well versed in all the Ethereum stuff, but I know their scaling tech is pretty neat. Ethereum has various forms of [Rollup](https://medium.com/fcats-blockchain-incubator/how-zk-rollups-work-8ac4d7155b0e) constructions that are trying to solve a similar problem to us. However, since they are dealing specifically with tracking transfers, I'm not really sure how it would translate to our usecase. We really don't want the Zero Knowledge part, because for our particular public data usecase, we don't want the privacy. I'm not sure if these solutions could be adapted, but I wanted to just put it here for discussion.

### Grouping thoughts cont.
So we have a way to minimize the total state of any chain through detaching the content, but how many chains do we need? One for each person? One monolith?

I think the best way to look at it is in the nature of format. If we **must** have a chain to get certain guarantees, then perhaps we can leverage the properties of a chain for our own advantage. If the state transition is simply a binary [delta encoding](https://en.wikipedia.org/wiki/Delta_encoding), then edits can be performed to the 'target'. This would allow git-like operations to any 'value'.

In our data triple we have a Subject, Property, and Value. The Property has all the constraints, semantic meaning, and context for the Value. So what if we have chains based on Properties? That way a logical object (Subject), may have data that is repudiable and non-repudiable with a unifying Subject identifier? So PropDef would have a property like ```non-repudiable``` that is a boolean. Since the property also carries the FILE_EXT as well as any constraints, it can allow each of these chains to have encoding-specific validation of the binary after a new delta encoding has been submitted. If it fails, the 'transaction' is reject by the validators of this PropertyValue Chain.  Depending on the use-case these PropertyValue chains might simply have a ```parent``` field to allow any author to 'build' off of any other logical document by any other author. I could see particular use-cases where that is not allowed, and that is why having separate chains for each property allows flexibility (at the cost of initial configuration, but through self-similarity, tooling can lower these burdens).

To reference a version in your triple store, a Subject with any non-repudiable Property would simply have the chain coordinates as its 'Value'. These would be pointing to the last delta transaction for the document.

It is less clear who might validate these chains. They are sort of 'once removed' from the core chains, but are relevant to many different people. I would assume at first, validators might validate several chains, splitting their work. However, we could make the work re-usable across chains? This would make attacking the whole system easier, instead of making attacking any one chain easier. If an attack can, at worst cause the chain to stop (DDoS) then these non-repudiable chains would be most targeted since it would be the most disruptive to the larger use of the system. However, because the users are more diverse and there are simply more of them, it would be in their interest to put some sort of validation effort in, to combat against a single entity easily gaining >30% of the total 'staked' weight on that chain. Validators of these sorts of networks would need to at least keep the transaction state, and can 'forget' the observed delta binaries they have validated. This will save disk space, but still require CPU and Bandwidth. I could envision some sort of mining-pool like apparatus to where you do work for some validator(s) you trust to execute the protocol faithfully. Or perhaps a simpler design is having validators 'charge' work for answering queries. These ideas could apply to the core chain validators as well. It is someway to add work to make it more difficult to attack, but not require everyone to run a full validation node. It is a delegated trust signal mechanism.

## Non-Repudiable Data Summary
It is difficult to get the right balance. We don't want a monolithic chain like Ethereum, but once you start having more than one chain, it is hard to find a clean and simple division point. I'm not wholly satisfied with my constructions above. I suppose we don't need to determine at a low level what is non-repudiable, and let early apps build their own sub-chains with perhaps all non-repdudiable data for that particular app on a single chain. Then see how it evolves from there. If they are going to reference chain coordinates as the 'value' in the triple then that is all I proposed. The difference is that everything is repudiable in the base system, and we can 'key' up what an LLI has authored on various non-repudiable sub-chains, but in an ad-hoc instead of methodical manner.

## Repudiable Data
Repudiable data is the data as we mostly know it currently on the internet. [The Wayback Machine](https://web.archive.org/) is the only large scale project that takes some snapshots and makes these points non-repudiable. However, since they need to crawl the entire web, granularity is lacking and there are many intermedate repduiable states they cannot capture. 

I think repudiable data is pretty straightforward. I would propose using a [CRDT](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type) like GunDB to do data merges and allow offline editing. This data is what will be mostly used as the 'value' in the triple. Anything that is private by default would be here. Getting the two different data types to work together might be tricky, but there is no way (that I can see) how to get non-repudiation easily AND selectively. They are simply different paradigms. The best we can do, is make sure that the non-repudiable data can ***fit*** in to the rest of the triple pattern so it *looks* like repudiable data and can then integrate into the larger indexing and querying systems that will be built around that.

# Data Authorship
So verifying an author in non-repudiable data is pretty easy, since you have a signature at each state transition. However for doing repudiable data we have two options. Sign every state on every change, or sign a root of a merkle tree containing all the data. (Note: I would pick a merklized PATRICIA Trie since these build in an idempotent and deterministic manner).
## Sign all the things
This one is straight forward and easy to reason about. Any keys that are part of you LLI can sign at any time and publish the data widely (and wildly) and the CRDT will deal with it at ANY point in the network. Pros: Very flexible at the network level. Cons: LOTS of signatures for the entire system to validate (CPU). This is the approach GunDB takes. Everything is signed on your private graph and the network validates all signatures. The data itself is self-proving (With the LLI scheme, you would still need to see if the particular key they signed with is currently valid, so it is really self-prov**able**). We can speed up signature verifications using Schnorr Signatures [batch verification](https://bitcoin.stackexchange.com/questions/80698/schnorrs-batch-validation) property.
## Sign Once, share proofs
This one probably fits with the topology of the network just fine as I've described it so far. This requires the CRDT to only happen on data amongst your own private servers. One of them creates a root by hashing all branches of the trie that changed since the last 'commit'. Once fully hashed, you sign the root once. You can only 'prove' data that is part of that root. Data within the tree has no way of knowing if it is authentic without asking the LLI endpoint for a proof. The upside, this proof can verify N pieces of data. It is a bit more bandwidth (maybe not? signatures are long) but you can perform 1000's of hashes before you can do a single asymmetric key verification. This proof could potentially become zero-knowledge. This would slow things down some, but would save on bandwidth, and not reveal sibling nodes in your trie. Not sure what the break-even would be for Zk vs conventional merkle proof.
## Tradeoffs
Signing everything allows public data to be widely distributed and cached anywhere. However, if you want non-repudiable public data you can't use this method at all, and I think most fully-public data is going to want this quality. So I think we can take that out of consideration. The closest you could get is widely shared, semi-public data (think large organization/institution). As mentioned before with regards to an organization setting, if you trust your own software, and your sys admins, then you could store data in a log like manner in your repudiable data store. It really depends on your trust profile. This brings up a good point though. If we only have a single signature at the root, but we allow other authors for subsets of the data, do we not still have to store their sig *within* our trie? Or do we just say that they edited it but throw out the sig, since the main server (the one creating the merkle trie) always has more authority and is trusted as the arbiter of truth in this federated namespace? If we want to know *who* last edited it, we would need need to either build a repudiable log of changes or we need a spot for this info alongside the CRDT metadata. Putting it in the CRDT metadata makes the most sense as you only care about the *last* edit. You can still build a repudiable log, but now don't have to record the editor manually.
## Futureproofing
Quantum proof signatures are big and ugly and expensive. It is *when* not *if*, and I don't foresee quantum proof signature being very performant when the need arises.
## Authorship Summary
Given the following:
* The proposed network topology (data is always queried at source)
* Fully public data is almost always going to want to have the property of non-repudiation
* Quantum Proof signatures suck

I propose the signed root be used as the current design for repudiable data authorship.

# Replication/Syncronization/Caching
I envision an owner of an LLI basically having a couple servers, one of which is 'public' that answers the queries. Tooling would be needed to help simplify the configuration and orchestration between these. One trick could be to add the locally generated key on these servers to your LLI/PKI. Then all servers 'know' who they belong to and who it is safe to share private data with. There is still going to need to be some amount of configuration, depending on the device, amount of storage, etc.

Regardless of the authorship structure, I think the data will probably end up in a merklized PATRICIA trie for, at minimum, the Syncronization attribute.
## Replication
As discussed before, you might replicate data for your friends, so in the event of data loss, they can recover files. I will go in to more details later on what I mean as 'files'. I expect the files will be encrypted before sending, and will be opaque. The only thing you know, is that you are storing this for only the people you have selected. You cannot answer queries on your friends behalf or know the contents. Perhaps this can change, but being able to answer queries makes you now need to know ALL of their permissions. This is perhaps where the magic zero knowledge stuff comes in, but to start with, I am going to keep this simple.
## Synchronization
This is mostly between your own devices. The key issue is repudiable data (basically key-value data) is hard to syncronization without asking for all the keys. Using a merklized prefix trie would allow you to probe the other servers and then sync a set of branches. Since the merklized patricia trie is effectively a copy on write structure the online node could track a root when the other goes offline, and when it comes back it can even know in advance exactly what updates to send the now back online node. There would still be a level of diffing, as the offline node might have new offline changes to send the other direction as well. Either way, having a trie makes it much easier to reason about how to potentially save a lot of time on restarts.
## Caching
If you are running the app locally and building a view, it would be nice to cache all your friends data changes constantly in the background. Then you can simply query your local database in their namespace to save time and avoid network calls. Your server could always send out another query to ensure the data is accurate. If it isn't, then as long as you can push new changes to the app view (through websockets if in the browser) then the view will update with data as it comes in (GunDB used this pattern extensively and works really well). Having a prefix trie allows you to query or cache an exact match of your friends data to allow really fast view builds. The synchronization technique could also apply to a friends servers if you are trying to hold a whole subtree of their data locally.

# Describing the content of 'Value' in the triple
One reason to hash the data of the value, is to provide de-duplication. This new system is not inherently content addressed. Hashing is nice to 'compress' the identity of large binary blob into a relatively small value. Since we are trying to treat 'primitive' datatypes as if they are files, we also want to treat very small files, as if they were 'primitives'. If we had a file that was only 16 bytes, would we really want to hash that? So ideally we want:
* Small values 'inlined' to prevent excess disk usage (and increase query performance)
* Large values hashed, and ideally chunked (to allow partial fetching. ie: streaming a video).

If we also go the single signature PATRICIA trie route, we need to include whatever bytes are at the 'value' in our proof. So if we have large values they will increase our proof size. If we are optimizing for proof size, then we want to inline anything less than our hash length and hash anything longer than our hash length.

So what is all part of the our 'Value'?
* Metadata
  * CRDT Metadata
  * Last Edited By
  * Last Edit Time (if similar algo to GunDB, a unix TS is part of CRDT metadata)
* Binary Blob
* [Suffix(es)](https://github.com/ThinkingJoules/distributed-web-data/blob/main/semantic-objects.md#suffix-cases)
  * FILE_EXT
  * LANG_TAG
  * UNITS

Do we want our low level system to know about some or all of the metadata? Is the CRDT optional? There are many CRDT algos, what if different data or usecases wanted different merge characteristics? I think we would need to pick a single CRDT algo. If a network had many different algos then negotiating and dealing with all the different types of metadata would be difficult. I'm not familiar enough with CRDT's to know how much overlap their might be, but I would imagine gaining different properties through different algos will require categorically different metadata.

## Provability
Regardless of authentication method, as some point we are going to sign some bits. These bits may or may not be a hash, the point is what do we include in the 'Provably Authored' bits? Another way of stating this; what parts of the 'Value' are malleable without invalidating the signature?
### CRDT Metadata
This does not matter if it is changed, because ultimately we are only using this to arrive at some state that we are ***then*** going to sign. 
### Last Edited Time
We generally regard metadata as non-authoritative but we also use it extensively, else why would we have it? We will talk more about time below.
### Last Edited By
If we did a trie data structure in a company setting, I don't think you need this as part of the signature. If we were to include it, then we should include their signature for the updated state, so this is truly authenticated author. This would basically be a mix of 'Sign all the things' *within* the domain namespace trie of some 'company' LLI. This only makes sense in that context. Basically the more widely shared, trust decays, and this usecase can become more relevant. I think to allow all potential use cases it should be permitted, but not required. The obvious opt out, is you wouldn't want to sign all of your own changes within you own LLI namespace. So once we add the ability to opt out, it can be used however. Adding it will significantly increase proof size. This requires that the merkle proof itself be correct && any signatures *within* the proof to be valid in order to trust the 'Last Edited By' data. So you end up with double verification, some of which (signatures) is very expensive computationally.
### FILE_EXT
Re-interpreting the bits will almost always lead to a different 'understanding' (often corrupt or nonsense result) of the intended value. FILE_EXT should remain as part of the official 'value'. 
### LANG_TAG
This feels a little different. Since the base encoding can't be interpreted wrong by the computer the text will still be correct. I was once a website that I was reading in English, but the browser prompted me to translate it to English. I simply ignored the prompt since I can judge for my self. I think LANG_TAG *could* be ignored. It really is metadata hints more than a 'binding' declaration.
### UNIT
I think this is very much a human corollary of 're-interpreting the bits' for FILE_EXT, it is just in textual (usually) base 10 form. So this would need to be included as well.

## Privacy and Provability
To prove things in a merklized manner, you must supply enough data to reconstruct the path back to the root. In a regular merkle tree these would simply be a series of hashes. However, in a merklized PATRICA ***trie***, we must supply all the plaintext data for nodes on the path to the root. So anything that is required to be part of the 'bits' needs to be given. This means if we have the following:
```
key1 = "LLI"
value1 = "Secret Data"

key2 = "LLI/Subject1"
value2 = "More Secrets"

key3 = "LLI/Subject1/height"
value3 = "6"

```
In order to prove the Subject1/height is indeed 6, you would need to supply "More Secrets" and "Secret Data" in the proof. Obviously our namespace LLI and the subject without a property aren't really 'triples'. Only key3/value3 logically contain our triple (Subject1, height, 6). As long as we don't store data on things less than a 'triple' key, we are fine. This is the tradeoff for gaining idempotency and determinism. So we need to be aware of prefixes in our trie.

## Repudiable Time Log
So what if we wanted to easily create a log? If we already have the idea of prefix trie, then simply extend the key:
```
LLI/Subject/Property/Timestamp = value at timestamp
```
We basically move the timestamp out of the metadata and into the key. This is now a 'Quad' key. This breaks our 'everything is a triple' but makes it really simple to reconstruct past states. Just need to ask for all keys in the subtree starting with ```LLI/Subject/Property/```. This is really simple given a prefix trie. If we don't want to include quad keys are part of our signature, we would simply pretend the triple key has no children and hash accordingly. That way this log can stay in the same structure but not be considered 'official' as part of the signature. However, trying to ignore them in queries is trickier, especially on a subtree search.

### Triple, Quad, ...
Let us just make up some stuff and see what it looks like.

So an LLI would be ```dr:3.1.2``` this is indicating the subchain created in block 3 of the registration chain, and on that chain(the identity chain for this example) we are referencing the identity created in block 1, 2nd transaction within this block.

A subject is curious, as ultimately the properties should define it, not its symbol. So if you were online and all your devices sent subject creations to the 'primary' server, in theory, you could simply increment an integer. If you are editing offline, then a local identifier could be created in the app, and then when back online, could send a 'create' message to have your central server designate a subject. If you want to do this completly offline and have it always work, then simply use some sort of UUID or another high entropy scheme. So with a subject we would have ```dr:3.1.2-subject1```

Property is going to be chain coordinates (let us assume block=txn) ```dr:3.1.2-subject1:2.75```. This is our 'triple' key

If we added a timestamp ```dr:3.1.2-subject1:2.75@1652212408``` This is our quad key for repudiable logs.


Can we nest things by putting another subject as our subject? ```dr:3.7.1-dr:3.1.2-subject1:2.44@1652212408``` This would be more in RDF mode where user 3.7.1 is saying something about dr:3.1.2-subject1 and describing what they are saying as property=2.44? I think to allow this sort of encoding, we would need to make our own version of something like a CID from IPFS. The downside is that our URL's are now much more opaque (numbers really aren't great, so it isn't much of a loss).

I have no idea the right way here. Just throwing out some ideas. 

If we think of the triple as a sort of graph, where the Subject and Value are nodes and the Property is an edge connecting them, then could we add properties to edges with:
 
 ```
 dr:3.1.2-subject1:2.75:2.37
 ```
It looks a little funky, but it makes sense? This data wouldn't really fit RDF, and it wouldn't really fit a full labeled-property graph (the edges are undirected). You could always assume the edge is directed from subject to value, and if that value is another Subject then it makes sense. So maybe this would be similar to something like [Neo4j](https://neo4j.com/docs/getting-started/current/graphdb-concepts/)? 

Circling back to the privacy issue; if we did properties on edges, then the value the edge is pointing to would be exposed during proofs. We could fix this by doing something like (let us call this the expanded form):

 ```
 dr:3.1.2-subject1:2.75/data = binaryBlob
 dr:3.1.2-subject1:2.75/crdt = crdtMetadataBlob
 dr:3.1.2-subject1:2.75/lastEditedTime = unixTS
 dr:3.1.2-subject1:2.75/lastEditedBy = 3.1.2
 dr:3.1.2-subject1:2.75/FILE_EXT = 5.55
 dr:3.1.2-subject1:2.75/LANG_TAG = 6.1
 dr:3.1.2-subject1:2.75/UNIT = 7.68
 
 dr:3.1.2-subject1:2.75:2.37/data = binaryBlob
 dr:3.1.2-subject1:2.75:2.37/crdt = crdtMetadataBlob
 dr:3.1.2-subject1:2.75:2.37/lastEditedTime = unixTS
 dr:3.1.2-subject1:2.75:2.37/lastEditedBy = 3.1.2
 dr:3.1.2-subject1:2.75:2.37/FILE_EXT = 5.32
 dr:3.1.2-subject1:2.75:2.37/LANG_TAG = 6.1
 dr:3.1.2-subject1:2.75:2.37/UNIT = 7.22
 ```
We could then prove anything we wanted independently without exposing values with a common prefix. So it can work, it is just a matter if we would want to start thinking in prefixes instead of triples.

Lots on uncertainty at this point the document. These are some really big factors and I personally don't have enough experience to confidently assert 'the right way' to move forward. The timestamp seems like a really straight-forward case, as it would be difficult to get the same effect as easily. Adding properties on edges gets kind of weird, but I can imagine when it could be really useful. It would make performing queries and indexing a little more complicated. 

# Inline vs Hashed
If we are working on our expanded form above all the ValueProperties are going to be small except the binaryBlob. This is where we would want to have the option to have it inline or a hash. If we use [Blake3](https://github.com/BLAKE3-team/BLAKE3) as a hasing algo, we can verify 1024bytes chunks using [Verified Streaming](https://github.com/oconnor663/bao/blob/master/docs/spec.md).

To generalize value, someone might ask for ```dr:3.1.2-subject1:2.75/data/0``` for the first chunk or ```dr:3.1.2-subject1:2.75/data/?start=65&end=2543&noProof=true``` request chunks that include bytes 65..=2543 of the file. If noProof = false or not included, it would send the proper data to verify the chunks with the root hash which is present at ```dr:3.1.2-subject1:2.75/data```. We could potentially use the patricia structure itself, in which case there wouldn't be a hash at ```dr:3.1.2-subject1:2.75/data```. The downside to this approach is that the content hash is no longer portable, since it now implicit and dependent on our PATRICIA trie node encoding spec.


--- WIP ---
# Looking at Multiformats and CID
IPFS uses the aforementioned 'multiformats' to build their [Content Identifiers or CIDs](https://github.com/multiformats/cid). Is this new system going to have a CID or something similar? I think perhaps. 

Let us explore a different representation, where we move the concept of CID up to the concept of our 'value'.



IPFS uses a single table in an attempt to save a byte for context namespace. I think it is best to break ours out to separate 'tables'(chains) and simply be slightly more inefficient, but the tradeoff is permissionless extensions and directly addressable/dereferencable definitions.

### Multiformat - Address
[These](https://github.com/multiformats/multiaddr/blob/master/protocols.csv) are URL-esque paths and schemes for addressing endpoints. I think it would be good to have a primitive equivalent to this. These symbols would allow multi-transport clients to route data as needed based on specific application use case. These chain coordinates can then be used as a universal alias for the 'scheme' portion of a URL.

### Multiformat - Base
[These](https://github.com/multiformats/multibase/blob/master/multibase.csv) would be useful for understanding a particular text encoding of binary data. I don't know that these are strictly necessary. If this new system is binary instead of text based, then these would really only be needed when transferring data on text based protocols. I personally feel it is up to the transport(address or scheme, from above) to specify encodings in their own way, as the text/binary dichotomy has always existed and is something for the protocol to determine. I think there could be a primitive 'Base' chain.

# Extending Primitives
If you are making data machine readable, should you make it machine operatable? If we added another chain for [OPCODES](https://ethereum.org/en/developers/docs/evm/opcodes) we are well on our way to creating a universal (within this system) way to add functions to some sort of new machine semantic programming language. This seems extreme, but that is what is great by having this multi-chain system. We can just create the primitive chain first so we can have numbers and stuff, and then build on top of it later. That is the beauty of linked/semantic data. An interesting extension of this would be adding a suffix to primitives and can build conversion tables. Converting inches to feet or anything else the system has defined could be semantically understood and auto converted.
