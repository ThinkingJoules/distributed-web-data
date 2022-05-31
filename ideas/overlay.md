# Context
The goal of this document is to create a shared understanding of what an Overlay Network (ON) is, how this is distinct from a network protocol, and how both are distinct from the tooling built around combinations of ONs and protocols.

The first part of this document explains in detail ways to think about the aforementioned components/layers. The second part of this document will explore how to create ONs with properties that are helpful to us in building out a new distributed web (dweb).

Some basic info about Overlay Networks can be found on [wikipedia](https://en.wikipedia.org/wiki/Overlay_network).

# Part One: ONs vs Protocols vs Tooling
## Aspects of Overlay Networks: Bootstrapping and Mapping
*Any* overlay network needs some sort of hard-coded IP Address built into the client in order to bootstrap itself into existence. (This is true for DNS, as well. For DNS, there are [13 root server IP Addresses](https://www.icann.org/resources/pages/what-2012-02-25-en#:~:text=What%20about%20root%20servers%3F) that are used to 'bootstrap' the DNS registry.)

The first step to bootstrapping is to find relevant participants for the given ON construction:
- DNS starts with the 13 root servers; these root servers know about all the TLD servers. This finds us all participants, and then one can know *all* domain name -> IP Address mappings. 
- In a DHT, participants joining a network must be able to 'find' their neighbors (there are many constructions, but they are all conceptually similar to this idea). In theory, a participant could 'walk' around the nodes in a DHT by always asking a person who their nearest known neighbors are.

The point of bootstrapping is to discover the system's (ON's) 'map': that is, how the system goes about mapping some *symbol* to some *concept*. 

As a user, we usually see this in the form of a URL (I will use URI for the rest of this document to encompass hash based identifiers as well). Each ON will have its own understanding of how to deal with the URI. Technically, an ON could forego the use of URIs and use its own encoding rules for 'identifiers'. I think this is probably not advised, as many other protocols that could potentially be reused or integrated to any particular ON will probably have been built around rules defined in the [URI RFC (3986)](https://datatracker.ietf.org/doc/html/rfc3986). This is why there is a 'Universal' Resource Identifier, to allow many different systems and protocols to be woven together and reused.
### Adding new concepts to the 'Map'
How a user adds new information to an ON, is a matter of how the ON is set up. 

DNS is permissioned and regulated by [ICANN](https://www.icann.org/resources/pages/what-2012-02-25-en), and only by their authority can new TLDs be created. TLDs in turn then have permission to allow/disallow certain domain names within their TLD. 

A DHT is generally used in a distributed system that allows permissionless addition of hashes (and content). 

Any permissionless ON needs to be careful how both 'spam' and attacks upon the system are guarded against. 

### Shared software
All participants within an ON must share the same understanding of how the protocol works. This may seem obvious, but I want to state it explicitly: depending on the nature of the ON, the protocol may need to plan for malicious actors running non-protocol compliant software. 

## Examples of ONs
### DNS
DNS maps a naming convention to a *location* (IP Address). 
### DHT
A DHT on the other hand, maps a hash with its corresponding data. A DHT is considered *content addressed*. This means that the data within a DHT is not mapped to a *specific* location. A DHT uses the closeness of the hash to the node identifier to create a sort of fuzzy mapping to a location (IP Address). This fuzzy mapping allows specific nodes (servers) to go offline, and still allow users to find the correct data. The multi-way mapping is handled within the rules and logic for the particular DHT implementation (ON).
### Blockchain
I don't think most people would consider blockchains to be a form of ON, but I think they conceptually fit our definition. They often build directly from IP Addresses on up. You can find all participants on the network, and collectively create a shared mapping. How the ON collectively creates the shared mapping is more complex than any other ON discussed, since there are consensus algorithms and many other complex pieces to the puzzle, but it all results in a shared mapping. Thus, I consider blockchains overlay networks. (If the ON is permissioned, then I would call it a shared append-only log. I believe being permissionless is what defines a blockchain as separate from a shared append-only log.)

Blockchains generally don't use URIs to do their mappings. For example, Bitcoin uses [RPCs](https://developer.bitcoin.org/reference/rpc/) as a sort of typed symbol and returns a typed concept. Blockchains often use RPC, because all the data is contained within the shared append-only log of data. This makes running a remote procedure call logically equivalent to running it on your own log. The singular shared log is effectively the singular concept that blockchains are mapping to: they just don't address it in URIs. This is mostly because the cryptographic structure of blockchains makes them about the most useless data structure to use directly. So all nodes in the ON must process the entire log, and extract information. This is often why blockchains have to build all their own tooling; there is no meaningful 1:1 mapping to the shared data structure. 

### Summary
Hopefully the above examples illustrate that symbols do not need to be exclusively names or hashes and the concept a symbol maps to does not need to be an IP Address. The 'concept' is anything that other participants within the ON can understand. If an ON uses 1:1 mappings it can directly identify data, and may or may not use the URI spec.

## Tooling vs Protocol vs ON
Before going any further, I want to dive into the internet we are all very familiar with: the browsable hypertext (http/html) web.

*DNS* is an ON built as a tool to make IP Addresses easier. (We [humans] use DNS as a way to bootstrap ourselves into the hypertext web.)

Per the *http* spec, a request allows a 'host' to be either an [IP Address or DNS](https://stackoverflow.com/questions/50321842/http-is-an-ip-address-allowed-in-the-host-header-field). This implies that if all servers only hosted a single http website, then *http* would not need DNS at all. Put another way, http does not *require* DNS to function. Using DNS can create a One -> Many mapping of IP Addresses to Domain Named websites, allowing a single server to host many discrete websites. (I don't know if http clients do anything *with* the DNS host, except read it to determine which website to respond with. If this is the case, then we could 'abuse' the http spec and use a non-DNS name service [ON] in the 'host' part of the request header.)

A *browser* is **tooling** built around both *DNS* and *http*. A browser uses DNS to find an IP Address listed in the URL and sends the *IP Packet* containing the http request to the *IP Address* listed in the DNS record.

At a technical level, we don't need DNS to use the web. However, creating a registry of all 'active' IP Addresses defines a known search space in which to crawl the web and index everything. Without an ON such as DNS, creating something like Google or DuckDuckGo would be much more difficult. In essence, DNS + http is basically a very narrow sliver of the entire internet, and this tiny sliver (while still very large) is only workable using an ON and not IP directly.

Since domain names are for making IP Addresses easy for humans, most domain names will host something at port 80 (http). However, this is not a requirement. So, while DNS creates a 'short list' of IPs, it does not mean that *every* one of them will have indexable http/html content. Servers can have other services at other ports. DNS is just a convenient way for humans to find servers, and *indirectly* a way to create a short list of IP Addresses to crawl for indexing. 

If we wanted to create a *shorter* list of IPs that only have indexable html content, then we could create a second ON to register a subset of the domain names. In practice, we have developed conventions such as the [robots.txt](https://developers.google.com/search/docs/advanced/robots/intro) file, that improves indexing. So, we can see that there are ways to create sets within a set without needing an ON for everything. However, the more sets within the set, the more complicated the high-level ON will be. (And the more extra resources will be required to crawl the extra-large high-level ON.)

## Summary: 'What is an Overlay Network'
With all of the above discussion in mind, let's attempt to simplify how we define an ON:

**Overlay Networks are a set of members (nodes) who collectively work together to create a unified and complete mapping of symbols to corresponding concepts.**

An ON might be conceptually simple and have its symbols integrated *into* many different tools (DNS: domain name -> IP mapping), or it might be highly specialized, where members perform a very specific set of rules to perform the mapping (DHT: Hash -> Content), or it might not have direct URI-like mappings (Blockchains: RPC Call -> RPC Return Concept)

We can see that ONs are very broad. In theory they can be composed to allow partitioning of lists of IPs by context. 

Any ON may require protocols and tooling within it. Or the ON *itself*, can be used in other protocols, tooling, or even other overlay networks.

# Part Two: What to do with these ideas
## Goals for Dweb + Overlay Networks
### DNS Replacement
I think we need to have a permissionless alternative to lower the cost and friction to becoming a first class internet citizen. Currently DNS is costly to set up, and there is also no way to avoid a yearly premium to keep your name. 

Being permissionless will add some complexity — but making a permissioned replacement will end up requiring some sort of monetary transfer to pay for administration of the system (just use DNS at that point). As noted in other documents in this repo, I think we must allow name collisions as well. This makes it not *exactly* like a DNS replacement. Colliding names can cause some issues, but I think they are really just a shorthand way to discover friends. Once in our browser equivalent (tooling), the user can assign any name they want to their 'friends', much like how we can name our contacts (name -> phone number). Previously I proposed that the PKI system act as the DNS replacement, and I think this still makes the most sense. A First Class Citizen needs to be able to be verified, else what is the point of creating this primitive?

For more details on my other musings on DNS/PKI/Identity see my [section in the architecture doc](https://github.com/ThinkingJoules/distributed-web-data/blob/main/architecture.md#digital-identity) and the follow up section on how it might work in more detail in my [document on sharding](https://github.com/ThinkingJoules/distributed-web-data/blob/main/ideas/sharding.md). My idea on sharding, in the language of this document, is basically creating a DHT-like overlay, but instead of storing hashes, we are storing and validating blockchains. Unfortunately we have the complexity of both DHTs and blockchains combined into a single ON. However, I think this solves our problem and achieves internet scale. (No one ever said reinventing the web would be simple!)

### Primitive Symbols
I [previously discussed](https://github.com/ThinkingJoules/distributed-web-data/blob/main/ideas/primitive-symbols.md) the realm of possibilities for how the creation and discovery of Atomic Data's Property Definitions could work. The spectrum went from: "crawl the entire ~~internet~~ DNS replacement overlay network" to "have a single append-only log of all definitions". This discussion is still ongoing and I think this document on overlay networks strengthens the validity of our understanding of options. They are all flavors of picking an existing ON (DNS), or creating a new one. If we create a new ON, then it is a matter of scope, and how it might fit in with our DNS replacement, and our larger design.

I would like to add a note that even doing an RDF-like web crawling approach to find and index all Definitions (which is permissionless due to the name-spacing) still has a potential spam problem. A user could generate billions of definitions, and upon discovering said Definitions during the crawl-index phase, the indexer would still need an automated way to reduce or limit the injection of state into the indexed set. I think the solution posited [here in the previous document](https://github.com/ThinkingJoules/distributed-web-data/blob/main/ideas/primitive-symbols.md#potential-solution-proof-of-work) is still required.

The problem with trying to find a complete set of anything, is that permissionless-ness invites potential abuse. While a permissioned system creates administrative overhead (along with the potential problem of acting as a gatekeeper to use the system).

We need a complete set for primitives (i.e. Property Definitions), because we need **maximum** reuse of the symbols to create truly interoperable data. The design of the system will create a prevailing way to build within it. We need to make sure everything is structured around unifying, discovering, and reusing the primitive symbols.

## Can we use IPFS?
This is by far the largest and most mature overlay network built with distributed web ideology. I think it is worth discussing.

Because IPFS aims to be generic, it suffers from immense complexity. This requires using a complex client to do something seemingly simple within our system. 

Because our larger system is aiming to be location-based instead of content-addressed (users own their own data, and so must host it themselves), then our core ideology is at odds with the core ideology of how IPFS works. The larger system we are building is working in a different paradigm than IPFS. 

The most likely spot I could see using IPFS in the larger system, is with caching, and distribution of [Public Statements (NRD)](https://github.com/ThinkingJoules/distributed-web-data/blob/main/architecture.md#public-statements-the-hard-part) across the network. However, this is still a very specific problem, and could be solved using a much simpler algorithm. This is more of a distribution/caching problem (like CDNs). Since we can have no expectation of other users saving (pinning) this data, if the author goes offline it still may be unreachable. I think it would be worth exploring other protocols for content distribution and ephemeral caching. I think something closer to [FreeNet](https://en.wikipedia.org/wiki/Freenet#Technical_design) (minus all the privacy and anonymity for our usecase) is more what I'm thinking. The routing seems simple enough that we could reimplement the core of it, but could adapt it for our specific usecase.

Both IPFS and FreeNet are generalized around files. IPLD allows DAG structures for doing more complex hash structures. Since our NRD is currently envisioned as a broadcasted root, we could make this IPLD-compliant. However, let us briefly explore the alternative. Using non-IPLD, we really want to have each hash represent our binary representation of a node for our trie structure that generated the root (as opposed to a CID). The network would store these trie nodes, some of which will contain values. This would allow users to ask this distribution network for hashes that represent these nodes, and walk down the trie to discover/prove state. This is just like IPLD and its Merkle DAG, but we want to do a non-generic Merkle Trie (for performance and non-inclusion proofs). Our system would really only have one 'file type' (or CID): a trie node. Our trie nodes contain more than just a hash of their children — they also contain part of the key, as well as the entire value in-lined. We could adjust the structure of our NRD trie to allow the value to be chunked in-trie. Our hash might have one extra leading byte to allow for versioning and upgrading the trie node structure.

Since our larger system is location-based, the fastest route will always be to ask the author, as they can ship all the nodes in a single roundtrip if needed. However, this would put undue strain on popular authors, and having a distribution network can ease the burden in a simple manner. 

Regardless of distribution, the author still needs to always persist the NRD, as we cannot force long-term storage on more participants.

In summary of IPFS, I think we *could* use it for a portion of our system. However, it would potentially be overkill since it is a general purpose solution. Being *so* general, it could also cause people to abuse our distribution network for non-trie compliant CIDs. This should(?) be detectable by the CID, and so could be dropped. Either way, this is something that would warrant more discussion and exploration on tradeoffs when this system gets to a point of further development.
