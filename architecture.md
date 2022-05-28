
# Architecture Overview
I will start at the data/query level and and then work down to the foundational ideas on how to accomplish said goal.
This is a federated, namespaced system at it's core
First we will focus on data within each users' namespace

# Data Model
I believe the key thing to be solved in order to unsilo data/apps, is a straightforward way to understand how to reuse existing labels in the system. Content addressing and other distributed data schemes have been around for a while now. I think these can be used, but lack a broader framework in which to be leveraged. The bulk of my research and learning has been about ways to represent and model data that can work for any use case, but not be a total pain to work with.

I have settled on an Object Graph approach using a 'Triple' like primitive.

## The Triple
If you decompose SQL tables, you get RDF triples. Each 'cell' is now a row. You lose the structure of tables, but that is a good thing: we need the flexibility. (There are ways to add back rigidity, if emulating a table is needed.) Let's look at triples in more detail.

### RDF Triple
The RDF spec is messy. But I understand how it ended up the way it did. I'm trying to prevent ending up there as well. For those unfamiliar with RDF, it is a triple, because there are...three components:
* Subject - Basically some ID to key up other triples that share this ID.
* Predicate - What semantically are we trying to describe about the Subject? These would have been columns in a table.
* Object - Basically, what is the "Value" for the predicate on this subject.

