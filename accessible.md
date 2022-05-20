# Accessible vs Permissionless vs Distributed
I would like to define and clarify some common terms, how I think about them, and what I feel is nesessary for this new system. 

# Accessible
## For a User
To be 'accessible', I include consideration of broad barriers to participate. I assume someone who wants to participate:
- Has a computer 
- Has access to the internet

Since this is the distributed web, and people own their own data, then they will also be in charge of hosting their own data. This may seem inaccessible for most (setting up and running servers can be hard), but I think if we build a framework in which to operate, one-click setups or no-code server setups are entirely possible. I think a service could even charge for the *convenience* offered. This is okay, because the owner of the data has no lock-in to their provider.

I want to absolutely minimize any *need* to use money directly in this system. Acquiring a token is absolutely out of the question. If it requires a token then I deem it inaccessible.

Is this fair? Requiring someone to host their own data is much more difficult than acquiring a token. I disagree. A token requires the user to sign up on an exchange (if they even have one where the token is available), probably trade into some intermediate currency pair, and then trade into the token for this system. Getting a token listed on an exchange is also a difficult process, and you would need it listed on tons of exchanges to ensure the lowest friction for your users. Compare this to a process of paying a service to host your data. Sign up, pay in your native currency (credit card does auto-conversion) and enter some minimal configuration info. Also, running your own server is possible so there is a *choice*.

Is requiring a domain name (using DNS) accessible? I personally don't think so. It requires recurring payment, and while just as simple as our hypothetical 'one-click' server service, *there is no alternative*.
## For a Developer
In my opinion this comes down to the question of: 

How hard is it for the developer to quickly orient themselves and understand the prevailing method to *use* the system to accomplish a goal. 

Both the RDF and DID specs would be examples of very flexible and highly descriptive, yet disorienting, projects for newcomers. Like both of those projects, this project aims for interoperability. However, to make it easier to reason about, we have to cut the complexity down. To cut the complexity down, we have to operate in a new paradigm (by new paradigm, I mean: not be backwards compatible with old systems). To operate in a new paradigm, requires developers learning many new things. However, it is easier to learn many straightforward things, than it is to learn a single very complex thing. (Of course, it remains a hope that this project ends up 'straightforward'. Atomic Data already managed to simplify RDF in a way I found really useful, so I tend to believe we can make progress in this direction.)

# Permissionless
## DNS
As noted with DNS, you have to pay recurring fees to keep your name registered. That is because you really don't *own* a domain name. You own the *right* to use it. So I think it is against the ethos (in my opinion) of the *ideals* of the distributed web. I think the best we can do here, is to allow creation of some symbols to represent a Domain for a user's namespace. These symbols won't be human readable. We can map human friendly alias(es) to this symbol. We would need to allow collisions of these aliases, otherwise they will become scarse and then cost money/tokens to acquire.
## Data Creation
This falls under the censorship-resistant category. The design of this system is not concerned with aggregating content, merely creating shared semantic symbols with which to label content. Censoring or curating content is for high level systems to handle, the ones who create aggregation indices or feeds.

# A 'Distributed' Web
## Physically Distributed
Owning your own data doesn't mean it can't be in the cloud. I would expect most people to have it hosted in a datacenter. The topology of the internet is not really set up for pure p2p connections. Ideally, our system can allow people great flexibility to determine how or where the data is. For example, someone might have a public node in the cloud, where all requests come in, and then that node actually communicates with a private server on their own premises that actually has all the data to fulfill the request. 
## Logically Distributed
This is the core aspect I care about. Users are the core primitive in this entire system, not applications. A *user* has data. A *query language* allows asking others about their data. A *UI* displays a composition of all this data. ...And, that's it. When data/semantic symbols are widely shared by developers, the concept of an 'app' as we know it disappears.

