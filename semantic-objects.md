# It's semantics all the way down!
This document is going to be a bit of a rambling that hopefully I can clean up over time.

# The Triple
Basically the issue is that the SOD needs to work for all potential app data. If you decompose tables, you get RDF triples. If you are not familiar, the format is Subject-Predicate-Object (more later). It is a bit odd, and a little thorny to work with. [Atomic Data](https://docs.atomicdata.dev/atomic-data-overview.html) constrains RDF but still works in triples. Lets look at these in more detail.

## RDF "Triple"
The RDF spec is messy. But I understand how it ended up the way it did. I'm trying to prevent ending up there as well, but I just want to explain why I quoted "Triple". For those unfamiliar with RDF, it is a triple, because there are three components to a "fact" or piece of data:
* Subject - Bascially some ID to key up other triples that share this ID.
* Predicate - What semantically are we trying to describe about the Subject
* Object - Basically, what is the "Value" for the predicate on this subject.

The "Object" in RDF is always a string. However the string might be another resource(points to another Subject), a blank node (ignore this, we don't care about it for our construction), or it can be a literal. The literals are where I think the triple becomes conceptually fuzzy. So, the literal string has an encoding with optional properties to it. For example, you can add language tags: ```object = '"some string"@en'``` or datatypes: ```object = '"some string"^^<http://www.w3.org/2001/XMLSchema#string>'``` I believe you can do both at once. Not sure if there are other 'properties' but you see my point. So an the "object" really is an object in the sense of object oriented. It just has it's own special encoding and syntax.

So the RDF triple is at a high level still a triple, but we could think of it like so (in pseudo Rust):
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
RDF requires secondary things (programs? specifications? integrations?) to add Property-Datatype constraints. RDF is built like the internet, many layers. However this is also more for developers to understand and get up to speed with. That would be the impetus of the Atomic Data project. Frustration of working with RDF.

## Atomic Data Triple
[Atomic Data](https://docs.atomicdata.dev/atomic-data-overview.html) combined a lot of things in RDF to make it more usable. So let me quickly cover how it works. Basically "predicate" is now called a "property". Properties have a strict datatype at definition, getting rid of the weird encoding of the "Object", and thus "Object" is now called "Value". So something like this:
```rust
struct Triple{
  subject: URL,
  property: URL,
  property_datatype: URL, //'looked up' from the property referenced
  value: Value
}
enum Value{
  Anonymous(Vec<(propertyURL,Self)>), //replacement for Blank Nodes
  Data(String), //'decodable' type of property_datatype
}
```
So it is pretty clear to see why this is more usable than RDF. Atomic Data adds language tags through a [Translation Box](https://docs.atomicdata.dev/schema/translations.html), thus preserving all the functionality of RDF in a much cleaner fashion. Atomic data also doesn't use things like 'blank nodes' because they are not globally addressable pieces of data. Instead they use [paths](https://docs.atomicdata.dev/core/paths.html) to resolve nested objects. All Atomic Data is RDF but not the other way. So we have a better way of using the core concepts of RDF through Atomic Data. However I feel there is still something fundamentally missing to properly describe things.

## Language Tags/Translation Boxes
To most clearly illustrate my thoughts, lets focus on something that is part of RDF: languages. This is important as the internet is global. But there are lots of things different across the world. Here is my example on why "Translations" needs to generalized to the broader system:

Imagine you were a global non-profit org and wanted to create a survey in an RDF/AD like system. Great, you can translate your questions and make sure your UI loads the correct translation. But what if your survey asks a person's height? The property might have a symbol with a semantic meaning of "height of person" and value that is... What? Well we know it can be an unsigned integer, since people can't be negative height. But, should it be an integer? Depends on the resolution we want. But we still haven't figured out the **units** of this float/uint. If we used centimeters that would probably suffice in integer form. But what if we want to collect data from the stupid Americans and there weird units of measure? The survey participant would have no idea how many centimeters tall they are. 

So in the AD model, we would change the properties to be "height of person - cm" and "height of person - inches". So now we need a way to give these properties a common taxonomy so queries that ask for Subjects that have "height of person" it will return all people that have a height in either cm or inches.

## When to Suffix
So instead of a taxonomy, I think we should use the idea of a suffix again, like RDF, but differently. What things have suffixes? Almost everything? I think we need to sperate concerns of **WHY** we might put suffixes on things. 
1) There is a technical need to understand how to interpret the bits. Are these bits UTF-8, UTF-16, unsigned integer (we would probably fix the endianess of the system), etc.
2) There is a unit associated with a concept. In daily life, almost everything we deal with is a number and unit. Time, distance, speed, currency. We don't think about language as a unit, but I think it falls in line with the rest of these. It is simply String + Human Language instead of Number + Human Intuitive Construct

So I posit, that RDF wasn't way off with it's suffix in the value, I just think we need to think about how to describe stuff and where to put it.

## What is a Property, What is a Datatype?
So lets define some terms here. I would like re-use the property and datatype vocabulary, but I would like to define how I think about them.
### Property
So as you can tell from my example about height, I feel that a property is akin to a literal dictionary definition. So for height we get "the measurement from base to top or (of a standing person) from head to foot." Obviously there are other defintions below this, that I didn't mean, like "the most intense part or period of something." That is more of a problem of symbol mapping and is why you shouldn't include more than one definition when defining a property in this system. If we had some sort of shortname, these would collide and you would need disambiguation in the UI so the user was using the right "height", since it's true identifier is a [chain coordinate](https://github.com/ThinkingJoules/distributed-web-data/issues/2). So a property is a definition. This makes sense, and also why adding the units as directly part of it, is not quite right.
### Datatype
To me, either everything is a data type, or nearly nothing is. I prefer nearly nothing. I believe RDF puts units as part of it's suffix as 'datatype'. I think, in RDF you would need to create a specific datatype for all combinations of integers/floats and potential units. Basically you can't compose the combination, you need to define it. I tend to think of these as the technical reason for a suffix.

So I think of datatypes as primitive data: Boolean, Numbers (ints/floats), UTF-8. Numbers are a fun one. Do we think about the concept of numbers or the memory layout of them? So Rust would have u8,u16 etc. But really they are just [Natural Integers](https://en.wikipedia.org/wiki/Natural_number) with a maximum value constraint added? Or are they actually Integers with a min and max value constraint? Generally in computer science we don't think about the abstract types, but usually in Signed/Unsigned Integers, or Floats. Trying to represent numbers in binary is really hard, and ultimately all of this will be on computers. So I think we should opt for the computer types. Especially because floating point numbers are well understood and using abstract concepts aren't **really** what we are representing. I think that signed and unsigned integers could be simply variable length encoding, but I could see people wanting to use fixed size encoding as well. When we get into defining schema, constraints will need to be allowed.
### Units
I think we need to add a new word to our vocab so we can properly describe things. Units are really no different than properties, in that they are purely semantic concepts we made up. We just need to understand and convey the meaning of the symbol so it can be (re)used properly.
(The obvious next step when adding units, is adding conversions or translations. This is a deep subject and something we can build to, but lets put that in the back of our mind for now.)
I think that the units would really only be able to constrain the datatype to that of a 'string' or a 'number'. We could get around this if we split languages out from other units. Something to explore, given the language translations probably won't fit in to any sort of addressable function that could be called within this system. Would need to translate by running it through a trained language model of some sorts. Where as most of the unit *conversions* could be straightforward math.

## Typedef
So how would we define a property? Well for backwards compatibility I think we need to make 'units' optional. If we can build a conversion system, everyone will want to add/require them since the system will automagically deal with stuff in the background for us. Depending on how conversions work, I could see properties defining a list of valid combinations. What would we need to fully define this? Lets work on the height example from before

```js
shortName = "height"
description = "the measurement from base to top or (of a standing person) from head to foot."
units = [
  {
    unit: "{symbol for centimeters}",
    representations: [{symbol for u-var-int},{symbol for u8}],
    constraints:{max:305}
  },
  {
    unit: "{symbol for inches}",
    representations: [{symbol for u-var-int},{symbol for u8}],
    constraints:{max:120}
  }
]
```
This is just a rough idea. I think allowing different representations is fine, as long as we can encode the '{symbol for (repr)}' at the front of all binary encodings. Ultimately, there aren't going to be too many representations so I would think all clients should be able to figure out any of them. Representations (datatypes) might be easiest if we build them in to the implementation, since these are so low level the clients will need to implement them. We could also ensure the absolute smallest symbol tag. The downside, is that the datatypes are no longer directly addressable. Something to think about.

I don't know how to handle different base units, however, if we can sort of make auto-conversion happen then properties just have to put things in proper precision for their usecase, and the UI can convert to feet+inches or decimal feet or whatever makes sense for a particular usecase and context. At the low level, we just want to make sure it is well understood so conversions can be easy and automatic. Time is a solid example of bases not being nice. I would assume things would get stored in some sort of epoch based origin (Unix for example). If you didn't need much precision you could store it in Hours or Days to save data footprint, and then convert to the correct unit if you needed to add, say 10 minutes to it or something. Time is always hard and always will be.

This is why auto conversion will be tricky, but not impossible. We have already solved all of these problems many many times. So if we solve it one more time, then developers within this data paradigm should be able to work in more abstract things such as units, and pay less attention to the how.

Another advantage to all of this, is that if UI's (apps) built by different people, want to work in a different unit, given that context, they can. For example if app 'A' want to display a time duration in decimal seconds and app 'B' in milliseconds, regardless of what is 'on disk' either can display there preferred and also save back data in there preffered without needing to think about it, or have it 'mess up' the other apps logic.


--- WIP ---
# Low Level Primitives
I could imagine one of the early chains would define specifications for low level primitives that could be used and referenced in other projects (blockchain or not). The txn fee would probably be a crazy high amount of work. Something like days to weeks (maybe even a month?) of PoW to avoid unnecessary or useless primitives to be added (of course, someone still needs to implement it in software in some sort of client somewhere). If you could use the txn coordinates instead of hash, you could use this as a prefix on all the binary encodings so all data would be identifiable regardless of context. With sufficient work requirement you could easily end up with only a 2-3 byte prefix for all identifier tags. The goal is that this would become something like a permissionless and directly referenceable version of [multiformats](https://github.com/multiformats/multiformats) which is used in IPFS Content Identifiers. Not sure if all of 'multiformats' would be on a single chain or multiple. My default is more, simple chains. So looks like 3 chains according to their repo. [Addr](https://github.com/multiformats/multiaddr/blob/master/protocols.csv), [Base](https://github.com/multiformats/multibase/blob/master/multibase.csv), [Codec](https://github.com/multiformats/multicodec/blob/master/table.csv). 

Things I would consider for a 4th chain; Primitives: boolean, utf-8 blob, binary blob, var-int(u/i), fixed length integers (u8,u16...,i8,i16,...), floats of various flavors. Primitives would rarely be expanded, but I could see the early types going up to like u256, so if some app needed a u512, they could still add it. I think all IPFS CIDs are utf-8 that represent binary, and their 'Base' list is how to figure out how to get the right bits out of the string. My preference for the system is everything is binary by default. UI's can make things textual for us humans.

This could allow a new permissionless version of CIDs for use later in our non-DHT synching protocol.

# Extending Primitives
If you are making data machine readable, should you make it machine operatable? If we added another chain for [OPCODES](https://ethereum.org/en/developers/docs/evm/opcodes) we are well on our way to creating a universal (within this system) way to add functions to some sort of new machine semantic programming language. This seems extreme, but that is what is great by having this multi-chain system. We can just create the primitive chain first so we can have numbers and stuff, and then build on top of it later. That is the beauty of linked/semantic data. An interesting extension of this would be adding a suffix to primitives and can build conversion tables. Converting inches to feet or anything else the system has defined could be semantically understood and auto converted.
