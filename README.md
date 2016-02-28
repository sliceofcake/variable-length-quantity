# variable-length-quantity

## Overview
A scoped function collection for encoding and decoding variable-length quantities. [https://en.wikipedia.org/wiki/Variable-length_quantity](https://en.wikipedia.org/wiki/Variable-length_quantity) VLQs are used in the MIDI file specification. A run of Bytes represents a variable-length integer. The most significant bit of each Byte is a continuation flag - they are all set to 1 except for the flag in the final, least significant Byte, which is set to 0. They are useful when you wish to economically specify a number in a file format. --also when you don't want to place any constraints on how large that number can grow.

## Use
### Basic
`vlq.encode(765);` --> `[133,125]`  
`vlq.decode([133,125]);` --> `765`  
`vlq.decode([0,0]);` --> `false` the continuation bits here are incorrect
### Read Until Valid VLQ
```
var byteA           = [0b10001111,0b11010001,0b00010000,0b00000101];  
var firstValue      = false;  
var firstValueByteA = [];  
while (firstValue === false && byteA.length > 0){  
	firstValueByteA.push(byteA.shift());  
	firstValue = vlq.decode(firstValueByteA);}  
```
`firstValue` --> `256144`  
`byteA` --> `[0b00000101]`  
It's probably better for performance to sequentially check the most significant bit of each element, and only run `decode` once you're certain you have a valid value. Nevertheless, this works and demonstrates the false fail-fast.

## Programmer Configuration Variables
### Scoped Function Collection
The functions and programmer configuration variables belong to the object currently named `vlq`. You may change this to any identifier with no repercussions.
### Encoded Representation, Data Type
`representation` --> one of [`array`,`string`]  
`vlq.representation = "string";`  
`vlq.encode(765);` --> `"}"`  
`vlq.decode("}");` --> `765`  
I prefer array representations - string representations in `encode` and `decode` are handled by intermediately being converted to their equivalent array representations. If you are processing massive amounts of data, you might want to consider the possibility of a string-specific implementation that avoids a full conversion to an array.  
Be careful when using string representations - you will often encounter unprintable characters. When 765 is encoded, it has two characters, though only the second character is printable. In certain environments, unprintable characters may not show up at all leading to potential human-related errors based on misleading observations.

## Miscellaneous
I avoided using bitwise operators on values larger than one Byte. You can therefore use integers larger than 2^32 [current JavaScript limitation of bitwise operators].
