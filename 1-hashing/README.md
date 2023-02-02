
# Hashing

Hashing describes the process of taking arbitrary-sized information and compressing it into fixed-sized values.

This is a one-way road: A Hash can usually not be reverted back into its original value.

## Example

A Hashing function for Names could for example just take the first character (ASCII, 1 Byte) of a name and take its numeric value as a Byte:

<img src="https://c-for-dummies.com/blog/wp-content/uploads/2021/08/0801-figure1.png">

- Marc -> M -> 77
- Ansgar -> A -> 65
- Tom -> T -> 84
- Martha -> M -> 77

## Use-Cases
- Hash-Tables: Used to look up data in O(1)
  - ensures no duplicates
  - allows O(1) removal
  - allows O(1) check for whether sth. is inside
  - Pathfinding (visited Nodes)
  - Reactive Programming (update caching)
- Bloom Filter
- Geometric Hashing: Creating Grids for QuadTrees, Pathfinding, ...

(Variations)
- Checksums: Reduce a whole File to one Number. Used to find Corrupted Data, Blockchain
- Check Digit: Reduces a number to one digit. Used to detect faulty bank account numbers (the last digit is the Check Digit)
- Fingerprint: Reduce a whole data item (e.g. file) to one string. Used to Identify a File, find duplicates etc.
- Lossy Compression: Convert Data from one format to another one, which can be reverted to something close to the original.
- Randomization Function: Generates completely random numbers from slightly changing data (seed + index)
- Error Correction Code: Send some part of the data compressed as a redundant extra information. This redundant data allows the receiver to find Errors and often even fix them himself. e.g. QR-Codes, Live-Streaming
- Cipher: Convert information from one encoding into another to make it hard to reconstruct the original. But it is possible to reconstruct it, if you can reverse engineer the Cipher.

## Idea:
- calculate the location of an item from the item itself
- this creates an ordering system such that
- by looking at an item, you have a very good idea where it is located

## Vocabulary
- associative container
- data consists of ky and value
  - so-called key-value-pairs

Example: key = name, value = phone number

Items can be located by their key only
- hashing is just concerned with storing and finding keys
- the values just happen to be attached to the keys
- some collections, don't even require values

## Hash Function
A function that generates a hash value from the key. Assuming that the data is going to be stored in an Array, we need to generate an Array-Index from the key.

e.g.: Key = Name (`string`) => Hash Function => Hash (`int`)

Examples: 
- Add up all Characters
- Use first Character
- More realistic: Add up all characters, each bit-shifted and multiplied with a prime number

For the upcoming chapters, we will assume that we take a very simple one: The first letter an ASCII Character.
