# 205 Hashing

## Introduction

If we have a Collection of a lot of items and we need to find items very often.
- e.g. because we need to remove them again
- or because we want to check, whether a certain item is already inside

Then a Linear Search has catastrophic performance: `O(n)`

Also, a Binary Search might not be so nice
- The complexity of `O(log n)` might be not good enough (even though it's extremely good)
- Or the items might not be orderable (e.g. what order do you put Colors in? Or Transforms?)

## Problem
- We need to find an item in a large, unsorted collection.

## Use Case
- Pathfinding: Keep Track of visited nodes, so we don't run in circles
- Grouping: Grouping objects by common traits
- Identifying: Finding an Object through its ID

## Passing Criteria
Write Unit Tests for .NET's `HashSet` and `Dictionary` classes.

Features:
- HashSet
  - You can insert items `Insert`
    - It returns true, if the item was not inside already
    - It returns false, if the item was inside already
  - You can check, whether an item is inside `Contains`
    - It returns true, if the item is inside
    - It returns false, if the item is not inside
  - You can remove items `Remove`
    - It returns true, if the item was inside
    - After removing, it should not be inside anymore
    - It returns false, if the item was not inside
  - You can enumerate the collection to get all items that are inside

## Excellent Criteria
Guess what? Implement a `TurboHashSet<T>`.

You will need:
- `bool object.Equals(object other)`
- `int object.GetHashCode()`

Which conveniently are available on ALL classes in .NET.

A Collision Resolution Strategy.
- Might want to go for the most "stupid" one for now: try the next three slots

A Buffer Size Strategy. 
- Might want to go for 30% extra Space, as mentioned earlier

The following Methods:
- `bool Insert(T item)` returns `false`, if it was already inside, else it adds it and returns `true`
- `bool Exists(T item)` returns true, if it is inside
- `bool Remove(T item)` returns `false`, if it was not inside, else it removes it and returns `true`

Internal Method:
- `Resize` which will increase the buffer size, if necessary (Bonus)

## Bonus (Hard)
- Coalesced Hashing

## Bonus (Intermediate)
- Write your own class with a Hashing-Function