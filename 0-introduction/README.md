# Firstly, some Code:

Before I start boring you with all the details, let's first look at some code to demonstrate, what the upcoming technologies will enable us to do:

## HashSet

Imagine having a Graph of Nodes where each Node knows its neighbors.

We want to create a List of all Nodes by starting at one Node and walking all of its neighbors and so on until all Nodes have been collected.

But we need to make sure that we don't walk in Circles.
- e.g. A -> B -> A -> B -> A -> B -> ...
- or A -> B -> C -> A -> B -> C -> ...

Let's use a `HashSet` to keep track of what nodes we have walked already and not walk that node again. A `HashSet` does not allow duplicate values and will friendly remind us, if we inserted something twice:

```cs
public class Node {
   public Node[] neighbors;
}
```

```cs
public HashSet<Node> WalkAllNodes(Node startNode){
   // all nodes that we want to visit
   Queue<Node> toWalk = new Queue<Node>(){startNode};
   // all nodes that we have found
   HashSet<Node> visited = new HashSet<Node>{startNode};

   while(toWalk.Count > 0){
      var current = toWalk.Dequeue();
      foreach(var neighbor in node.neighbors){
         if(visited.Add(neighbor)){
            // returns true only, if the node was not inside already. A HashSet cannot have duplicates.
            // Which means, that we have a new node
            toWalk.Enqueue(neighbor);
         }
      }
   }

   return visited;
}
```

Have you noticed, that we did not need a `public bool visited;`-Field on the `Node`?

## HashMap

And now, we want to imagine having a ScoreBoard for all Players of a BattleRoyale-Match. A Dictionary (HashMap) allows us to store and retrieve one type of Value/information (the score) associated by its Key (the player):

```cs
public class Player{}
```

```cs
public class ScoreBoard {
   public Dictionary<Player, int> scores;

   public void AddScore(Player player, int score){
      scores[player] += score;
   }

   public int GetScore(Player player){
      return scores[player];
   }

   public int PrintAllScores(){
      // Using some LINQ Magic here to order the Pairs of Key and Value by their score, large to small
      foreach(var score in scores.OrderByDescending(kv => kv.score)){
         Console.WriteLine($"Player {score.Key} has Score: {score.Value}");
      }
   }
}
```

## Why not Both?

And now an example, in which we register all spawned buildings in the GameController, which groups the Buildings by their BuildingType. This allows our EvilBoi to pick a GoldStash as a target without needing to search through all buildings.

Even for a million buildings, this would find the 12 GoldStashes without breaking a sweat!

First, the `BuildingType`; a simple enum:
```cs
public enum BuildingType
{
   Tower,
   Bunker,
   GoldStash
}
```

Now, the `EvilBoi`, he'd like to find all buildings of type `BuildingType.GoldStash`. That must be expensive! D:
```cs
public class EvilBoi
{
   public Building target;
   void Update()
   {
      if (target == null)
      {
            var goldStashes = GameController.instance.AllBuildingsOfType(BuildingType.GoldStash);
            // TODO: Attack!
      }
   }
}
```

Okay, the `Building` has a `BuildingType` and registers itself with the `GameController`. Nothing special here.
```cs
public class Building
{
   // If the BuildingType can change, you need to Remove if (before changing) and Add it again (after changing)!
   public BuildingType type;
   void OnEnable()
   {
      GameController.instance.AddBuilding(this);
   }

   void OnDisable()
   {
      // TODO: Remove Again!
   }
}
```

The `GameController` is the class that does the magic: It uses a Dictionary in Combination with a HashSet to keep track of all buildings, grouped by their BuildingType:
```cs
public class GameController
{
   // TODO: Fix Evil Singleton!
   public static GameController instance;
   private Dictionary<BuildingType, HashSet<Building>> allBuildings = new ();

   public void AddBuilding(Building building)
   {
      if (!allBuildings.TryGetValue(building.type, out var buildings))
      {
         // if there's no building of this type, yet
         // we need to add a new collection for this type
         buildings = new HashSet<Building>();
         allBuildings.Add(building.type, buildings);
      }

      // add the building to the new or existing collection
      buildings.Add(building);
   }

   public IEnumerable<Building> AllBuildingsOfType(BuildingType buildingType)
   {
      allBuildings.TryGetValue(buildingType, out var buildings);
      // make sure, not to return null here
      // rather, return an empty array
      // officially saying: Count: 0
      return buildings ?? Array.Empty<Building>();
   }
}
```

