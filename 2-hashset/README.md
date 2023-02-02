
# HashSet

The following example will look at a certain .NET Collection as an example for Hashing. A HashSet. It utilizes Hashing to Allow for the following functions:

- Insert O(1)
- Search O(1)
- Delete O(1)

- No Order
- No Duplicates (Bug or Feature?)

## Hash Table
Now, we need a Hash-Table. Allocate an array that should be at least 30% larger than the number of expected items.

- It can still be resized later.
- But that costs extra performance.

For the upcoming example, we will assume a Hash table of size 15:

|0|1|2|3|4|5|6|7|8|9|10|11|12|13|14
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|
||||||||||||||||||||||

## Insert

When you want to insert an Item, calculate its HashCode. And then its Hash Table index by using % HashTableSize.

If we want to insert "Marc" into the Hash-Table:

- Key: Marc
- Hash-Function: First Char ASCII (M) -> 77
- % 15 -> 2

We store Marc at Index 2 in Array.

|0|1|>2<|3|4|5|6|7|8|9|10|11|12|13|14
|-|-|:-:|-|-|-|-|-|-|-|-|-|-|-|-|
|||Marc|||||||||||||||||||

If we want to insert "Tom" into the Hash-Table:

- Key: Tom
- Hash-Function: (T) -> 84
- % 15 -> 9

We store Tom at Index 9 in Array.

|0|1|2|3|4|5|6|7|8|>9<|10|11|12|13|14
|-|-|:-:|-|-|-|-|-|-|:-:|-|-|-|-|-|
|||Marc|||||||Tom||||||||||||

## Search

We can find out, whether in Item already is in the Table using the same method:

If we want to check, whether "Nina" is in the Hash-Table:

- Key: Nina
- Hash-Function: (N) -> 78
- % 15 -> 3

|0|1|2|>3<|4|5|6|7|8|9|10|11|12|13|14
|-|-|:-:|-|-|-|-|-|-|:-:|-|-|-|-|-|
|||Marc|||||||Tom||||||||||||

Not found! Nina is not stored in the collection!

Let's see, whether Marc is in the Hash-Table:

- Key: Marc
- Hash-Function: First Char ASCII (M) -> 77
- % 15 -> 2

|0|1|>2<|3|4|5|6|7|8|9|10|11|12|13|14
|-|-|:-:|-|-|-|-|-|-|:-:|-|-|-|-|-|
|||Marc|||||||Tom||||||||||||

Turns out, he is, hurray!

## Hash-Collision

Of course, the Hash-Function cannot deliver an absolutely unique result for every name!
- but it should try to spread keys as evenly as possible

Let's look at a collision. We want to insert Elise into the Hash-Table:

- Key: Elise
- Hash-Function: First Char ASCII (E) -> 69
- % 15 -> 9

|0|1|2|3|4|5|6|7|8|>9<|10|11|12|13|14
|-|-|:-:|-|-|-|-|-|-|:-:|-|-|-|-|-|
|||Marc|||||||Tom||||||||||||

Oh no! It turns out Tom is already blocking Slot 9 of the Array. What now?

## Collision Resolution

We need to find a method for solving Collisions in our Hash-Tables. All of the upcoming methods mean one change, though: We can not rely an on object in a slot being the one that we're looking for just by it being there. We need to also confirm that it is the object that we're actually looking for:

```ts
procedure insert(item){
   hash = item.GetHashCode()
   hash %= buckets.length
   if(hash buckets[hash] == item)
      return // already inside. we're done.
   
   // resolve hash conflict
   // ...
}
```

### Chaining

We could make every slot an Array of Objects. This allows us to store multiple values in one Slot.

Now, if a slot has more than one item, we need to search through all entries of a slot.

This solution is okay, but inefficient and means a lot of extra Memory Allocations (extra Array for each slot, extra Array Buffer, ...)

Pro:
- can postpone resizing for a long time
- very easy to implement

Con:
- like linked list
  - bad cache performance
  - needs more memory

Could also a Binary Tree instead of a list
- only reasonable, if the list becomes very long

### Open Addressing

Store all entries directly in the Hash Table.
- if the slot is already used, just pick another slot using a `Resolution Strategy`

Simple `Resolution Strategy`: use one of the next two slots. If they are both in use as well, resize the table.

Addressing Strategies include:
#### Linear Probing
- use next x slots
- disadvantage: prone to clustering, some areas of the table will be "full' before others

#### Double Hashing
- calculate alternative slot using `Second Hash Function`
- prevents clustering
- disadvantage: bad cache performance, additional hash calculation

