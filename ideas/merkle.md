# Goal
In this document we hope to explore what merklized structures are, and when they should be used. We will also explore their usecase in the context of our larger system we aim to build.

# Merkle Structures: What are they?
All the various "Merkle ___" constructions all share the usage of cryptographic hashing. Depending on the required properties of a given application, the construction can be altered to fit the needs. The original design by Ralph Merkle was that of a tree.

Trees are a subset of a Directed Acyclic Graph (DAG), and so generally anything that is considered a DAG can be *merklized*.

What is the process of "merklizing"? Basically, any given structure will need to have a known 'serialization' of how to translate the 'nodes' in the graph into a [hash chain](https://en.wikipedia.org/wiki/Hash_chain#Binary_hash_chains). (Note: The wiki link talks of a binary hash chain, but this can be generalized to an n-ary hash chain. The tradeoff is less granularity on detecting differences within the structure, and increased proof size to prove a member is related to the root.)

# Merkle Structures: When to use them?
There are two (non-exclusive) reasons to use a Merklized structure: Finding Differences and Identity 'Compression'.
## Finding Differences
Probably the most familiar to all programmers is the Git source control program. [Git uses merkle trees](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects) to determine differences in a repository and create a sort of 'version on change' of the source files. 

Another usecase for merkle structures, is for synching a database efficiently from other peers that have presumably been online while your node was offline. For example, if the machine in question was offline briefly and missed a single update (to a key), a merklized structure could allow you to detect the single change very quickly, as opposed to asking for every key to determine that all but one has changed.
## Identity 'Compression'
I don't mean compression in the normal 'binary footprint minimization', but it is sort of similar. If you wanted to know whether two pieces of data were identical, without comparing every byte, you could hash the file. This hash is now a key (identity) for the series of bytes it was ran upon. This identity becomes a new mathematical 'name' for the series of bytes, allowing files to be asked for without needing a way to agree on a naming convention (you still need to agree on a hash function).

This works great, but creates a problem with large files. To prove a piece of data matches its identity, you must have *all the data*. Merkle trees in conjunction with 'chunking' the file, results in a different hash than hashing the whole value at once, but, if this is within a system that understands the merkle+chunking structure, then it can decode and verify that a *chunk* of data is part of the root 'identity' without needing all the rest of the file. 

(Note: [Blake3](https://github.com/BLAKE3-team/BLAKE3) has a built-in tree structure, where the hash is chunked per the hash spec. This means you can hash the whole file and then a tree can be built, per the hash spec, to connect any chunk to the root. This comes at the cost of not being able to specialize the chunk size for a given application, as well as the loss of a directly de-duplicate-able chunk. Basically the hash function handles the tree specification to allow the application to treat it as they would any other hash, at the cost of 'introspecting' or configuring the tree structure.)

# Structures
So we have an idea of what 'merklizing' looks like, but *what* should we be applying this to? I think we should cover 3 basic structures: [Trees](https://en.wikipedia.org/wiki/Tree_(data_structure)), [DAGs](https://en.wikipedia.org/wiki/Directed_acyclic_graph), and [Tries](https://en.wikipedia.org/wiki/Trie)
## Tree
This is the original structure Ralph Merkle used. It is often built in a binary form, but as noted earlier, does not need to be.

In a Tree, you have two types of nodes: Internal and Leaf. All the data is contained on the leaf, and the internal nodes are simply there to connect all the leaves up to the root. The data on a leaf does not necessarily require a key. A merkle tree without a key, can be built from an array of data, where the insertion order determines the hashes that result in the root. Even if you have the same data, but inserted in a different order, you will get a *different* root hash. Depending on what you are doing this may be a helpful property, in other uses, it can be detrimental.
### Uses
Nearly everywhere in distributed systems. From the [wikipedia article](https://en.wikipedia.org/wiki/Merkle_tree):
> "Hash trees are also used in the IPFS, Btrfs and ZFS file systems[4] (to counter data degradation[5]); Dat protocol; Apache Wave protocol;[6] Git and Mercurial distributed revision control systems; the Tahoe-LAFS backup system; Zeronet; the Bitcoin and Ethereum peer-to-peer networks;[7] the Certificate Transparency framework; the Nix package manager and descendants like GNU Guix;[8] and a number of NoSQL systems such as Apache Cassandra, Riak, and Dynamo.[9] Suggestions have been made to use hash trees in trusted computing systems.[10]"
## DAG
A DAG is hard to nail down exactly. Technically a Linked-List is a DAG and not a tree. This means DAGs can be either more complex than a tree, or potentially simpler.

The core distinction between a DAG and a tree, is that we can have more than one *path* from one node to another. This allows linking to no longer be purely hierarchial (as in a tree). This means that any linking is allowed, as long as we uphold the *acyclic* invariant (don't make links that could create infinite loops).
### Uses
Most of the protocols and programs listed in the 'Tree' section, usually incorporate those individual trees into some sort of internally known super structure that allows the 'Copy on Write' (CoW) effects. In the simplest form, the tree roots are simply linked in a list (which is technically a DAG), each newer root pointing to the previous tree root. Putting all of these roots in a list means that all unchanged branches can be found in many previous roots, hence the *entire* structure becomes a DAG, even though, conceptually, it is simply a series of trees connected by a list. So, in short, almost all *applications* work in DAG structures, not trees.
## Trie (aka Prefix Tree)
A Trie is an interesting twist from a Tree. It *requires* the association of a key to a value. This key is then used to navigate the Trie to find the key you are looking for. Due to this navigation on each node, tries build deterministically, and inserting two keys in a different order result in the *same* root hash. You 'build up' a key as you walk down the trie, so the entire key is never stored on any single node (except for the case where the trie has exactly one key in it). The *position* in the trie determines the key for that particular node. This is why it is sometimes called a prefix tree, because all keys below any given node *share* a prefix with that node.

There are two common types of Tries: Radix and PATRICIA. We will discuss these in more detail later.
### Uses
Ethereum uses a PATRICIA tree in part of its construction to prove state. There are a couple other blockchain projects using variations on this idea as well. 

Outside of the merklized uses, they are used in many applications ([according to wikipedia](https://en.wikipedia.org/wiki/Trie#Applications)):
>"Trie data structures are commonly used in predictive text or autocomplete dictionaries, and approximate matching algorithms.[11] Tries enable faster searches, occupy less space, especially when the set contains large number of short strings, thus used in spell checking, hyphenation applications and longest prefix match algorithms."

# Relevance to our Dweb Idea
We are using the Triple as our core data model, and we have [decided](https://github.com/ThinkingJoules/distributed-web-data/blob/main/architecture.md#snapshots-or-all-transitions) that we must snapshot our Non-Repudiable Data (NRD). I'm focusing on NRD here, as that is the primary usecase for merklized structures. However, we could reuse the same approach for the private data portion of our larger system. 

Merklized data structures are great for snapshots. We can prove any particular piece is part of the snapshot. If we want to add authentication to this, we can simply sign our root, instead of signing every piece of data.

Which of the three data structures discussed will work best for us?

## Trie vs Tree
Let's talk about the low level construction first, and not worry about a DAG.

Since our data has 'keys' (Subject + Property) then using a Trie (Prefix Tree) makes a lot of sense. So we have Subject + Property = Value. All of our Properties would share the subtree of our Subject Prefix. This allows our multipart key to *feel* more multipart. We can see the prefix, so using a Prefix Tree is hard to argue against.

This works great for finding our value. The subject and property key segments should be relatively short. However, the 'value' in our triple could potentially have a very large binary blob. This lines up with our idea of using a Tree for incremental proofs.

Let's try using a single Trie for the triple, and then a nested Merkle Tree for *each* value to see if we can get the best of both worlds.

## Trie + Tree
The advantage of using a separate tree for the binary blob, is that we can now represent the value with a hash, and prove that any chunk belongs to this hash. We also gain de-duplication of the chunks across *all* values.

The downside of this approach, is that we now have two different data structures that have different encodings and therefore different proofs. We also need to traverse two tree-like structures. However, the second tree will only be large on values that have many chunks.

We get a nice separation of concerns here. We can prove non/inclusion of any value by the deterministic qualities of a Trie, and that value could be arbitrarily large, and allow fetching and proving in chunks.

### Small Values
So the above construction works great when we have large values (like files), but surely we don't want to represent a boolean, 1 bit of information, with a 256 bit hash? 

We could allow the skipping of the hash entirely, as long as the value is less than some threshold. This would allow us the ability to represent most all numbers and boolean values, as well as short strings, directly in-line. In theory we could inline the entire first chunk of data without hashing. Only when we have enough data that we need two chunks, we hash and build a Merkle Tree. This runs the risk of increasing proof sizes, but if you want the data anyway, and we can avoid this key in our trie from being a prefix of some other key below it, then it really would be free. So we could potentially allow the value to be up to the size of our chunk size from a networking perspective. From a de-duplication perspective, we just need to have some idea on how much overhead we can afford.

So what is a good chunk size?
### Chunk Size
The balance here, is needing to not generate too much overhead by having too small of a chunk size. This is at odds with the fact that the smallest possible chunk size will give max de-duplication. So how much overhead is too much? For context, the ext2/3/4 file systems use about 1.6% overhead on their inodes.

How do we calculate this? Let our chunk size be 'X'. If we assume a balanced tree, then we just need to calculate a tree with a root node and two leaves to find our overhead.

Let's run this and see what we get:
```js
const HASH = 32;
const ROOT_SIZE = 2*HASH;
const LEAF_SIZE = HASH
const BOTH_LEAVES = LEAF_SIZE*2
const TOTAL_LABEL_OVERHEAD = ROOT_SIZE + BOTH_LEAVES

let testChunks = [256,512,1024,2048,3072,4096]
let overhead = (chunk) => TOTAL_LABEL_OVERHEAD/(chunk*2) //chunk*2 since we have two 'full' chunks worth of data in this tree
let res = []
for (chunk of testChunks){
    let dec = overhead(chunk)
    res.push({chunk,overhead:Math.round(dec*10000)/100})
}
console.table(res)
```
|  chunk (bytes)| % overhead |
|:-------------:|:----------|
| 256           |   25.0 |
| 512           |   12.5 |
| 1024          |	6.25 |
| 2048          |	3.13 |
| 3072          |	2.08 |
| 4096          |	1.56 |


IPFS uses a chunk size of 1024b, which yields about 6% overhead. Coincidentally 1024b is also the chunk size used internally with the Blake3 hashing algorithm. 1024b also fits nicely into a single IP Packet. There seems to be a lot of things using this 1024b number. So I guess we could use that, and say that 6% overhead is acceptable? If we wanted lower overhead (but probably less de-duplication) we could choose a larger chunk size.


# Trie Nodes
## First: Difference between PATRICIA and Radix Trees
Before explaining a PATRICIA Node, I want to clarify the difference between PATRICIA Trees and Radix Trees (both tries).

The inventor of the Radix Tree originally called this creation a [PATRICIA Tree](https://en.wikipedia.org/wiki/Radix_tree#History) (Practical Algorithm to Retrieve Information Coded in Alphanumeric). According to wikipedia:
>"Today, Patricia tries are seen as radix trees with radix equals 2, which means that each bit of the key is compared individually and each node is a two-way (i.e., left versus right) branch."

From my research on implementations of Radix vs PATRICIA trees, wikipedia seems incorrect in the statement that Radix and PATRICIA trees are the same except for one parameter. 

The effect of the difference is that an empty Radix Tree has a single node, whereas an empty PATRICIA Tree will have *no nodes*. I will demonstrate a simplified node of each type (in pseudo rust):
```rust
enum Char{
    A,
    B,
    C,
    D,
    //...rest, omitted for space
}

struct RadixNode{
    children: HashMap<Vec<Char>,Self>//may be empty
}
struct PatriciaNode{
    key_extension: Vec<Char>//may be empty
    children: HashMap<Char,Self>//may be empty
}
```
As you can see, the type signature of the 'children' is most indicative. A Radix Tree carries the key extension bits entirely on the 'edge' that points to the child. For a PATRICIA Tree, the sparseness of a subtree is compacted together *on* the node. The children then point to subtrees that contain the next different bit.

I think this nuance shows up when trying to 'compact' the sparse subtrees. In a PATRICIA Tree you must read the key_extension bits first to know the full key *for that node*. Whereas, in a Radix Tree, you would read the 'value' first, because you would know the 'key' that labels the particular node *when you traversed to it*. This allows a Radix Tree to have a single node with a 'key' of nil with no children when the database is empty.

A true PATRICIA Node would use a 'skip number' instead of the Vec\<Char> construction. I put the bits directly on the node, as it allows you to detect a non-included key sooner in the traversal of the tree. It also allows a true 'building up' of the key as you go, instead of storing the full key on nodes that have a key. This saves a bit of space in-memory, else otherwise, related keys would repeat their entire prefix on the node that has a key.

## Address vs Key
To help explain the nature of a trie, I would like to define a term: Address. Since a trie requires a key and is deterministic, *every* node in a trie has an **address**. When we use the term **key**, we mean data inputted into the trie. The trie then creates necessary intermediate nodes that have *addresses*, one of which will *also* have the key being added. In a binary tree, the address on any given path will always increase in bit length as you go deeper into the trie. This brings me to a distinction for when we mean 'Prefix' in these tries. Let me illustrate the nuance with an example. Say we have four keys: AABAB, AABBB, BB, and AA. AABAB and AABBB *share* a prefix, however only AA *is a* prefix of either key. To be a prefix, a key must be shorter than another key and have all of its bits in common. I would define a **sibling** as two keys that *share* a prefix. So AABAB and AABBB are siblings. AA and BB are simply unrelated.

## Example PATRICIA Node
All tries have a single 'node type', since a key/value can appear internally and not just on a leaf.

To both simplify and represent our most likely real-life construction, we will work with a Binary PATRICIA Tree for the rest of this document. So the alphabet is {0,1}.

A node could have:
- Extension bits
- Key
- Value (if we want to allow key-only set-like constructions)
- Left Child {0}
- Right Child {1}

Lets use some pseudo rust to make this more clear:
```rust
struct BinaryPatriciaNode{
    key_extension: Vec<bool>, //may be empty if the left and right branches differ in the very next bit.
    kv: Option<Option<Vec<u8>>>
    left_child: Option<Box<Self>>,
    right_child: Option<Box<Self>>,
}
```

The double option is a bit messy. The outer Option indicates if there is a key present on this node, and the inner Option is for indicating if there is a value on the node, since we can only have a value if there is a key present. We could elide the inner Option if we simply count the Vec\<u8> with no bytes as a 'None' value. For now, let's assume a Some(Some(_)) vec has a non-zero length value. 

### Example PATRICIA Tree
Let's enter five key-value pairs into a PATRICIA Tree (pTree) to see how it works. The (binary) keys and value, in the order we will add them: 
- ( [0,1,0], [0,0,0] )
- ( [0,0,1], [0,0,1] )
- ( [1,0,0], [0,1,0] )
- ( [1,0,0,0,1,1,0], [0,1,1] )
- ( [0], [1,0,0] )

An empty pTree has no nodes. So let's add the first key:
```
└── Root {key_ext: [0,1,0], kv: Some(Some([0,0,0]))}
```
Our root node has an *address* of [0,1,0], and kv being Some(_) indicates there is a key here as well. Let's add the next key:
```
└── Root {key_ext: [0], kv: None} // address of [0]
    ├── left_child {key_ext: [1], kv: Some(Some([0,0,1]))} //address of [0,0,1]
    └── right_child {key_ext: [0], kv: Some(Some([0,0,0]))} //address of [0,1,0]
```
How do we get a three bit address from only having a total of two bits in the key_exts? Because each traversal implicitly adds a single bit. Going to the left_child will append a [0] bit to our address, going to the right child a [1] bit.

Now for our third sibling key:
```
└── Root {key_ext: [], kv: None} // address of []
    ├── left_child {key_ext: [], kv: None} // address of [0]
    |    ├── left_child {key_ext: [1], kv: Some(Some([0,0,1]))} //address of [0,0,1]
    |    └── right_child {key_ext: [0], kv: Some(Some([0,0,0]))} //address of [0,1,0]
    └── right_child {key_ext: [0,0], kv: Some(Some([0,1,0]))} // address of [1,0,0]
```
Because we have three keys and they are all sibling we end up with an unbalanced tree. Now let us add our very long key that shares a prefix with our last key:
```
└── Root {key_ext: [], kv: None} // address of []
    ├── left_child {key_ext: [], kv: None} // address of [0]
    |    ├── left_child {key_ext: [1], kv: Some(Some([0,0,1]))} //address of [0,0,1]
    |    └── right_child {key_ext: [0], kv: Some(Some([0,0,0]))} //address of [0,1,0]
    └── right_child {key_ext: [0,0], kv: Some(Some([0,1,0]))} // address of [1,0,0]
         └── left_child {key_ext: [1,1,0], kv: Some(Some([0,1,1]))} // address of [1,0,0,0,1,1,0]
```
And now our final key, which is a prefix key:
```
└── Root {key_ext: [], kv: None} // address of []
    ├── left_child {key_ext: [], kv: Some(Some([1,0,0]))} // address of [0]
    |    ├── left_child {key_ext: [1], kv: Some(Some([0,0,1]))} //address of [0,0,1]
    |    └── right_child {key_ext: [0], kv: Some(Some([0,0,0]))} //address of [0,1,0]
    └── right_child {key_ext: [0,0], kv: Some(Some([0,1,0]))} // address of [1,0,0]
         └── left_child {key_ext: [1,1,0], kv: Some(Some([0,1,1]))} // address of [1,0,0,0,1,1,0]
```
Since we already have a node that has an address of [0], we do not need to alter the structure of the trie, we simply need to change the kv to Some(_).


### Serialization
When working with cryptography everything is binary. So we need a specification on how to translate the abstract idea of a pTree node into a decipherable format. To aid in this, we can have a single leading bit mask byte to represent the 'shape' of the particular node, allowing fixed length segments from needing length encodings. Here is a purely illustrative example encoding (in [ABNF](https://en.wikipedia.org/wiki/Augmented_Backus%E2%80%93Naur_form)):
```abnf
binary-patricia-node = bit-mask-byte [child-data] [key-extension] [value-data]

bit-mask-byte = left-child-bit right-child-bit key-ext-bit key-bit value-bit 3%b0 ;8bits/1byte/OCTET, 3 unused
left-child-bit = %b1 / %b0
right-child-bit = %b1 / %b0
key-ext-bit = %b1 / %b0
key-bit = %b1 / %b0
value-bit = %b1 / %b0

child-data = [left-child-hash] [right-child-hash]  ;could be 0 bytes i.e. on any leaf node
left-child-hash = 32OCTET ;assumes a standard 256bit cryptographic hash
right-child-hash = 32OCTET

key-extension = u-var-int 1*OCTET ;number of **bits** = u-var-int value, must do math to determine number of OCTETs

value-data = u-var-int 1*OCTET ;num of OCTET = u-var-int value

u-var-int = (1*continue-byte terminal-byte) / terminal-byte
continue-byte = %b1 7BIT
terminal-byte = %b0 7BIT

```
We can see how our Rust struct can be translated into this binary representation. The hashing would be done during the snapshot as we traverse the tree in [post-order](https://en.wikipedia.org/wiki/Tree_traversal#Post-order,_LRN).

Remember that the 'key' and 'value' here, are very literally keys and values. There is no 'typing' of the key or the value. The key *might* be a Subject+Property, but it could also be Subject+Property+ValueComponent.

The important thing to note on the above serialization specification: if there is a value (as indicated in the bit-mask-byte) — *the entire value* must be serialized into the trie node for hashing. This is because there is no distinction between internal nodes and leaf nodes, since an internal node *can* have a key/value. This is why we want to be very careful how we handle values entered into the prefix trie. 

### Proof Generation and Verification
If we ever need to prove a key/value in our trie, we have to be aware that any key (that has a value) that is a prefix of the key we are trying to prove, will *also* need its value sent in the proof. In effect, a proof for one key, will also be proving *all key/value pairs* where the key is a prefix of the key being proved. Let me explain the differences between a merkle tree and a merklized prefix tree.

In a normal merkle tree, we only need to supply a single hash at each depth in the tree, since the verifier would be *generating* the other hash at each depth of the tree on the path back to the root the prover is attempting to prove. Since a trie can have keys as a prefix key/value *within* the trie, we have a single node 'type'. This means the protocol to prove/verify needs to be able to know the 'shape' of each node. This means that we must always supply more than *just* the sibling hashes to 'build back' to the root. 

Since our internal nodes can have key with (optionally) a value, we need to provide all the bits that *are not* the hash that the verifier can generate. Remember, the verifier can *only* reconstruct hashes. So we must provide them with all other bits of the node, so it will match the serialization spec. 

Let's work through an example to make this simpler. Using the trie from before:
```
└── Root {key_ext: [], kv: None} // address of []
    ├── left_child {key_ext: [], kv: Some(Some([1,0,0]))} // address of [0]
    |    ├── left_child {key_ext: [1], kv: Some(Some([0,0,1]))} //address of [0,0,1]
    |    └── right_child {key_ext: [0], kv: Some(Some([0,0,0]))} //address of [0,1,0]
    └── right_child {key_ext: [0,0], kv: Some(Some([0,1,0]))} // address of [1,0,0]
         └── left_child {key_ext: [1,1,0], kv: Some(Some([0,1,1]))} // address of [1,0,0,0,1,1,0]
```

Let's convert the necessary trie nodes to our binary serialization format that can prove the kv pair of [0,0,1]:

```
└── Root {bit-mask: 0b11000000, left-child-hash: TO_BE_GENERATED, right-child-hash: XYZ} // address of []
    ├── left_child {bit-mask: 0b11011000, left-child-hash: TO_BE_GENERATED, right-child-hash: ABC, value-data: [1,0,0]} // address of [0]
    |    ├── left_child {bit-mask: 0b00111000, key-ext: [1], value-data: [0,0,1]} //address of [0,0,1]
// We don't care about the rest of the tree directly
```
(Note: The bit mask should match up to the proper properties specified above in our example ABNF serialization spec.)

As we can see, by proving that key [0,0,1] has a value of [0,0,1], we also prove that its parent node with address [0] also has a key, and value-data of [1,0,0]. The value-data must be shipped with the proof, thus increasing its size. This is the nature of using different bit length keys in this trie.
