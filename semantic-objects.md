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
  _property_datatype: PropDef, //'looked up' from the property referenced
  value: Value
}
enum Value{
  Anonymous(Vec<(PropertyURL,Self)>), //replacement for Blank Nodes
  Data(String), //'decodable' type of _property_datatype, might be a URL for another resource
}
```
So it is pretty clear to see why this is more usable than RDF. Atomic Data adds language tags through a [Translation Box](https://docs.atomicdata.dev/schema/translations.html), thus preserving all the functionality of RDF in a much cleaner fashion. Atomic data also doesn't use things like 'blank nodes' because they are not globally addressable pieces of data. Instead they use [paths](https://docs.atomicdata.dev/core/paths.html) to resolve nested objects. All Atomic Data is RDF but not the other way. 

So we have a better way of using the core concepts of RDF through Atomic Data. However I feel there is still something fundamentally missing to properly describe things.

## Language Tags/Translation Boxes
To most clearly illustrate my thoughts, let us focus on something that is part of RDF/AD: languages. This is important as the internet is global. But there are lots of things different across the world. Here is my example on why "Translations" needs to generalized to the broader system:

Imagine you were a global non-profit org and wanted to create a survey in an RDF/AD like system. Great, you can translate your questions and make sure your UI loads the correct translation. But what if your survey asks a person's height? The property might have a symbol with a semantic meaning of "height of person" and value that is... What? Well we know it can be an unsigned integer, since people can't be negative height. But, should it be an integer? Depends on the resolution we want. But we still haven't figured out the **UNITS** of this float/uint. If we used centimeters that would probably suffice in integer form. But what if we want to collect data from the stupid Americans and there weird units of measure? The survey participant would have no idea how many centimeters tall they are. 

So in the AD model, we would change the properties to be "height of person - cm" and "height of person - inches". So now we need a way to give these properties a common taxonomy so queries that ask for Subjects that have "height of person" it will return all people that have a height in either cm or inches.

## When to Suffix
So instead of a taxonomy, I think we should use the idea of a suffix again, like RDF, but differently. What things have suffixes? Almost everything? I think we need to sperate concerns of **WHY** we might put suffixes on things. 
1) There is a technical syntax to the bits. Are these bits UTF-8, UTF-16, unsigned integer (we would probably fix the endianess of the system), etc.
2) There is a human grammar/syntax to the UTF-8. What human international language is this text in?
3) There is a unit associated with a concept. In daily life, almost everything we deal with is a number and unit. Time, distance, speed, currency. We don't think about language as a unit, but I think it falls in line with the rest of these. It is simply String + Human Language instead of Number + Human Intuitive Construct

So I posit, that RDF wasn't way off with it's suffix in the value, I just think we need to think about how to describe stuff and where to put it.

## What is a Property, What is a Datatype?
So lets define some terms here. Let us re-use the property and datatype vocabulary, but I would like to define how I think about them, at least at this point in the document.
### Property
So as you can tell from my example about height, I feel that a property is akin to a literal dictionary definition. So for height we get "the measurement from base to top or (of a standing person) from head to foot." Obviously there are other defintions listed in the dictionary that I didn't mean, like "the most intense part or period of something." That is more of a problem of symbol mapping and is why you shouldn't include more than one definition when defining a property in this system. If we had some sort of shortname, these would collide and you would need disambiguation in the UI so the user was using the right "height", since it's true identifier is a [chain coordinate](https://github.com/ThinkingJoules/distributed-web-data/issues/2). So a property is a definition. This makes sense, and also why adding the units as directly part of it, is not quite right.
### Datatype
To me, either everything is a data type, or nearly nothing is. I believe RDF puts everything as part of it's suffix as 'datatype'. Basically you get one tag to define any encodings (since it is all text), underlying types (integer, boolean, etc), and the unit (centimeters, inches, etc).

So ***I*** think of datatypes as primitive data: Boolean, Numbers (ints/floats), UTF-8. Numbers are a fun one. Do we think about the concept of numbers or the memory layout of them? So Rust would have u8,u16 etc. But really they are just [Natural Numbers](https://en.wikipedia.org/wiki/Natural_number) with a maximum value constraint added? Or are they actually Integers with a min and max value constraint? Generally in computer science we don't think about the abstract types, but usually in Signed/Unsigned Integers, or Floats. Trying to represent numbers in binary is really hard, and ultimately all of this will be on computers. *So I think we should opt for the computer types*. Especially because floating point numbers are well understood and using abstract concepts aren't *really* what we are representing. I think that signed and unsigned integers could simply be variable length encoding, but I could see people wanting to use fixed size encoding as well. When we get into defining schema, constraints will need to be allowed.

More on datatypes later.
### Units
I think we need to add a new word to our vocab so we can properly describe things. UNITs are really no different than properties, in that they are purely semantic concepts humans made up (more preciesly, they carry a learned *intuition* of scale). We just need to understand and convey the meaning of the (UNIT's) symbol so it can be (re)used properly.
(The obvious next step when adding units, is adding conversions or translations. This is a deep subject and something we can build to, but lets put that in the back of our mind for now.)

## PropDef
So how would we define a property? Well, if we want backwards compatibility I think we need to make 'units' optional during definition. If we can build a conversion system, everyone will want to add/require them since the system will automagically deal with stuff in the background for us. Depending on how conversions work, I could see properties defining a list of valid combinations. What would we need to fully define this? Lets work on the height example from before

```json
{
 "shortName": "height",
 "description": "the measurement from base to top or (of a standing person) from head to foot.",
 "datatypes": 
 [
   {
    "unit": "{symbol for centimeters}",
    "representations": ["{symbol for u-var-int}", "{symbol for u8}"],
    "constraints": {"max":305}
   },
   {
    "unit": "{symbol for inches}",
    "representations": ["{symbol for u-var-int}", "{symbol for u8}"],
    "constraints": {"max":120}
   }
 ]
}
```
This is just a rough idea. I think allowing different representations is fine, as long as we can encode the '{symbol for (repr)}' at the front of all binary encodings. I think this is doable, only if we can do [chain coordinate](https://github.com/ThinkingJoules/distributed-web-data/issues/2) as the tag.

I will talk about the Feet + Inches vs Inches, or what I will call the 'multi-base' problem, later.

Auto conversion will be tricky, but not impossible. We have already solved all of these problems many many times. So if we solve it one more time, then developers within this data paradigm should be able to work in more abstract things such as UNITs, and pay less attention to the how.

Another advantage to all of this, is that if UI's (apps) built by different people, want to work in a different UNIT, given that context, they can. For example if app 'A' wants to display a time duration in decimal seconds and app 'B' in milliseconds, regardless of what is 'on disk' either can display there preferred units and also save back data in there preffered units without needing to think about it, or have it 'mess up' the other apps logic.

## Suffix Cases
So lets look at a human level, how we are already familiar with using suffixes.

* Filetypes - Operating systems use these as hints to know what program understands the syntax/grammar of the bits in the file.
  * These would be "FILE_EXT"
* Lanuguage - We don't usually think about this one, but at world-scale, it is clear.
  * These would be "LANG_TAG"
* Units - As mentioned above, most numbers will have a suffix to give it a human intuitive scale (Time, Length, Currency, etc).
  * These would be "UNIT" 

We can see our earlier suffix distinctions coming through. FILE_EXT is basically a language tag for computers. VS Code relies heavily on file suffixes to properly enable the right extension that can 'read' the file and run background linting, code completion, etc. So FILE_EXT and LANG_TAG are really similar conceptually. But what if we, say, have a markdown file written in english? Can we have *both* tags at once? Ultimately if a computer could understand how to differentiate the markdown syntax from the human syntax, I suppose it could translate a ```.md.en -> .md.it``` file. Many computer files deal with human text. So this seems like it needs to be allowed so future applications are able to build auto-translators. Most browsers already have this. It is basically ```.html.en -> .html.it ```. I'm not sure if browsers detect the language or by 'detect' it looks for a language tag in the html document. I have been on foreign (to me) language sites that the browser never prompted me, so I suspect it still needs hints. However this could change in the future as ML models get better.
<!---
### Nested Syntax
I don't think we want to allow more than the two different language extensions. For example if we had an array of .pdf's and we have an 'Array' FILE_EXT, then we could do .array.pdf.en? I mean, perhaps I picked the wrong example, since Array *sounds* useful (I don't think we want it at this high of a level)... But if we want to restrict things, then we would have to make a new extension like .arrayPDF and then we would probably not require a LANG_TAG, and instead add the LANG_TAG as part of the encoding, since PDF's can be of different languages. Arrays in Javascript can be whatever, but in Rust, a Vec must be a single Type. I think it is best to simply make people build their own container file extensions.

Let us try a different example. What if we wanted to do a tuple of some sort? Maybe like for RBG lights. .u8.u8.u8 This would be convienent, but wouldn't provide any information, besides the property symbol itself to query with. I don't see it adding much, but making things more confusing and complex at this level.
-->
### Queries
An example of what this might look like. If we had a property that meant 'height' (as defined from example earlier). Then we could query the data for all subjects that have this property (and probably other properties to ensure we have humans and not buildings) and then simply output the heights in whatever unit we wanted to work in (if the automagic conversions were working). These hints allow hooks for control flow to be composed with minimal code. The hooks can also be used for high, mid, or low order querying. High order would be selecting subjects that have X property(s). Akin to Classes, but descriptive instead of prescriptive. Mid order could be further selecting based on suffixes (heights in 'centimeters', .pdf encodings, .en lang tags). Low order would become tricky, given that auto conversions would need to work. So you could say 'Things with Height > 60"' and still return proper results for things that have units recorded in something other than inches.

## Datatypes vs filetypes
Let us look at datatypes for a second. Are these also file types? What is the difference between a novel in a .txt vs inlined in the value as a 'string'(datatype)?
In the full system [document](https://github.com/ThinkingJoules/distributed-web-data/blob/main/all-at-once.md) you will see that at a technical level I *think* we have to treat everything as binary and chunk things to allow 'streaming' of any file/data-type. However, at this level we don't have to treat them the same. I *think* you could make a distinction without any trouble. So, should we?

Is .u8 or .u-var-int or .utf-8 okay? Well the .utf-8 feels like a subset of .txt or something. I'm not sure where the [line ending difference](https://en.wikipedia.org/wiki/Newline#Issues_with_different_newline_formats) lies. Anyway, if two things are semantically understood as different for humans but logically the same from a technical perspective, I would prefer to uphold the conventions in the UI and keep the logic ... logical. There is really only one reason (that I can think of) to tag data/file-types, and that is to build control flow in applications to deal with it properly. For a UI, it could be the difference to display a .jpeg in an ```html<img>``` tag versus say a .u8 might have ```html<input type="number" min="0" max="255">``` (or min, max from the "constraints" in the PropDef).

The biggest thing I can see that I don't like, is that it would be nice to have some sort of hierarchy or taxonomy of filetypes. However, if we look at this from a developer/symbol perspective, everything is just symbols. If everyone knows the 'core' file extensions, then they will work *as* datatypes. If we split datatypes out, then this is a new primitive in the System, even though the tooling has to be built around it regardless of how/where the symbol was created. If we can also build [good search](https://github.com/ThinkingJoules/distributed-web-data/blob/main/identifiers.md#reuse-through-discoverability---one-step-farther) by putting these core things on a chain, then finding what we want should be pretty easy.

So lets go with datatypes=filetypes and see how that plays. 

Everything is a binary blob, with a FILE_EXT and an optional LANG_TAG (ideally this would be required based on the FILE_EXT, for continuity across all data that has that extention. So something like a markdown (.md) would **always** need a LANG_TAG).

Let us think of the implications for filetype conversions (*not unit conversions*). So in programming, most commonly with numbers, we can ['cast' between types](https://en.wikipedia.org/wiki/Type_conversion). So can we extend this to files? We convert many types of things to many other types. .jpeg<->.png, .mp4<->.mov, u8<->u16, Numbers<->Strings. Some conversions are lossless and some are lossy. This would be something to be aware of when doing conversions. *UNITS* suffer from the same problem. Moving from one UNIT with higher precision to a lower precision type can cause a loss of information. This is technically even true when converting integer centimeters to decimal meters. Float representation is *often* lossy (if you are working at that level of precision, you probably already know this, and build around it). The information world is a complicated place.

So I see some similarities here. All are convertable things. The complexity and lossyness of the conversion varies widely. .u8->.f64 is pretty trivial. .u8.cm -> .f64.meters is a bit harder. A .docx.en -> .pdf.it is very hard. HOWEVER, **if** the subroutines were written, you could convert from TypeA -> TypeB.

So let us state: TYPE=FILE_EXT=datatype=filetype. TYPEs are unit-less.

So what we really have is a binary blob plus a FILE_EXT with a (conditional LANG_TAG) || (UNIT). (Note: The Unit is probably written in a language, so should need its own LANG_TAG, however, since we are using it as a symbol, the language conversion of the description (which WOULD have a LANG_TAG) would simply need to be translated to the target language in the UI.)

## TypeDef (FILE_EXT)
We need to add another low level Symbol for our TYPEs. PropertyDef would use this Symbol for listing what is valid. This Symbol would also be used within the Value so it can be evaluated when read. Naively the Def might look something like:
```json
{
 "binarySuffixAbbreviation": "md",
 "description": "Uses the CommonMark Spec: https://spec.commonmark.org/",
 "langTag": true,
}
```
All the real info is contained in the URL, which may or may not last. Is this okay? I think it is fine. Once the system is bootstrapped, why couldn't the DNS URL one day become addressable within this larger system? If our system can store any sort of renderable text, future specs could be written here. Hopefully we can have a Git-like option for data that makes sense to use that way.

## UnitDef (UNIT)
```json
{
 "unitAbbreviation": "cm",
 "unitName": "Centimeter"
 "description": "a metric unit of length, equal to one hundredth of a meter",
}
```
Units will almost always be described in some conversion to other similar units. This definition only needs to work for humans.

# The Triple 2.0
Okay. Let us take a look at what sort of mess we have created. We will try to fit it in our best candidate, Atomic Data:
```rust
struct Triple{
  subject: URL,
  property: URL,
  _property_def: PropDef, //'looked up' from the property referenced
  value: Value
}
enum Value{
  Anonymous(Vec<(PropertyURL,Self)>), //replacement for Blank Nodes
  Data(Data),
}
struct Data{
  bytes: Vec<u8>,
  binary_type: URL, //FILE_EXT, constrained based on _property_def
  _binary_type_def: TypeDef, //TypeDef 'looked up' from the binary_type referenced
  units: Option<URL>, //required/constrained based on _property_def
  lang_tag: Option<URL> //required based on _binary_type_def
}
```
So it looks extremely similar to Atomic Data. It wouldn't take much of a change. We have preserved blank nodes using the Atomic Data 'Path' convention. We also have a place to put the LANG_TAG, but it is now dependent based on what encoding/FILE_EXT used, which itself is constrained by the property_def.

Seems like a lot going on, but from a user perspective it should really be welcomed. Back to our survey question. When entering their height, there could be a UNITs drop down to the right of the number input field. They select whichever one they want or feel the most comfortable entering, and the UI can update the validation on the field(s) for entering. The developer now doesn't have to anticipate all user input preferences. This makes it better for everyone, as long as converting between types/units is nearly free.

## Multi-base UNITs
The multi-base problem is still present however (Height in Feet + Inches). Let us list where multi-base is an issue:
* Time
  * [Years<->Months,...<->Milliseconds](https://en.wikipedia.org/wiki/Unit_of_time)
  * Duration (in US) is often written HH:MM:SS. This is 3 u-ints and each have a UNIT.
* Length/Distance
  * US: Feet + Inches
* Angles
  * Degree Minute Seconds
* Weights
  * US: Pounds + Ounces
* Currency
  * Almost all currency is now base 10, but how many layers of base 10? US is two. Dollars and Cents. Some currencies might not be base 10?

Technically most any set of units could be put together with the lowest precision first.

What is NOT a problem?
* Area/Volume
* Velocity/Acceleration/Jerk/...
* Temperatures
* Pressures
* Power/Energy

So why do multi-bases exist? Units are ***intuitive*** scales to reason in. Large numbers are not easy to intuit, thus we use multi-base conventions. Basically things that don't span vast scales, so large numbers aren't much of an issue. Or they span large scales, but aren't used by everyday people (they are engineering units).

So where should we put this complexity? This is a little bit like the thoughts on numbers. We used computer specs, not mathmatical concepts. At this stage we are trying to get a consistent and understandable data model/layout. At this level of the system I think I would opt for a single unit tag. If the only reason we have multi-base is to allow humans to avoid large numbers (and therefore better intuit data), then our computers certainly don't care. So I opt for pushing the complexity to UI's/middleware. Since there is only a handful of instances, there will simply be more tooling built around input/display of those odd ones. I think we will probably have to store UI configs in data somewhere, so we could store the users preferred display type (and language so UI's could auto populate that during creation). These would be a separate set of well known (in the UI context) symbols to mean "HH:MM:SS" or "Feet+Inches". Formatting and displaying data in a UI can have lots of configs, but ultimately we don't want the data to be rounded, we want to display it rounded. Displaying and computation are two different concerns. In Google Sheets, you can change the number of decimal places to display, but there is also several forms of ROUNDing functions. The former computes on the unseen decimal digits, the later performs the rounding and then operates on the output.


