
# HashMap

A HashMap is very similar to a HashSet, but while a HashSet only allows unique values, the HashMap allows duplicate values. How does it do that? Let's return to our phone book example:

Index|0|1|2|3|4|5|6|7|8|9|10|>11<|12|13|14
|-|-|-|:-:|:-:|-|-|-|-|-|:-:|:-:|:-:|:-:|-|-|
Key||||Marc||||||Tom||Elise||||||||||
Value||||070847780||||||0781717122||189174748||||||||||
Next||||-1||||||11||-1|||||||||

In other words: it stores Keys AND Values for each entry. This allows you to map ONE key to ONE Object. Since multiple keys can map to the same Object, you can argue that you can map N keys to ONE Object:

Index|0|1|2|3|4|5|6|7|8|9|10|>11<|12|13|14
|-|-|-|:-:|:-:|-|-|-|-|-|:-:|:-:|:-:|:-:|-|-|
KeyCode||||KeyCode.Ctrl||||||KeyCode.Space||KeyCode.Alt||||||||||
Value||||Fire||||||Fire||Fire||||||||||
Next||||-1||||||11||-1|||||||||

This does not only mean, that we can have duplicate Values, but also, that the key and the value can be two completely different things.

## Operations:

- Insert `O(1)`
- Update `O(1)`
- Search `O(1)`
- Remove `O(1)`

## Use-Cases:

Whenever you want to look up one value (or multiple, by making the value a List) by some key:
- The player to a certain ID
- An Action to a certain input
- GameObjects to a certain Tag
- GameObjects to a certain component
- Event Listeners to a certain Event Type
- All players in a certain league

## Dictionary

In C#, the Generic version of this collection is called a `Dictionary<TKey, TValue>` and it can be used like this:

```ts
class Player
   league: id
   clan: id

playersByLeague: Dictionary<id, player>
playersByClan: Dictionary<id, player>

procedure OnPlayerLoaded(player)
   if(not playersByLeague.hasKey(player.league))
      list = new List<player>
      list.Add(player)
      playersByLeague.Add(key: player.league, value: list
   else
      list = playersByLeague.Get(key: player.league)
      list.Add(player)
   end if
end procedure
```

## Useful Example

Again, we can extend a class this way. And not only by a `bool`-Value, but by any value:

Can you see, how the Attribute:
- each Player has a field of type `T` which has any value of type `T`

Is sort of the same as:
- each Player can be the key in a collection with a value of type `T`

And allows us to somewhat "extend" an existing class without changing it?