The "Object" in RDF is always a string. However the string might be another resource (points to another Subject), a blank node (ignore this, we don't care about it for our construction), or it can be a literal. The literal string has an encoding with optional properties to it. For example, you can add language tags: 
```
object = '"some string"@en'
``` 
or datatypes: 
```
object = '"some string"^^<http://www.w3.org/2001/XMLSchema#string>'
```
I believe you can do both at once. I'm not sure if there are other 'properties', but you see my point. So the "object" really is an object in the sense of object-oriented. It just has its own special encoding and syntax for its properties.

We could think of it like so (in pseudo Rust):
```rust
struct Triple{
  subject: Option<URL>, //Option, to allow "Blank Nodes"
  predicate: URL,
  object: Object
}
enum Object{
  Resource(URL),
  Literal(Literal)
}
struct Literal{
  value: String,
  lang_tag: Option<LangTagEnum>,
  data_type: Option<URL>,
}
```
RDF requires secondary things to add Property-Datatype constraints. RDF is built like the internet: many layers. This means there is more that developers have to understand and get up to speed with. This brings us to the impetus of the Atomic Data project: frustration from working with RDF.

### Atomic Data Triple
[Atomic Data](https://docs.atomicdata.dev/atomic-data-overview.html) combined a lot of things in RDF to make it more usable. So let me quickly cover how it works. Basically a "Predicate" is now called a "Property". Properties have a strict datatype at definition, getting rid of the weird encoding of the "Object", and thus "Object" is now called "Value". The PropertyDef itself is simply another Resource (Subject), with certain properties used to describe the Definition of said Property (this is called reification: it's triples all the way down). So something like this:
```rust
struct Triple{
  subject: URL,
  property: URL,
  _property_datatype: PropDef, //'looked up' from the property URL
  value: Value
}
enum Value{
  Anonymous(Vec<(PropertyURL,Self)>), //replacement for Blank Nodes
  Data(String), //'decodable' type of _property_datatype, might be a URL for another resource
}
```
So it is pretty easy to see why this is more usable than RDF. Atomic Data adds language tags through a [Translation Box](https://docs.atomicdata.dev/schema/translations.html), thus preserving all the functionality of RDF in a much cleaner fashion. Atomic data also doesn't use things like 'blank nodes' because they are not globally addressable pieces of data. Instead they use [paths](https://docs.atomicdata.dev/core/paths.html) to resolve nested objects. All existing RDF data can be put into Atomic Data.

I will pause here on the basic idea of a Triple, this is more than enough information to continue for the uninitiated.

## Leveraging the Property
Atomic Data leverages the property to simplify usage of the resulting Value. If these PropDefs are easily found by developers, using a full text search implemented on the semantic description, it can be reasonably assumed that other developers would reuse these. Obviously, the description of what this symbol represents must be clear and easily understood by *all* other developers to get reused. 

There is the potential for confusion about which labels to use, but I believe data labels will converge on useful symbols without explicit coordination (obviously coordination could speed this process up). The reason I believe this is that these labels will function no differently than ordinary language, and in ordinary language, people use words that are useful. Often, the most useful words are the ones already widely known (otherwise you would have to define everything all the time to everyone). So, if one app creates and heavily uses a symbol, other developers, seeing how it is used, will understand the symbol through example, and will be inclined to reuse it to make themselves more widely understood. On the other hand, if someone represents their idea/app/semantics poorly, and it doesn't work well with other apps, then when a good use case demands its integration, data translations/equivalencies can be made (as, again, happens in normal language). 

## Queries = API = Language
If we are getting rid of the notion of an 'app' and simply having 'data' and 'UIs' then there shouldn't be any APIs. I do not know if we need a specific query language per se, but asking questions using a shared set of symbols and syntax seems like the only reasonable way to attempt an endeavour like this. So in order to have query languages, we need some universal symbols. This is what we are trying to do by having PropertyDefs widely known and easily accessible. These symbols then act as tags to use in queries. "Get all Subjects that have X, Y, & Z Properties". 
### Classes
I think there are two ways to get the 'effects' of tables back. We can be prescriptive — adding a class identifier directly to a subject to enforce a rigid schema. Or, we can be *descriptive* — I think of this like Traits in Rust, except function signatures simply become a list of PropDefs. The descriptive usage of a class def will only validate the properties that are contained in the class def (it 'ignores' other properties on the subject). 

To keep things from being Authoritative, I would opt for finding Classes at query time (using them descriptively), and not adding any labels directly on the subjects. This will slow things down on queries, but prevents stale or rigid labels. There is no reason UIs can't use the class definition to ensure validation is met on input, and just skip adding the label directly.
### Indexing
Having a known syntax for the triple could also allow automatic/dynamic indexing. We would need to index on properties, since we will need to 'compose' subjects if we allow classes as shorthand symbols in the query language. Because apps currently specialize their whole stack for performance, we need a way to claw back as much of that performance as possible. Since our data model is very general, I believe we pay for it in creating lots of indices. This will cost more for disk space (and write costs), but we **cannot** sacrifice the flexibility. Indexing is a cheap bandage if it gets us flexibility + fast queries.

## Properties = Root Morpheme
(The next few sections I use terms from this [wikipedia article about Morphemes](https://en.wikipedia.org/wiki/Morpheme))

The property is the prime symbol here. However, I personally feel like it is doing *too much* work. If we want max reuse of the symbol, we need to give it a *bit* more flexibility. (And now I recreate all the pitfalls of RDF...)
## Double Duty Symbols = Bad
To most clearly illustrate my thoughts, let us focus on something that is part of RDF/AD: languages. This is important as the internet is global. But there are lots of things different across the world. I posit that either everything should be internationalized or nothing should. Here is my example on why I think Language Tags (Value Suffixes in RDF) need to be generalized to the broader system:

> Imagine you are a global organization and want to create a survey in an RDF/AD like system. Great, you can translate your questions and make sure your UI loads the correct translation (using the language tags as hints). But what if your survey asks a person their height? The property might have a symbol with a semantic meaning of "height of person" and a value that is... What? Well we know it can be an unsigned integer, since people can't be negative height. But, should it be an integer? Depends on the resolution we want. But we still haven't figured out the **UNIT** of this float/uint. If we use centimeters that will probably suffice in integer form. But what if we want to collect data from the Americans and their weird units of measure? The survey participant would have no intuitive idea how many centimeters tall they are. 

In the Atomic Data model, we could change the properties to be "height of person - cm" and "height of person - inches". How do we easily write a query that asks for Subjects that have "height of person"? If these property symbols are basically like a dictionary definition, adding a unit of measure will end very badly. 

We could revert how we use Atomic Data back to how RDF would work. We would have a singular property again, but the Value would need to be text with the number and units together. This leaves validation and parsing of the unit symbols up to the org. What if want to combine survey data from several organizations and they use different symbols for units? Now the data is no longer clean and interoperable, and any 'Number' datatypes are going to be rendered useless, as you need to add text symbols for units.

Either our Property is doing double duty, or our value is. We need our symbols to do one thing, and one thing well! I'm [not the only one](https://www.nature.com/articles/d41586-022-01233-w) that thinks machine readable units are needed for interoperability!

## Composability is key!
Like language, we need morphemes to do exactly one thing for us. Composing these together allows great expression. For right now, let's strip the idea of a Property Definition down to a literal Definition. So our Root Morpheme is basically a dictionary definition. A *single* dictionary definition for a *single* word. So we need to add some bound morphemes (prefix/suffix) to fully express what we want.

## Intuitive Human Constructs vs Machine Readable Constructs
I think we should use the idea of a suffix again, like RDF, but differently. What things have suffixes? Depends on what context. Almost everything? I think we need to separate concerns of ***WHY*** we might put suffixes on things. 
1. There is a technical syntax to the bits. Are these bits UTF-8, UTF-16, unsigned integer (we would probably fix the endianess of the system), .jpeg, .docx, etc. In short, what subroutine does the computer need to run to handle things correctly?
2. There is a human grammar/syntax to the Unicoded text. What human language is this text in? Indirectly this is *technically* another hint for computers. Program control flow can use this hint to 'pick' which translation to display onscreen, or simply use a ML model (or whatever does text translations nowadays) to translate Source Lang -> Target Lang.
3. There is a unit associated with a concept. In daily life, almost everything we deal with is a number and unit. Time, distance, speed, currency. We don't think about language as a unit, but I think it falls in line with the rest of these. It is simply String + Intuitive Human (Language) Construct instead of Number + Intuitive Human (Scale) Construct. Units are a way for *both* humans and computers to understand how to interpret *numbers* (numbers, the universal language).

Having built some things in software before, these also feel like problems I have needed to deal with in order to accomplish a goal (less with languages, but binary and units very much).

So I posit, that RDF wasn't way off with its suffix in the value, I just think we need to think about how to describe stuff and where to put it. 

## Bound Morphemes
So we have a 'Definition', from above, and it looks like we cannot combine/reduce any of the three suffixes without losing expressibility. So we have three high-order languages: Binary(computers), Text/Unicode(human), Numbers(math). 
### Binary
Since we are working in software, we **must** have a binary morpheme (suffix). We are already very familiar with these. We usually call them file extensions. For this new system, I don't think we should distinguish from 'primitive' data types. I think all number representations, boolean, UTF-8, etc should each get their own suffix. File extension doesn't feel right to use, as it seems like a mismatch for the primitive values, but logically it fits (how do we 'read' these bits). I think the most common types will have tooling built around them to make them easy to use (i.e. in IDEs or UIs).
### Language & Numbers
These bound morphemes may or may not be required in combination with the binary suffix. That would have to be indicated during the Binary Definition.
#### Language
This is basically the language tags that RDF/AD use. A .utf8 binary suffix should probably require it. Text-based binary suffix encodings are hard, since we often represent binary as text. We could always add .hex, .base58, .base64, etc as Language Tags, to meet the requirement that **ALL** .utf8 binaries have a "Language". It is overloading this symbol a bit, but I think it could work. Ideally we store binary as binary, and simply represent it in textual form in a UI. Alternatively you could make a *binary* suffix of .hex and in the spec it could simply state that it is utf-8 encoded. (I would prefer to not overload things, but just thinking through ways the system could be used improperly.)
#### Numbers (Units)
This would be basically the same as our generic Definition, but for the Number Space, not the Descriptor Space. However, I would probably make the symbol namespace different as to not mix or confuse. That would also allow adding more info to describe it more precisely. This Unit Definition is only for humans to understand and ensure the tooling and subroutines built around it are implemented correctly.

## What makes a "Word"?
So to create a 'Property Word' we need a Definition + BinarySuffix, and then depending on the rules of said BinarySuffix, either or neither LanguageSuffix or UnitSuffix. I'm using suffix here, as that is where RDF placed the LanguageTag in the Value, where file extensions reside in file naming conventions, and units are (usually) placed after the number.

So we gained composability but did we make this a pain in the butt to work with?

I would argue, that from a user perspective, adding an additional drop down in a UI to select units before entering a number, would allow users to select the unit they feel they have the best intuitive grasp on (if it is subjective), yielding more accurate data collection. If they know the answer in one unit, but do not know the conversion (for example inches -> cm), this can also prevent any sloppy or incorrect conversions forced upon the user by a mismatched developer-user preference (in the context of a survey, it could simply result in more surveys abandoned). If a user can save their preferences, UI's can even autofill this Units dropdown.

From a developer's perspective, this eliminates any parsing of text (even though we all love Regex...) to separate the double duty usage of the string 'value'. Computers are great at math, so let's just have them do these conversions. Using the UnitSuffix, we can create conversions between many different compatible types (just as we must do for language translations). In our query language, this also adds different types of 'tags' to help filter or find subsets of data (i.e. return only ".jpeg").

## Systems of Measure
Adding Units of measure (UoM) is not fully thought through yet. There are many different systems of measurement. We have at a high level, the difference of Metric vs Customary vs Imperial. And within these and across these we have conversions. This isn't a problem directly, it just adds some complexity to creating conversions across all UoMs. New Units don't get added often, so I don't know that this is really an issue. This is not trivial (as noted in the linked Nature article), but *the complexity exists whether we want to acknowledge it or not!*

Currencies might be the exception, due to the rise of cryptocurrencies and tokens. I imagine that currency as a Unit is mostly a hint for a UI to fetch exchange rates and display appropriately for the user. This is a lot like language tags, where suffixes provide hints for program control flow to integrate with ways of resolving the conversion/translations. I think only fixed conversion units can be built in and have convert-on-the-fly query/UI integrations. Currencies and Languages would be left to middleware that uses the appropriate hints from user preferences and the suffix saved with the data to make any necessary conversions/translations.

### Compound Units
Because Units are used by humans for intuitions of scale, there is another issue that humans struggle with: large numbers.

I'm in the US so I will talk about what I know. We (unfortunately) use Customary. It is common to measure a person's height in Feet + Inches. However, in my previous description I only have a single UnitSuffix. So how do we deal with compound units such as Feet + Inches?

### Complexity is Inevitable
I think this is where we have to compromise. If we can create UnitSuffixes, then we can convert between suffixes that make sense. These conversions will need automated tooling built around them in order to allow auto conversion during queries. (For example "All people > 5 feet tall", would get people with UoM in inches, centimeters [and anything else a conversion subroutine has been written for].) So if we have ways to run subroutines that can handle different conversions then I think we need to push the compound units into the UIs and allow these subroutines to aid in converting down to a single unit for storage in the datasystem. It isn't great, since we want UIs to be as simple as possible to build, but I believe this is where we need to draw the line. Allowing anything more than a single UnitSuffix is going to be hard to deal with on the query side. It forces us to mix both binary suffix and unit suffix together (2 numbers encoded in binary (feet,inches), instead of single 'primitive' binary suffix symbol). In short, we want to avoid adding a dimension of combinatorics to conversion subroutines. We already have combinatorics of representation (binary suffix) with the unit suffix.

## The Goal of Adding Units
If we can build these auto-conversions for (NumberRepr,Unit)->(NumberRepr,Unit) we have gained a way for different apps/UIs to record things in different units without messing up queries or having to conform to a fixed Unit (or representation) from another app/UI that originally created it. We can have App A store a 'height' in .f64.inches and App B store it as .f64.meters and finally App C queries for "heights > 5.u8.feet" and will get data that matches what was entered from both App A and App B.

### Technical Note on Conversions
Let us think of the implications for filetype conversions (*not unit conversions*). So in programming, most commonly with numbers, we can ['cast' between types](https://en.wikipedia.org/wiki/Type_conversion). So can we extend this to files? We convert many types of things to many other types. .jpeg<->.png, .mp4<->.mov, u8<->u16, etc. Some conversions are lossless and some are lossy. This would be something to be aware of when doing conversions. *UNITS* suffer from the same problem. Moving from one UNIT with higher precision to a lower precision type can cause a loss of information. This is technically even true when, for example, converting integer centimeters to decimal meters. Float representation is *often* lossy (if you are working at that level of precision, you probably already know this, and build around it). The information world is a complicated place.

So I see some similarities here. All are convertible things. The complexity and lossiness of the conversion varies widely. .u8->.f64 is pretty trivial. .u8.cm -> .f64.meters is a bit harder. A .docx.en -> .pdf.it is very hard. HOWEVER, **if** the subroutines were written, you could convert from TypeA -> TypeB. So taking all the suffixes into account we can convert one set of bits to something we can reason about into a different set of bits (equivalent/lossy or not). This combination matrix will add in the complexity of creating the auto-conversion tooling to enable universal queries, but **this complexity must be dealt with somewhere!** The alternative is to simply push the complexity on to dApp developers and I think this is unacceptable and will keep us stuck in a logically siloed system.

## Recap of the Data Model
So far we are still using a 'triple' structure, we just made the 'value' portion have a high-level encoding that captures metadata about the user's binary bits. 

So let's keep the rough idea of the triple in our head, but I would like you to think of the triple in terms of keys in a prefix tree. The Key=Subject+Predicate concatenated together. The value for each of these would be the high order encoding we just talked about. If you get all 'keys' below the Subject in this prefix trie it will contain all the properties that this subject has.

# The Nature of Data
If users will all own their data, and we have p2p queries and background data synching/caching we have a few problems.
1. Some data is private, or tightly shared. We need a way to set permissions to filter out private data from queries by unauthorized people.
2. Company Setting, widely shared, but not public data. We don't need the whole world to have a security edit-log, but we want to be able to keep all previous states, that perhaps, only admins can see (you trust these admins not to edit the logs...I guess? Otherwise, in private data, you simply want a log of changes for yourself. Either way, this usecase still exists).
3. Data used in Public Statements (i.e. social media posts, newspaper articles, blog posts, etc.) needs to be...public. We need a way that anyone in the system can quickly detect when someone has deleted/edited a post or, inversely, created a post we can't see (non-Public Public Statement, which could be used to confuse or manipulate perception).

These all require mechanisms for the owner of this data to exert control (or *not exert* control, in the Public case). These are in order of least complicated to most. Let's work through these and think about what we are missing.

## Permissions
This is going to be a hard one to solve simply for user experience. The finer-grained the control, the more effort for a user to set (or understand) their permissions. The implementation of permissions is pretty flexible, in that it really only needs to query a set of four sets. You need to be in the intersection of a (Group [of users]), ([With __] Capability), ([Group of] Subjects), and ([Group of] Properties) in order to be able to perform the action indicated. This gives you 'cell' level granularity (this further adds to the performance overhead of the distributed web — resolving permissions will probably take longer than getting the data...). 

A key difficulty is making rules or adding new Resources to the proper sets. If you had no rules, then it would push the complexity to the App to set the proper permissions on creation of new data (assuming "no-action" is fully private). This would mostly be in a 'Company' Setting. However, a company is probably building a custom UI and so can set the permissions in the UI code (mostly just add new Subjects to the proper group). I think there are many ways of doing permissions, but trying to optimize for user experience and ease of understanding 'who' can 'see' 'what' ('when'? 'how long'?) is difficult. Hierarchical systems are nice for inheritance, but not all data is hierarchical. 

If we assume, for the average user, most of their data is either fully private or fully public, then I am not terribly concerned if we don't have a 'great' solution for the middle ground at the present moment.

## Permissioned Time Log
So what if we want to easily create a log? If we already have the idea of prefix trie, then simply extend the key:
```
Subject/Property/Timestamp = value at timestamp
```
This is now a 'Quad' key. This breaks our 'everything is a triple' but makes it really simple to reconstruct past states. Just need to ask for all keys in the subtree starting with:
```
Subject/Property
```
This is really simple given a prefix trie. The challenge is that now if we ask for all keys below a Subject, we get all past states of all properties that have a log. I think we can work around this, since this 'Quad' isn't *really* part of our actual data model. It *would* need to be part of our permissions model though. Either way, it is just a convenient way to colocate it near the rest of the data. I don't know that this is *exactly* how this might looks, but is how *I am* thinking about the problem. This is similar to the data model used in Apache's [Accumulo Database](https://accumulo.apache.org/docs/2.x/getting-started/design). Note the different 'segments' in their 'Column' portion of the key. Their 'Column' is basically like our 'Property'. So indirectly they are using a triple-like method to decompose the tables within their database architecture. Accumulo also has 'cell' level permissions, so it might be interesting to explore how they describe/model permissions, since our system will also want this same property.

We still need to discuss the encoding of these key chunks/segments, but let's not worry about that just yet.

# Public Statements: The Hard Part
#3 from above is difficult enough to demand its own section. I will call this category: non-repudiable data (NRD).

First let us list the requirements to solve the NRD problem:
1. We need NRD replicated to diverse actors so the author cannot easily convince all replicators to simply delete the log of it ever happening.
2. We must have a mechanism to detect asymmetric dissemination. (Group A must *know* that they cannot see eg a tweet that was created, a tweet that presumably Group B can see.)
3. We need to easily detect updates/edits, or deletions of the NRD.

I think #1 is undeniable, since we no longer have a trusted 3rd party (centralized app). I also think *how* we execute #1 is the hardest part for **accomplishing NRD on the distributed web**. #2 and #3 are related to each other.

All of these are intertwined and I believe blockchains are the only framework in which to reasonably think about the problem.

## Understanding why we **need** a blockchain
I think blockchains are constantly overused. Often the overuse is for reasons that I think our so-far-discussed data model, and its widely understood semantic symbols, could address. But once we end up in a global, permissionless system, I think blockchains are the only way to keep your sanity and reason about a complex system. Let me try to make my case to those (rightfully) skeptical of blockchains.

Let's clarify why we are in this particular design space. You, as a user, probably don't need any proofs if you *trust* your friends and follow what they post. Repudiable data would probably work just fine. This is trusted, but permissionless (your friends can respond however they want to your request).

If there is any way to discover new content (how else would you gain *new* friends to trust?), you will have no trust context on other users and what they post. In this situation, all you really need to do, is acquire signed data from someone *other* than the author (in centralized apps, we treat the app as a singular, shared 'other' party amongst all participants).

But how do you *know* the non-source isn't *also* the source posing as a second person or simply a second actor colluding with the author? (Centralized apps don't have this, given the trusted, singular, and shared 'other' party.) This is still permissionless, but is still trusted, just in a different manner. You have to trust that they are not somehow colluding with the author to somehow deceive you. (Similarly, in centralized apps, we do *trust* that they treat all users equally. We *trust* that certain people are not given the ability to edit their tweets, while most others cannot.)

You could sample users randomly and see what they all say, but what if you find two (or more) *conflicting* states? You still need a way to resolve the conflict, *and know that everyone else will eventually resolve it the same way*. You cannot use trust graphs, else everyone will disagree about who said what, which should be provable if we do this right. There is only *one* true history: we need a way to trustlessly gain knowledge of said history.

This is really all blockchains and their consensus mechanisms do. They solve this problem in a known, testable, and quantifiable way. 

I believe the *key* reason we say blockchains instead of append only logs or databases, is that we need these histories observed by *enough 3rd parties, that collusion is so unlikely, we can believe whatever that group of 3rd parties say the history was*. In other words, trustless consensus of a series of events. 'Trustless' is not quite accurate; rather, blockchains are a highly modeled trust environment, where attacks and failure modes are known ahead of time and can be reasoned about. We know *how* someone could break the trust model, and therefore we can reason about the difficulty. If the difficulty = impossible, then it is truly trustless. If it is only 0.00000000000000001% chance, then it is not trustless, but we call it that anyway.

Having a diverse set of 3rd parties works in both directions. We also gain *permissionless addition* of state. I think this is foundationally what we call blockchains. **How** we keep this system stable and defend it from attack is an aspect *of* blockchains. Which is why I would argue that you can have a blockchain without a token. When I talk about blockchains in this document from here on out, I am meaning tokenless blockchains as described in [more detail here](https://github.com/ThinkingJoules/distributed-web-data/blob/main/tokenless-blockchain.md).

## Blockchains and NRD
We don't need (or want) the actual content of the NRD to be on chain. A hash can suffice. If you store all of your own NRD then as long as everyone can identify who is publishing the hash on-chain, they can go ask that author for the data that matches the published hash.

This seems nice. We impose minimal requirements unto chain validators (on-chain state growth is minimized), while offloading the most expensive part (fielding queries, persisting data) to the owner of the data. At worst, an author might lose all their content (server failure) and no one will be able to retrieve the hash that is stored in the blockchain. This is fine, as the blockchain can still be verified without the content. The loss of content is not ideal, but there are a myriad of ways to shard/replicate/preserve your own files. Replication is flexible and a solvable problem that can be handled in implementation. The point is, we can have a pretty lightweight blockchain.

When can we **not** detach the state from the blockchain? Only when we need to enforce some sort of rules on valid state transitions. This is why anything to do with token transfers **must** be directly in the block: to validate the state transitions. Basically, by inlining the data and making it part of the cryptographic structure of the chain, we avoid the fragility of broken links. (With a token, not being able to validate a state transition would ripple down the chain and invalidate all future transactions connected to the broken link. This would not be a well engineered system!)

*So the storage requirement of a chain grows with the state required to validate the rules within it*.

This is true in one meaning, but the growth of the state also depends on how often state gets added. 

How quickly will the state of the chain grow? I think there are three ways to put things on the blockchain for our usecase:
1. Every state transition is recorded. Think of this simply as signing some data. In our triple model this might be like signing: *Subject+Property+Hash*. The hash, for example, could represent the state of the 'Value' portion of the triple.
2. A user signs the *root* hash of their merklized patricia trie (or whatever construction we pick). This would **not** allow network participants to easily know *which* Subject+Property you have updated, just that you updated one or more.
3. You sign the root as in 2, but then submit this information to an aggregator who puts *your* root in to a merklized tree and submits a *single root* (containing many different users' roots) to the chain. This would **not** allow network participants to easily know *which* users have published new state, or *what state* has changed for the users that did submit a signed root.

## Discoverability of New/Altered NRD States
Finding new or altered states using method #1 from above is quite trivial. Just read through the chain, and find the latest state of whatever Subject you have questions on. This of course comes at the expense of the chain growing at a pace of M (users) updating N pieces of data. M\*N is not the whole story, as the overhead of the signature and other associate data is repeated. Computationally, this involves many more signatures for validators to verify (really the *only* 'state transition' that validators have to check) in order to do their job. If we are working at a scale of all types of social media, blog posts, news articles, etc. **combined**, it seems unlikely that any single blockchain could handle this.

The issue with approach #2 and #3 is that it is not clearly evident when something changes. In other words, the difference is that #1 is a log of *all* changes, #2 and #3 are snapshots, with *potentially many intermediate changes*. These 'unofficial' states can be shared but never proven (from an on-chain root). 

Other dApps have been built using method #3. They skip over #2, because generally they put this root into a token-based blockchain (Bitcoin, Ethereum) and they have to pay for the txn fee themselves. Most systems rely on donations and/or some benefactor to pay these fees to keep users from needing to acquire a token or do some weird high-friction micro-transaction system. These systems almost all use IPFS' Linked Data system (IPLD) to build cryptographic DAGs. This allows anyone with the root (on-chain), to ask the IPFS network for the root node, which contains information about the next set of nodes, etc. This allows anyone to walk the DAG and discover new or updated state from the previous snapshot. So performance of the blockchain is no longer the limiting factor, instead it is aggregation on the intake side, and DAG traversal on the consumption side. I have not benchmarked the average fetch speed of IPFS, but I often hear reports of the performance being less than optimal. If this is true, then I suspect this is probably the largest bottleneck of a system in this style. Since *who* updated is opaque, you would need to be walking the top portion of the DAG on every update to determine if anyone you are following has posted updated/new state.

Of course, you don't have to traverse any of these DAGs, and simply query people, and have them provide you with a series of DAG roots where the hash is published, to verify what they send you is 'official'. The downside, is that they could purposefully not tell you about roots that have updates that are more recent, to mislead you.

Approach #2 would basically start with blockchain consensus. Now a user has to build their own DAG, and pay fees directly to the network for publishing state changes. On the reading side, you can simply watch for a specific user publishing new state, and only walk their DAG to find state you care about. This is far easier to reason about, and will be faster. This saves us from walking the 'top' of the DAG. This would require a blockchain to scale far enough to allow M (users) to post updates. This is probably going to exceed the limit of what a single chain can do.

The key commonality between #2 & #3 is that the speed of the system is all about how quickly you can discover data contained in the root.

If we could make system #2 work at the blockchain level, then in theory, you could avoid using IPLD and simply ask users for a merkle proof *to their latest published root*. This allows for faster resolution, as a merkle proof can prove multiple pieces of data in a single proof. Another advantage is that it would cut down on total round trips compared to traversing an IPLD DAG one node at a time. You ask a user for some query, and they return the data along with a merkle proof to the latest root, which you can verify through the network, and then simply verify the proof is correct.

The nature of this merklized structure would need to be determined and optimized for this usecase.

In short, proof generation should *always* take more effort (compute time) to produce, than to verify, and both need to be as fast as possible (verification is most important) in order to keep UX as close to centralized apps as possible. I suspect that walking a DAG is going to be much slower than user-provided proofs.

On the other hand, the advantage of using the traversable DAG system (approach #3), is that it is easier to crawl the network to find state and aggregate or index it for building feeds. With my modified approach #2, you need to contact all users and ask them about information whenever they post a new root. However, I think you could argue that the only thing IPFS is doing for us here, is dealing with all these connections in the background through their DHT mechanism and providing whatever caching they have built in. Remember, the user/author is probably going to be the only one to actually pin their state. I don't really see a need to use IPFS if we already know *where* the data is. We can use hash based naming, and just skip the content routing.

However, IPFS might be helpful as a *backup* resolution method. If the author's server is offline, but say, has a friend replicating their most recently rooted public trie, then you could simply ask IPFS for said hash, and walk down the trie. This can allow your server to have spotty uptime, but maintain some accessibility (even if it is a bit slower). The problem with this method is that it is really only helpful if your friend(s) will pin *at minimum* your most recent public trie nodes. A full replication would require *all* trie nodes for *all* published roots. Obviously, most people ask about the most recent state, so that could reduce the burden on your friends server. If you can't afford any replication (only sharding), then if any one of your friends with a shard of your data is offline at the same time you go offline, then this *portion* of your data is offline. This is an improvement at least. We don't want to reveal our 'friends' in our DNS-like resolution layer, so content addressing provides a bit of anonymizing for us. So this could be a good place for IPFS within our system. I just don't want to require the root hash to be a CID. I would want it to be a vanilla 32b hash that the protocol understands, can 'build' a proper CID for the backup resolution method. That way CIDs and IPFS are not a *core dependency*. Since this is all public data, we don't need to worry about permissions here. (This also means that private data will not have this backup method for retrieval. If your server goes offline, private data is completely offline. Sharding and replicating private data is very complicated due to privacy and permission enforcement reasons.)

### How to get this to scale?
If we continue with the idea that approach #2 with user provided proofs is the most flexible and performant, then we need to make sure the blockchain side of it can scale up. Otherwise our only option is #3.

If we need to cover *current* social media volume we are looking at something like 25,000 transactions per second (on average) for *only Facebook, Instagram, and Twitter*. This is well outside of any single chain capability. For example, the most scalable and decentralizable (in my opinion) consensus algorithm is Avalanche's Snow Consensus protocol. Their whitepaper says 4000 tps with signature verification, and 20,000 tps on just pure consensus. I don't think they have gotten anywhere near 4000 tps in production. What is more interesting is just how expensive signature verification is! This is where centralized social media sites get it easy. They don't need to auth signatures on every piece of data! I don't know how much slower quantum safe signatures are, but I would assume it isn't going to help this situation!

Honestly, this doesn't paint a very good picture for NRD in a distributed setting. I don't think it is going to be easy, and to get enough diverse participants, we will need *lots* of users to contribute to running the network. The more diverse actors that are doing validation, the more we can scale up by sharding. In a sense, this system could work if people cared enough to want to opt out of the low-friction, high-performance centralized apps. Do I think this will happen? Friction can be overcome with willpower and network effect pressures. Performance can only be engineered. What are our options to engineer performance?

The 25k tps seems like a big number, but it only averages to about 0.000009 tps per user (assuming 3 Billion users). Given that information, each chain could be a single user. To scale horizontally, we can create shards that are an aggregation of many of these individual user chains.

Now we just need enough validators validating each shard to satisfy our 'diverse' validator requirement. But we also need enough *honest* validators per shard to fend off bad actors.

Let's start from the premise that some percentage of users will run validation nodes to help create more shards. If we say there are about 3 Billion Users and 0.01% of them run nodes. This gives us 300,000 unique actors. We have no idea how many might be colluding or are going to attack the network. For now let's just go with 300 validators per shard. This gives us 1,000 shards. This would require each shard to have 25 tps for the aggregate network to reach our needed throughput. This seems pretty reasonable. In production, Avalanche has hit the mid-teens tps with no issues. So this doesn't seem impossible. We just need to figure out how to dynamically scale the shard count up and down with number of validators. If we can manage horizontal scaling, then if users complain the system is slow, the only response should be that more of them should contribute to the network.

I know sharding is generally not good for blockchains and has a lot of caveats, but since we don't have a token, I think we have far more options. Reading through [this](https://medium.com/nearprotocol/unsolved-problems-in-blockchain-sharding-2327d6517f43) article makes sharding in our paradigm seem much more possible. This is another reason not to incorporate a token or any sort of double-spend-like capabilities to our system. I don't know the specific sharding construction to use, but I believe this is possible.

### Snapshots or All Transitions?
If we assume for the moment that we can scale by sharding highly enough, what is the proper level of state transition granularity?

So far we have gone with the snapshot idea (specifically approach #2). But if we could get throughput by sharding, should we require more throughput? Should we put *all* of a user's NRD state transitions on-chain? As noted above, it does have a couple benefits.

Here is why I think we can only do snapshots: If the data in this system is well understood semantically, and users can create any widely understood objects, and we record all state transitions (cannot 'double spend' a state), then anyone could create something that resembles a token. It may not be optimized like a true token-exchanging blockchain, but technically you could 'trade' tokens by way of well understood data. This is a pretty neat property, but I don't know that we want to allow tokens within this system. This can create extractable value, and invite attacks to the network. So if we want to keep this tokenless, then we **cannot** capture *every* state transition, as it enables abuse of the system. Fun thought experiment though! 

Exchanging of tokens is something that should happen in a paradigm optimized for it. Just as I am proposing non-token data have its own optimized paradigm (to allow moving away from approach #3).

So if we **must** do snapshots, then I propose we focus on making verification and discovery very fast. We want to make obtaining the effect of approach #1 as easy as possible.

# Digital Identity
As discussed in the summary, we need a Long Lived Identifier (LLI) to represent a user/author to allow more than one key and key rotations. This PKI system would absolutely need to be on a blockchain, as it is the foundation to knowing if signatures are correct and verifying authorship. I think we could use the tokenless blockchain architecture again. This would allow new users to acquire an LLI without needing a token. Being tokenless is vital to lowering friction and thus increasing user adoption. The LLI is the first step to entering this new data paradigm. Here are the roles that the LLI fulfills:
- Used as a (static) namespace (Domain) for the users data identifiers.
- Can associate (collide-able) human-friendly alias(es)
- Used to find server endpoints for contacting the users' server directly (IP Address or otherwise, DNS replacement)
- The keys associated with the LLI will allow authoring of data with multiple keys.

The LLI is a very important piece of how I envision a system like this being built. The LLI is used to do many things, but at it's core it is simply the first transaction on the blockchain the adds the first key. However we [represent that point](https://github.com/ThinkingJoules/distributed-web-data/issues/2), that is the LLI.

Some systems use Decentralized IDentifiers (DIDs). To me, it feels like RDF. It uses many of the same terms as RDF, and I think is existing in a paradigm of the old web. Reading the spec, it is not clear *how* to use it to accomplish a goal, much like RDF. I think that is why adoption has mostly been undertaken by larger organizations.

My view on distributed data is that everything derives it's integrity *from the digital signatures*. In other words, *everything is downstream of the PKI*. I believe if we have a solid way to handle PKI, we can then build anything we want on top of it. Like Atomic Data did for RDF, I would like to do for DIDs. Simplify it so it can be quickly and easily understood by developers. This PKI chain can be used for any thing or any system. I think it is a primitive that is wholly independent from our data model idea. *I* am choosing to incorporate it into the data model we discussed earlier.

To avoid talking about specific implementation details, I will roughly summarize my vision:

There is a dedicated chain to handle PKI updates. Because this can be tokenless, we can shard this to help scale it up. We cannot detach the state transitions or use the snapshot method, since we must always validate all keys and signatures to ensure no violations occur.

# LLI and our Triple
Heading back to our triple. RDF uses URIs as subject and property identifiers. Atomic Data uses http URLs, as they *should* dereference to the Resource. I like the alteration Atomic Data uses. It allows identifiers to effectively be hyperlinks to learn more about the resource. This really leverages the graph like nature of the *web* on top of the graph like structure of the triples. I think we want to carry this through to the new system. The biggest difference is that I don't want to build on top of DNS. So our URL might still be http, but we will resolve it through a different mechanism. Since we are using the same basic topology as DNS (resolves to a particular, physical server) I think we can leave the scheme portion alone. This can allow http(s), ws(s), etc transports intact. However, the hostname and top level domain, will need some adjustment. Since I like prefixes, I think we should build this new URL in a way that will play nice with our idea of a prefix trie. Like Atomic Data, we could require http as a standard URL transport.

## A new TLD, A new WorldWideWeb
If we work in prefixes, then we want to work from top level down, opposite of how domain names work. Logically domain names are com.domainname.subdomain.www. Funnily enough, 'www' is actually [a subdomain](https://www.nexcess.net/blog/is-there-any-reason-to-use-www-in-your-domain/#:~:text=What%20exactly%20is%20the%20%E2%80%9Cwww,Internet%20like%20Gopher%20or%20FTP.) that exists to indicate where the the domain serves webpages from. I never knew that until trying to understand URLs in the context of this project. 
### A New Start
To help differentiate DNS URLs from others, I think we should start the URL with a fixed string. For now let's just use 'dweb'. So we have:
```
http://dweb
```
If these are going to be our keys (Subject+Property) then having http:// is already 14 bytes (7\*2) of overhead that carries very little information. Tim Berners-Lee [regrets this syntax](https://stackoverflow.com/questions/36433409/why-does-http-contain-two-slashes-and-file-three-in-a-browser-navigation) so I think we can adopt his preferred style with no '.' and all '/'. Changing from '.' to '/' style will also help visually differentiate these from DNS URLs. 

### Chain Registration
If we use the idea of chain coordinates and designate some sort of 'Registry Chain' (as the registrar of all 'dweb' chains), then the TLD would effectively be the chain coordinates of the creation transaction. Since this would be a low velocity chain, I would probably make it 1 block = 1 txn, to allow the chain coordinates to be a single number (0 index offset from genesis block of registrar chain). This registrar chain would probably allow a chain to register an alias as well. Since this is *super* low level to the system, we might want to actually *reject* colliding aliases as part of validation on new transactions. This could make things more human friendly. For the most space efficient identifiers, we would want to use the number (at least when persisting it on disk) to save on space. We could self-create the 'dweb' chain in the genesis block. This would have coordinate of '0' with and alias 'dweb'. Now we would have something like:
```
http://0 = http://dweb
```
So for now, let us assume these new TLD's will have unique aliases, but the chain coordinate will be used in any storage or (cryptographic) verification setting.
### User Namespace (LLI)
Since the LLI and PKI are needed for authorship let's create that chain next. This would be the second txn in the registrar chain (idx=1,alias=id). Since we will need throughput to handle global PKI updates and I'm not sure how sharding would work, let us for now assume that this chain will have blocks and txns (so two dimensional) and will only be used as a registrar chain for an initial key pair. There is no key rotations on this chain. Logically each users PKI is a signature chain, I believe we should be able to handle all the key rotations in a separate, sharded, chain system that can reference the LLI (and thus the original key pair) from the registrar chain. I haven't thought through if this would indeed work, but for now, let us just go with the idea that the LLI is a 2D chain coordinate (Block#,Txn in Block). So for some identity created in the 24th block and the 6th txn within the block we might have something like this:
```
http://dweb/id/23/5 = http://0/1/23/5
```
This is basically our new 'full' domain. This is what should resolve to this particular user's public endpoint. The 'dweb/id/' is basically the 'TLD', and the '23/5' is the 'Domain'. The two uints is basically this users identifier, and what we will be using to build an IP Address resolver and associate Human-Friendly-Alias(es) to.
### Semantic Definitions
The next most important thing we need, is a way to describe the Semantic (Properties) Definitions. This would be a 1D coordinate chain (idx=2,alias=def). So the third Definition added would look something like:
```
http://dweb/def/2 = http://0/2/2
```
### Other Morphemes
We would need a couple other chains based on our early discussion:
- Binary Suffix: 1D, idx=3, alias=fx
  - The 32nd fx would be: http://dweb/fx/31 = http://0/3/31
- Language Tags: 1D, idx=4, alias=ltag
  - The 1st ltag would be: http://dweb/ltag/0 = http://0/4/0
- Unit Definitions: 1D, idx=5, alias=unit
  - The 8th unit would be: http://dweb/unit/7 = http://0/5/7

### Property Word Def
We would need another chain to describe the actual full combination that make a 'Word' to define the 'Property' (idx=6, alias=prop). An example:
```
http://dweb/prop/33 = http://0/6/33
```

### Subject-Property
Now we have enough symbols, and a permissionless and universal way to find and allow anyone to add to these (I expect the PoW txn fee to be *very* high for some of these). The 'Subject' is basically our LLI from before, we just need to add another '/'. What comes after this? Another unsigned integer? Do we allow characters? If we are allowing offline creations of data, then it may need to be random (to avoid collision on sync), but that can happen with either numbers or alpha-numeric. This can be discussed some more, for now let's keep using uints.
```
Subject = http://dweb/id/23/5/1234 = http://0/1/23/5/1234
Property = http://dweb/prop/33 = http://0/6/33
```
If we merklize our PATRICIA trie for generating proofs, then we need to fix the conversion of these URLs to a known binary equivalent. If we bake the 'http://' in to the protocol, then we can elide this in our trie. If everything is unsigned integers, then we could encode those as [u-var-int](https://go.dev/src/encoding/binary/varint.go)s and drop the need for the '/' delimiter and allow a fully binary key. This would turn our prefix trie key in to something like this:
```
HumanForm = http://dweb/id/23/5/1234 + http://dweb/prop/33
BinaryForm = 5 u-var-ints (subject) + 3 u-var-ints (property) = 8 u-var-ints concatenated together.
```
Since there shouldn't be billions of Property Words, this portion of the key should use less than 6 bytes, with the most common (earliest created) using only 4 bytes. The subject will have 2 bytes of namespace overhead, then probably 3-6 bytes for the LLI portion, and then potentially another 4-8 bytes (depending on how the actual 'subject' id itself is handled). The actual key length will vary, but I would expect to have the binary key (the 8 u-var-ints) ranging in length from 12 bytes to 18 bytes. That is a lot of information in something that is less length than a single hash! If we are doing internet scale data, then we should be concerned about our data footprint!

I think this illustrates another benefit for using blockchains to create hash-like, one to one mappings. We gain serialization through blockchains and we can leverage that property to get savings elsewhere. It's basically an Array (not sure you could do more than 2 dimensions, otherwise it feels like you are into some sort of DAG? However this could be interesting in the context of our sharding discussion...). Our chain coordinates are then like memory pointers to offsets. Which is really exactly how we are using them. We are referencing the 'data' at said coordinates. As in programming, we, as humans, *understand* what this data is suppose to mean/represent.

## Proofs for N/RD
So I think I should explain a bit about why I keep talking about a merklized PATRICIA trie instead of a vanilla merkle tree. The reason for using a trie, is that it builds deterministically (a regular merkle tree, insertion order changes the resulting hash). This deterministic property allows the proof to also prove *non-inclusion*. In short, we can prove we *do not* have a key since we cannot generate a proof to 'hide' such a key (else determinism has failed us!). 

How is helpful to us? I *believe* this is important to solving the NRD problem. Remember we are only going to put our trie root on chain (32b). If we know what value we want to prove (Subject+Property) then in theory we could ask for a proof to the most recently published root hash. If we wanted to see if they changed the value, we could go back to a previously published root and ask for them to prove it there as well. Since we have non-inclusion proofs, we could *know* if it was not created yet. If the value is different, then they clearly edited it. If they cannot provide the proof (they lost merklized PATRICIA trie nodes associated with that snapshot) then we must be wary that they are trying to hide an edit. We can't know what it was, but we can be suspicious of this person (again, we may not 'know' this person). In a sense, being able to build proofs to any previously published root is a sign of trustworthiness. Yes, people can lose data. These look the same. This is the downside by detaching state from the blockchain. The only reason this works when we *don't* detach state, is that the state is replicated by *every* participant. Do we want to *require* everyone to store all past merkle states for *all* people? No. So this is where some lesser amount of replication falls to the owner of the data. Either people *own* their data and all the responsibilities. This is where making the configuration and setup of running your own server(s) as simple and easy as possible.

The nature of this merklized PATRICIA trie is a Copy-on-Write scheme. So the total state grows by two mechanisms. First, the more changes between commits, the more new intermediate nodes are created. Second, the more often you commit, the more state you create. For example if you make 1 edit per commit and each of the edits has a trie depth of 10. Then each commit will generate 10 new trie nodes. If you are trying to minimize disk usage, you should try to commit (at least publish) as few commits as possible. However, if you don't want latency in publishing your data then you will commit often with only a couple of changes in each commit. This is the tradeoff. 

If we put *only* NRD data in a trie, then all the intermediate trie states *need* to be kept, and acts *as if* it were blockchain data. If we put NRD & repudiable data (RD) in the same trie, then all the edits of our RD will add to the state we need to keep for the NRD. At a technical implementation level, I think it would be good to separate these tries. It would allow us to persist the NRD trie on disk as an append-only file (for crash consistency) and have the trie act as its own database. If every commit is published, then the entirety of this file would be your NRD.

We have two options for RD: we could reuse the single signature + proof (like we do for NRD) OR we can sign every state (like GunDB does). I think we will have [quantum computers](https://www.techradar.com/sg/news/ibm-says-it-will-have-thousands-of-quantum-computers-for-sale-by-2025) that [can break EC cryptography](https://bitcoin.stackexchange.com/questions/57965/are-schnorr-signatures-quantum-computer-resistant#:~:text=A%20general%2Dpurpose%20QC%20with%20several%201000s%20of%20qbits%20would%20be%20needed) sooner than later. Using the single signature would ensure a negligible performance hit. This would mean we could reuse all the proof generation/verification code. If we are building a *new* system that is underpinned *entirely* by digital signatures, we better not paint ourselves into a future corner. Since we don't need intermediate states saved for the RD, then we could explore other options for persistence.

## Value
So we need to work on the Value portion of the triple some more. We still need to address:
- Suffixes: Binary, Lang, Unit
- Anonymous Nodes (can we do the 'path' like Atomic Data?)
- CRDT Metadata
- Metadata
  - Last Edited By, Last Edited Time, etc.

There is a lot going on with the Value! If we want to be able to create proofs we also need to be careful how we encode all of this so we capture the parts of it that need to be provable, vs what can we edit without changing the 'hash'.

### CRDTs for Repudiable Data
I know very little about CRDTs, but I have used GunDB (CRDT based) and it works really well and is performant. I don't know how much overhead it has, or if it can be improved. I think CRDTs will be used on all data, even the NRD. It is so simple to merge data from multiple sources. I think of it a bit like a funnel that would helps get data into either of the tries.

I think Gun has 2 pieces of data to its CRDT Metadata, one of which is a timestamp, not sure on what the other one is exactly. For our purposes, we are using the CRDT as a mechanism at a network level. Because of this, there is no expectation to *prove* this data in any verified manner. 

However, if we want a 'Last Edited Time' and we want that provable, then our CRDT Metadata, might *not* be all the metadata for the CRDT algorithm. Since we are using the CRDT at a network level, and not really part of the data itself, then we just need to know where to get the timestamp from within the *value* metadata. Not CRDTs use timestamps, but I'm just commenting on the bits I know. When building stuff with GunDB I often use the CRDT metadata to display a 'Last Edited Time'.

(GunDB uses a trick with its CRDTs to keep people honest with their timestamps: It doesn't trigger the application of the data until your local time is > than the CRDT timestamp. If everyone does this, then people should only ever want to publish timestamps that are as recent as the average expected clock time on the network. If you publish something in the future, no one will see it (or it might be dropped by the network if too far) until they think it is the time you indicated. The game theory basically works out to, keep your local clock accurate and be honest.)

### Anonymous Nodes
Since we are trying to generate cryptographic proofs of data we must serialize all parts of our 'Value' that we want verifiable. Anonymous nodes present a problem, as it is basically a serialization. For this reason, I *believe* they would become a well known binary suffix. **However**, this definition is recursive. We can nest these nodes more than a single depth. This would mean the binary suffix would have to serialize *its* value in the same manner as our protocol 'Value'. I suppose this would work. But do we lose addressability? This feels a bit related any of the rest of the bits of the Value. How do we address those?

### What is in a Value
Our value is basically, itself, an anonymous node. We have several properties:
- Binary Blob 
- Binary Suffix (Resource)
- Lang Tag (Resource, Sometimes Optional)
- Unit (Resource, Sometimes Optional)
- Last Edited By (LLI)
- Last Edited Time (unix timestamp?)
- CRDT Data (whatever that might look like)

I think we need to find a way to make each of these things addressable. RDF does not allow addressability of blank nodes, and I don't think their 'datatype' or 'LangTag' are directly addressable. We are kind of trying to do both here.

Atomic Data uses an inline map, using the Property (resource) as a key.

Before we had this as our 'key':
```
HumanForm = http://dweb/id/23/5/1234 + http://dweb/prop/33
BinaryForm = 5 u-var-ints (subject) + 3 u-var-ints (property) = 8 u-var-ints concatenated together.
```
If we swap the '+' for ' ' in the HumanForm, it sort of looks like AD's path. It is 'Subject' + 1 or more 'Properties'. Obviously in our prefix trie, we always have exactly Subject+Property. The anonymous node is technically part of the 'Value' and so we lose out on any 'auto' indexing or querying we might build in at the high level. This would indicate that we shouldn't make them anonymous, and instead just point to another resource, making it a graph. To me, it feels like, we either make binary suffixes introspectable and addressable, or we make them opaque to the 'high level' system. Making them opaque would turn the data structure much more heavily in to graph territory. There are many options for graph query language that we could adapt. Remember, we can still use a basic http query for non-complex queries. Perhaps we could compromise a bit, and only allow a binary suffix to be arrayed with itself (more on this idea later)?

If we don't allow generic anonymous nodes, then our 'Value' becomes a special case and can now be handled by having a known convention within the protocol. I think this is needed, as there are other special things I haven't talked about yet...

### "Files" vs "Primitives"
We need to think about our proofs again (damn the distributed web for making things complicated!), and how our actual, raw, binary data relates. The 'Binary Blob' might be a multi gigabyte movie, or it might be a single u8. Due to the nature of our proof trie we have discussed, everything on the trie node must be given in order to prove the state. Do we want to ship our entire multi-gigabyte movie so they can verify it? Absolutely not. I propose that we build in to the system, a way to handle both small and large values. In short, 'large' values *must* be hashed. The naive split is anything less than 32 bytes can be in-lined directly, and anything longer will be hashed. This leaves us with the issue of still needing to ship the *entire* movie to verify this hash. Luckily for us, some really smart people made the [Blake3](https://github.com/BLAKE3-team/BLAKE3) hashing algo, and it is tree based — therefore it can do [verified streaming](https://github.com/oconnor663/bao). For our system, that means we can stream values longer than 32 bytes in a verifiable manner, in 1024 byte chunks. Building this into our system makes it work for *any* binary suffix. So if a .utf8 is only a couple of words, it would get inlined. But if it were novel length, it would be hashed and stream-able. In theory we could do the chunking in our own trie, however, the verified streaming would be our own construction and the top level hash would incorporate our trie node encoding structure. If we want to be able to use the Blake3 hash outside of our system then either we use the Blake3 verified streaming OR other systems would need to adopt our trie encoding to verify chunks. Since this is a file and the chunk order *is* important, then our trie construction is actually a detriment (a regular merkle tree is better for this usecase). This is something that would need to be discussed and settled. For now, I just assume the use of Blake3 and it's verifiable streaming. 

Either way, this line of thinking leads nicely into how to prove other pieces of the Value

### Extended Trie Key
This will follow a bit like our time log idea (and technically invalidate the usage of it), except we will be working with the individual components of the Value. We could address the components as (in HumanForm):
```
http://dweb/id/23/5/1234 + http://dweb/prop/2 + data = binaryBlob || hash
http://dweb/id/23/5/1234 + http://dweb/prop/2 + crdt = crdtMetadataBlob
http://dweb/id/23/5/1234 + http://dweb/prop/2 + lastEditedBy = LLI
http://dweb/id/23/5/1234 + http://dweb/prop/2 + lastEditedTime = unixTS
http://dweb/id/23/5/1234 + http://dweb/prop/2 + http://dweb/fx/ = 31 (would logically be: http://dweb/fx/31)
http://dweb/id/23/5/1234 + http://dweb/prop/2 + http://dweb/ltag/ = 0
http://dweb/id/23/5/1234 + http://dweb/prop/2 + http://dweb/unit/ = 7

```
In theory we could define them all as properties, but the suffix tags already have a known name space so we could use those directly instead of a layer of indirection. If this structure is going to be fixed per the protocol, do we gain anything by making these 'Properties'? Just using '/data' would technically work. This will need to be hard coded in to the client anyway, and all of these are quite self-explanatory. However, 'data' is 4 bytes, whereas 'http://dweb/prop/9' would only be 3 when encoded. So, perhaps just using a property def is best: it is well formed, well understood, and can be directly dereferenced for documentation sake.

To allow an upgrade path for changing the shape of the Value, we could add a version segment to the key. (since the data will per persisted and encountered by 'new' client software)
```
http://dweb/id/23/5/1234 + http://dweb/prop/33 + valueVersionInt + [per the valueVersion] = binary data
```
Adding the valueVersion is a bit of a mess, but would be needed as a hint for program control flow within the client software. If you were to edit an (old) triple you would need to move all data to the 'new version' subtree, and delete the old subtree. This would be a sort of migrate-on-write pattern to allow upgrades. The 'components' of the Value would be determined based on the 'valueVersion'. I don't expect this to upgrade (ever?) but in the event it is needed, this would be a very helpful hook. Another advantage of adding a segment to the key, is that we could encode it as a signed var-int, allowing all positive numbers be 'Value' versions, and negative numbers be used for other subtrees. For example we could use a value of '-1' to indicate that the next segment represents a unix timestamp, to regain our private data log functionality. A value >= 0 would mean the next segment represents some structured set of data representing the 'Value' of our triple.

Our data model is conceptually still a 'Triple', but like RDF we have a complex Value. For purposes of provability+addressability in the distributed web, our 'key' for the trie has 4 'segments': Subject + Property + valueVersion + valueComponent. We could technically consider the valueVersion as part of the encoding of the value, however, we put it as part of the 'key' to branch our trie properly. In effect then, we have really versioned the *Triple* using a suffix at the end of our Subject + Property. Is our triple now a Quad? Not really, since this is *versioning* the triple. But yes, a key now has 3 components plus a Value, for a total of 4 things.

How does this make us feel? I guess it works. We give ourselves an out by adding the valueVersion. We sort of end up using Atomic Data's 'Path' concept, but we use it to navigate a Triple (that does not allow anonymous nodes). I *believe* this could be AD compliant? It would require *every* value be an anonymous node with the 'Properties' listed above.

In our 'binary' form our key would be made up of several (u/i)-var-ints:
```
Subject(5) + Property(3) + valueVersion(1) + valueComponent(2 or 3) = 11-12 var-ints
```
We end up with 3 bytes of overhead for our 'dweb' namespace being repeated. I think this is fine. Perhaps someone will want to built their own registrar chain (namespace) for a different usecase. We would want to allow merging of the systems into a single 'distributed web'. Eliding the 'http://' in our trie key is fine yet. We can always associate that with a registrar, as there won't be very many.

I would imagine there would be networking conventions around asking for chunks or bytes of the binary. I assume all the components of the 'Value' would be serialized and sent together. If the binaryBlob is a hash, I would assume the convention would be to send the hash, proof info for first chunk, and some indication of how many chunks exist for the blob. I would guess that an initial request could also ask for either a chunk offset or byte offset and how many chunks or bytes they want on the first response.

## Arrays/Collections
RDF or GunDB does not have a mechanism to handle arrays. GunDB doesn't, because arrays are not CRDT friendly (it requires you to stringify them and store them as String). RDF doesn't because triples aren't really setup to handle them well. So unfortunately we have no easy examples of how to do this.

My first thought is to require arrays to be defined as a binary suffix. However, since arrays are simply a single binary suffix encoding repeated over and over (either fixed length or variable length encoded), it seems like it is unnecessary. Arrays are so fundamental to computers I think we should take some time to figure out how they might work.

In GunDB only Objects that have Properties can be incrementally merged at the Property level using CRDTs. The value is either all or nothing. The same is true for our Subjects and their Properties. We have one difference between GunDB, we are not built around Javascript types. We are already in binary, so we might be able to get around converting an Array to a string.

If we want to allow a binary suffix to be arrayed, but it is already length encoded internally, then our high level system needs to read these bits to know how to delimit. Fixed length is just math. So Fixed length binary suffixes are free to array. Variable length suffixes need some thought.

We could just stipulate that all non-fixed length binary suffixes start with a single u-var-int to allow this encoding to be able to at least read every Nth byte(s) to delimit. This u-var-int encoding is like a universal binary suffix that is built into the system. It allows us to do stuff like this and build variable length numbers in to our keys, so we never worry about overflowing or wasting disk space.

Are all values an array? From a program control flow perspective, we should want to know if a value will be an array or not. Some properties, it wouldn't make sense to allow multiple values. Some it might make sense to require a fixed number (tuple-like), and others it could allow as many as the user wanted. If we require all binary suffixes to encode with our u-var-int as the leading byte(s), then constraining a Property Word's value, could describe the allowable array size. If a value must be single, the it is simply "1..=1" (or is simply the default, and can be elided?). For example, if you were storing RGB LED values, you might have a binary suffix of .u8, and then require "3..=3" for the array (tuple) size. The RGB example could *also* be defined by having 3 separate properties, Red, Green, and Blue, each with a single .u8. Or someone might make .rgbled binary suffix that is, internally, three .u8's but constrain *this* suffix to only "1..=1". The three separate properties approach is the only one that is technically CRDT friendly (allows incremental updates). The other options, while valid, would update the whole 'value' if any single value changed.

I think this is still fine. It gives options for different usecases. I think the most common usecase for arrays, will be a .resource suffix that is arrayed to allow one -> many relationships to be formed easily among 'Subjects'. Since these are relationship *between* resources, the values will never really be CRDT'd, and so, could be stored together. If an application were creating a complex and 'delicate' structure, then we must also allow a 'CompareAndSwap' update transaction. In theory, this would be at the query/network layer, so we will not worry about this at the moment.

Using this approach, we do open up a very minimal and 'unlabeled' value. Using .u8 (a single byte) and allowing an array, is a universal binary blob. This is not ideal, as we would like people to label the type to allow some sort of hooks to query and allow interoperability. I don't think there is much we can do except discourage developers from doing this.

### Anonymous Nodes: Revisited
If we allowed such an 'Array' construction, then we could potentially recreate AD's Anonymous Nodes. The trick, is that the Value is still recursive. However, since this would now be a binary suffix, such as: .anonNodePair, then you could fix the encoding of the 'Value' portion and make it not follow the rules of the larger system. This would mean in-lining all values, regardless of size, as well as encoding the meta data. You would lose the CRDT capability on all the 'path' values. This would technically work, but we would lose a lot of functionality. You are basically inlining an entire resource. So if *anything* changed on this anonymous resource, it would required updating/passing around the entire binary blob. However, I think you would probably just end up with a custom binary suffix to encoded whatever you wanted and avoid any type of generic Anonymous Node suffix. If you want a Generic Node, you can just create a new subject and link to it. Anonymous Nodes are just really hard to reconcile with addressability or searching.

Going through this thought experiment indicates to me that 'Subject+Property' are much more like 'memory addresses', and less like tangible 'subjects'. By disallowing anonymous nodes, we are forcing everything to be a 'first-class citizen'. This is great for allowing our system to be simpler, but requires many more 'subjects' in order to capture all the information.

### Enums








