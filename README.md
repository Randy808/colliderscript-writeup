# What is this?
This is just a quick write up on colliderscript that I'm making for BitDevs. I only read around 40-60% of the paper today and skimmed around a lot, so I may get things wrong but I think the basic gist is there. I'm planning on updating this as I spend more time to digest the paper, but if you want more of my thoughts on this then [come through](https://www.meetup.com/miami-bitdevs/events/304310570) :)

# Some helpful terms

The paper defines the concept of **small script** and **big script**. 

**Small script** refers to the group of opcodes that can manipulate up to 32 bits (OP_ADD, OP_SUB, etc). 

**Big script** refers to the group of opcodes that can manipulate values higher than 32 bits (OP_CHECKSIG, OP_SHA256, etc). These are mostly used to "manipulate signatures, hashes and other cryptographic objects".

# How is a ColliderScript covenant created?

So to make a ColliderScript covenant, the covenant constructor first needs to make a whole bunch[^1] of hashes for some seed 'w' and store them in a set 'D'.

Then we make a locking script that requires a sighash 's1' and a chunked sighash 's2' (chunked sighash), and add a commitment to 'D' in it. 

Before we provide any old sighash in the witness, we make a special one by grinding s1 until the hash(s1) equals one of the items in 'D'. Finally we make s2 by splitting the s1 we found into 32 bits, and pass both s1 and s2 in as part of the witness stack (with a token called the 'equivalence witness' specifying the item in 'D' s1 and s2 hash to). 

An important thing to understand is that s1 can be verified in big script with off-the-shelf opcodes. Small script meanwhile has to implement some mechanism that can calculate the hash of multiple stack items (the chunked signature) as if they were 1 input. The result is probably[^2] multiple items of output but that's ok.

The link to big script and small script comes from the seed 'w'. We can force the small script hash functions to use the same 'w' as the big script. 

When the small script generates the hash of s2 as multiple items, it'll also generate the hash series of 'w' as multiple items, and they can be compared piece by piece. 

As long as we verify that h(s1) equals recursive_hash(w) in big script, and h(s2) equals recursive_hash(w) in small script, we know that the signatures are the same since we forced the same 'w' to be used.

[^1]: A 'whole bunch' is a bit vague so I'll just mention the number thrown in the paper, 2^70. Look for the quote "In practice t is 70 bits" in the paper for more details. 

[^2]: I haven't seen how the BitVM hashes work so I say 'probably' just in case there's some magic I don't know about 👀