Now, that you've understood, how impressive these collections can be, let's look at with what intention they were developed, what problems were faced on their way and what strategies can be used to overcome the issues:

# Introduction

If we have a Collection of a lot of items and we need to find items very often.
- e.g. because we need to remove them again
- or because we want to check, whether a certain item is already inside

Then a Linear Search has catastrophic performance: `O(n)`

Also, a Binary Search might not be so nice
- The complexity of `O(log n)` might be not good enough (even though it's extremely good)
- Or the items might not be clearly orderable (e.g. what order do you put Colors in? Or Transforms?)

# Inspiration: Static Array

In many ways, Static Arrays are the ideal structure performance-wise.
- If you know the index of the item that you want... e.g. 20
- Then you just receive it: `int item = array[20];`

Similar to how in an excel sheet you can quite quickly find row 22, since you know that every row is 5mm high, it should be at ca. 11cm distance to the top of the screen.

This is very efficient for your RAM. If you know the exact address of an object, it can directly jump there and read the value.

That's why Database-Tables love to use Numeric Primary Indices: Every Customer gets a Unique, Auto-Incremented id. And if Customer #2000 wants to place an order, the Database won't have to start searching, it knows exactly where the Customer-Information is stored and can directly look at entry #2000 without breaking a sweat.

But would it be nice to work with numeric IDs in our application all the time?

```cs
// check, whether the unit is queued to be spawned:
if(queuedUnits[unit.id] != null)
```

Also, what if we only need to store the Units #7987-#8217 in a small collection for a while?

|0|1|2|500|...|5000|...|7987|7988|...|8217|8218|...
|-|-|-|---|---|----|---|----|----|---|----|----|---
| | | |   |   |    |   |Unit|Unit|Unit|Unit|Unit|

That looks like a lot of wasted memory just to allow instant lookup. More than 97%...

# Generating a Unique Number for the Data

We could generate unique numbers for each possible information.

Let's play this through for Names:

e.g. 
- Letter A = 0
- Letter B = 1
- Letter C = 2
- ... 
- Letter Z = 23

And then convert each name to a number, e.g.:
- A = 1
- B = 2
- B = 2
- A = 2

We could just add these numbers and use them as an ID:
- A + B + B + A
- = 1 + 2 + 2 + 1
- = 6

Problem: Collisions:
- B + A + B + A
- = 2 + 1 + 2 + 1
- = 6

That's not nice, ABBA and BABA are not the same thing at all!

To allow for all kinds of combinations, e.g.:
- AAAA
- ABBA
- BABA
- ZZZZ

We'd have to "invent" a 24-based numeric system:
- A = 1 * 24^0 = 1
- B = 2 * 24^1 = 48
- B = 2 * 24^2 = 1152
- A = 1 * 24^2 = 13824
- = 15025

That's quite a big number already. 

`Maximilian Mustermann` would already need 24^20 = a number with 27 digits to create a unique index.

And what about all the special characters? UTF-16 allows 16 bit = 65k different combinations for each character.

=> Not possible.

This process would actually rather be called serialization or loss-free compression (a bad one at that). By using some specific rules, we are able to transform a name into a numeric format which we can store byte-by-byte on the Hard Drive and load it without problems again.

# Almost Unique might be enough

What if we were okay with the possibility of Overlaps between names like ABBA and BABA, though? Maybe, we could store all information in the array, but allow multiple items to be in the same place? An Array containing arrays so to say?

|0|1|2|...|6|-|
|-|-|-|---|-|-|
| | | |   |ABBA, BABA| |

I mean sure, it might require us to look at a second item in the rare case of a "collision", but 2 instead of 1 is still much better than searching through all names, right?