#### Quadratic Probing
- calculate alternative slots using quadratic function
- behavior somewhere between Linear Probing and Double Hashing

### Coalesced Hashing
- Combines Chaining and Open Addressing
- Entries stored directly in Table
- But receive index of next entry
- Or `-1` if there is none:

Index|0|1|2|3|4|5|6|7|8|9|10|11|12|13|14
|-|-|-|:-:|:-:|-|-|-|-|-|:-:|:-:|-|-|-|-|
Key|||Marc|||||||Tom||||||||||||
Next|||-1|||||||-1|||||||||||

Now, let's look at a collision again. We want to insert Elise into the Hash-Table:

- Key: Elise
- Hash-Function: First Char ASCII (E) -> 69
- % 15 -> 9


Index|0|1|2|3|4|5|6|7|8|>9<|10|11|12|13|14
|-|-|-|:-:|:-:|-|-|-|-|-|:-:|:-:|-|-|-|-|
Key|||Marc|||||||Tom||||||||||||
Next|||-1|||||||-1|||||||||||

Okay, of course Tom is still here, but let's just use some Addressing Strategy to find a new Address (e.g. 2 stops to the right). And then set the Next-Pointer to the new address:

Index|0|1|2|3|4|5|6|7|8|9|10|>11<|12|13|14
|-|-|-|:-:|:-:|-|-|-|-|-|:-:|:-:|:-:|:-:|-|-|
Key|||Marc|||||||Tom||Elise||||||||||
Next|||-1|||||||11||-1|||||||||

Very efficient and used by .NET!
- But additional memory usage

### Cuckoo Hashing
- Two Hash Function
- Meaning, for every key there are two possible slots

- If a slot is already taken
- Kick out the old entry
  - and put it into its alternative slot
  - if that slot is taken as well...
    - continue kicking that one out etc.

(The European cuckoo does this with eggs)

Very efficient and no Memory Overhead

### Perfect Hashing

Find a perfect Hashing Function that produces no collisions
- whole data set needs to be known
- calculation takes time
- space required to store hash function can reach size of data set
- makes sense only for large data set that does not change often

### Dynamic Perfect Hashing

Perfect Hashing with unknown data set
- possible, but very complicated

## Delete

We've not talked about deleting objects, but it's again the same as Searching: Find it and then remove it if you've found it.

Make sure, though, that if it has a `next`-Pointer, that you move the `next`-Object to the original slot. Maybe, you should only move the last one in the chain of `next`?

## Resizing

Hashing only works well if a certain percentage of the Hash Table is empty

Therefore, it probably has to be resized at some point.

Easiest way: `Rehashing`
- allocate new, larger hash table
- insert all elements into new Table
- not a copy operation: hash value will change
  - 57%15 = 12, but 57%30 = 27!

## Hash Function and Table Size
A Good Hash Function
- produces little collisions
- distributes entries evenly across table
- is fast to compute

Table Size should be a prime Number
- has no dividers
- prevents cycles and clustering

## Summary
Pro:
- Often much faster than binary search

Con:
- uses more memory, always wastes memory
- hash calculation has some cpu overhead that only pays off if there's enough entries

## Useful Example

This is very useful in order to collect all items with a shared meaning in a collection and quickly find out, whether an object is in there.

This basically allows you to add a `bool`-Property to a class that you can't change.

Imagine the following class in a Strategy Game with Fog of War:

```ts
class Building
   position: Vector2
```

We want to know, which buildings are known to a player. If a player happens to find that building, we want to remember that the player has seen this building. So, each time he finds it, we want to make sure that it's already (and still) in the collection.

We could make this change;

```ts
class Building
   position: Vector2
   seenByPlayer: bool
end class

procedure OnBuildingSeen(building)
   seenByPlayer = true
end procedure

procedure CheckBuildingSeen(building)
   return building.seenByPlayer
end procedure
```

But then again, is it really the Building's Property? What if there's multiple players?

We could instead do this:


```ts
class Player
   seenBuildings: HashSet<Building>
end class

procedure OnBuildingSeen(building, player)
   player.seenBuildings.Insert(building)
end procedure

procedure CheckBuildingSeen(building, player)
   return seenBuildings.Contains(building)
end procedure
```

Can you see, how the Attribute:
- each Building has a `bool` which is `true` or `false`

Is sort of the same as:
- each Building is in a collection `true` or not `false`

And allows us to somewhat "extend" an existing class without changing it?
