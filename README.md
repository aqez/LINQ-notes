---
title: LINQ
author: Tony Dwire
date: 2021-02-03
---


                    _                       _  _  _           _           _             _  _  _  _
                   (_)                     (_)(_)(_)         (_) _       (_)          _(_)(_)(_)(_)_
                   (_)                        (_)            (_)(_)_     (_)         (_)          (_)
                   (_)                        (_)            (_)  (_)_   (_)         (_)          (_)
                   (_)                        (_)            (_)    (_)_ (_)         (_)     _    (_)
                   (_)                        (_)            (_)      (_)(_)         (_)    (_) _ (_)
                   (_) _  _  _  _           _ (_) _          (_)         (_)         (_)_  _  _(_) _
                   (_)(_)(_)(_)(_)         (_)(_)(_)         (_)         (_)           (_)(_)(_)  (_)


- What is LINQ

- Before and After LINQ

- Using complex LINQ

- Writing our own LINQ-style extension methods


---

# What is LINQ?

Linq is a collection of interfaces and methods in the `System.Linq` namespace that make it easy to work with collections of data.

- LINQ stands for Language Integrated Query
- Many extension methods for `IEnumerable<T>` interface
- Also introduces some other interfaces, such as `IGrouping`, `IQueryable`, etc.
- All of these interfaces implement `IEnumerable<T>` so we will stick to looking at that for this video
- Extremely useful for working with collections - maybe the most useful collection library in existence


There are two main ways of using the LINQ functionality. One is via the query language:

```csharp
var numbers = new int[] { 1, 2, 3, 4, 5 };

var odd =
    from number in numbers
    where number % 2 == 0
    select number;

Console.WriteLine(odd); // { 1, 3, 5 }
```


The other is via the extension methods that the `System.Linq` namespace provides:
```csharp
var numbers = new int[] { 1, 2, 3, 4, 5 };

var odd = numbers.Where(n => n % 2 == 0);

Console.WriteLine(odd); // { 1, 3, 5 }
```

---

# IEnumerable<T>

`IEnumerable<T>` is a forward iterating, read only interface over a collection of `T`'s


## Example IEnumerable<int>
```csharp
int[] numbers = new int[] { 1, 2, 3, 4, 5 };

IEnumerable<int> enumerable = numbers; // This gives us a way to look at numbers in a forward iterating, read-only way

// Alternatively, we could write it like this:
IEnumerable<int> enumerable = new int[] { 1, 2, 3, 4, 5 };
```


## Sum

This may sound limiting, but what we can do with this concept goes very deep. For instance, we can get a sum:

```csharp
IEnumerable<int> enumerable = new int[] { 1, 2, 3, 4, 5 };

int sum = 0;
var enumerator = enumerable.GetEnumerator();

while(enumerator.MoveNext())
{
    sum += enumerator.Current;
}

Assert.Equal(15, sum);
```

---

# LINQ methods

The extension methods in the `System.Linq` namespace give us a set of easy to perform, nicely wrapped common operations that we can use on collections implementing `IEnumerable<T>` that we do not have to write ourselves. In this, and the next few slides we will go over the more common ones and the way they let us transform our code.

As an example of what we will be looking for, they can transform our code from something like this:
### For Each
```csharp
IEnumerable<Person> allPeople = GetSomePeople();
List<string> fourOrMoreLetters = new List<string>();
foreach(var person in allPeople)
{
    if(person.Name.Length > 4)
    {
        fourOrMoreLetters.Add(person.Name);
    }
}
```

To code that looks like this:
### LINQ
```csharp
var fourOrMoreLetters = GetSomePeople().Select(person => person.Name).Where(name => name.Length > 4);
```
---

# Where
`Where` is probably the most used extension method for me personally. It is used to filter the source and produce a new `IEnumerable<T>` that only produces elements that pass the predicate's test.

Given the following class and array:
```csharp
public class Person
{
    public string Name { get; set; }
    public string Age { get; set; }
}

var people = new Person[]
{
    new Person { Name = "Bob", Age = 30 },
    new Person { Name = "Jack", Age = 50 },
    new Person { Name = "Beth", Age = 42 }
};
```

### Get all of the people with a name starting with "B"

#### Before
```csharp
var startsWithB = new List<Person>();
foreach(var person in people)
{
    if(person.Name.StartsWith("B"))
    {
        startsWithB.Add(person);
    }
}
```

#### After
```csharp
var startsWithB = people.Where(person => person.Name.StartsWith("B"));
```

---

# Select
`Select` is the second most common one for me. It is 'map' from other languages. It is used to take one sequence of elements and turn it into another sequence of elements, one for each input.

### Given a collection of `Person`, select their names

#### Before
```csharp
var names = new List<string>();
foreach(var person in people)
{
    names.Add(person.Name);
}
```

#### After
```csharp
var names = people.Select(person => person.Name);
```


It is also important to note that the input and output of this method do not have to be particularly related, you could return a random number for each one, for instance. 

### Select double the age of each `Person`
```csharp
var doubledAges = people.Select(person => person.Age * 2); // [ 60, 100, 84 ]
```

### Select the third letter of each `Person`'s name
```csharp
var thirdLetters = people.Select(person => person.Name[2]); // [ 'b', 'c', 't' ]
```

---

# Any, All

The `Any` and `All` extension methods determine if any or all of the elements satisfy some predicate.

### Determine if all the people are under age 30

#### Before
```csharp
bool all30OrUnder = true;
foreach(var person in people)
{
    if(person.Age > 30)
    {
        all30OrUnder = false;
        break;
    }
}
```

#### After
```csharp
// Using `All`
bool all30OrUnder = people.All(person => person.Age <= 30);

// Using `Any`
bool all30OrUnder = people.Any(person => person.Age > 30);
```


---

# Min, Max, Average, Sum

These extension methods provide an easy way to get the min, max, average or sum of some predicate over a collection. They have 'default predicate' implementations for the built in numeric types, and other types can specify their own predicate.

### Given a list of people, get the average age

#### Before
```csharp
int sum = 0;
int count = 0;

foreach(var person in people)
{
    sum += person.Age;
    count += 1;
}

int average = sum / count;
```

#### After
```csharp
double average = people.Average(person => person.Age);
```

Note, for collections of numeric builtin types, you don't have to supply a predicate.

```csharp
// Don't do this:
double average = (new int[] { 1, 2, 3, 4, 5 }).Average(x => x);

// Instead do this:
double average = (new int[] { 1, 2, 3, 4, 5 }).Average();
```

---

# First, FirstOrDefault, Last, LastOrDefault, Count

These give simple access to the beginning and ending elements of a collection as well as counting items in a collection. They can also be given a predicate to determine the first/last/count where some condition is true.


### Get the last `Person` in the collection where the person's name starts with 'B'


#### Before
```csharp
Person lastStartingWithB = null;
foreach(var person in people)
{
    if(person.Name.StartsWith("B"))
    {
        lastStartingWithB = person;
    }
}
```

#### After
```csharp
var last = people.LastOrDefault(person => person.Name.StartsWith("B"));
```

### Count the people who have more than 5 letters in their name

#### Before
```csharp
int moreThan5Letters = 0;
foreach(var person in people)
{
    if(person.Name.Length > 5)
    {
        moreThan5Letters += 1;
    }
}
```

#### After
```csharp
int moreThan5Letters = people.Count(person => person.Name.Length > 5);
```

---

# Cast, OfType

`Cast` and `OfType` are similar but not exactly the same. `Cast` will go through each item in a collection and try to cast it to the destination type, and if it fails it will throw an exception. `OfType` will go through each item in a collection, determine if it can be casted, and return only those that can be casted as the destination type.

### Given a list of `Vehicle`s, turn them all into `Truck`s

#### Before
```csharp
List<Truck> trucks = new List<Truck>();
foreach(Vehicle vehicle in vehicles)
{
    trucks.Add((Truck)vehicle);
}
```

#### After
```csharp
List<Truck> trucks = vehicles.Cast<Truck>();
```


### Given a list of `Vehicle`s, get the `Helicopter`s
#### Before
```csharp
List<Helicopter> helicopters = new List<Helicopter>();
foreach(Vehicle vehicle in vehicles)
{
    if(vehicle is Helicopter helicopter)
    {
        helicopters.Add(helicopter);
    }
}
```

#### After
```csharp
List<Helicopter> = vehicles.OfType<Helicopter>();
```

---

# OrderBy, OrderByDescending

`OrderBy` and `OrderByDescending` allow for easy sorting of collections with a specified predicate that returns the thing to be sorted by.

### Sort a collection of `Person` by age

#### Before
```csharp
List<Person> orderedByAge = new List<Person>();

foreach (var person in people)
{
    int insertIndex = 0;
    for (int i = 0; i < orderedByAge.Count; i++)
    {
        if (person.Age <= orderedByAge[i].Age)
        {
            break;
        }
        insertIndex += 1;
    }
    orderedByAge.Insert(insertIndex, person);
}
```

#### After
```csharp
List<Person> orderedByAge = people.OrderBy(person => person.Age);
```

---

# GroupBy
Allows easy grouping of items based on some 'key'. 


### Group people by the first letter of their name

#### Before

```csharp
Dictionary<char, List<Person>> grouped = new Dictionary<char, List<Person>>();

foreach(var person in people)
{
    if(!grouped.ContainsKey(person.Name[0]))
    {
        grouped.Add(person.Name[0], new List<Person>());
    }

    grouped[person.Name[0]].Add(person);
}

```

#### After
```csharp
var grouped = people.GroupBy(person => person.Name[0]);
```

---

# Combining LINQ methods to get stuff done


What if we wanted to get the letter of the alphabet with the oldest (on average) people that have that letter as the first letter of their name?

Workflow would be..

1. Group everyone by the first letter of their name
2. Average the ages for each group
3. Order the groups by the average age
4. Take the first one

```csharp
var people = new Person[]
{
    new Person { Name = "Tony", Age = 32 },
    new Person { Name = "Tabatha", Age = 48 },
    new Person { Name = "Bob", Age = 38 }
};

var oldestAverageLetter = people.GroupBy(person => person.Name[0])
                                .Select(group => new { Letter = group.Key, SummedAges = group.Average(person => person.Age) })
                                .OrderByDescending(group => group.AverageAge)
                                .First();

Console.WriteLine(highestLetterAndAge); // { Letter = "T", AverageAge = 40 }
```

---

---
# Predicates as Separate Methods

The predicates we have seen so far have all been defined as lambdas. This is not the only way to do it though. Anything that can be used as a delegate matching the required type for the LINQ method can be used.

```csharp
var youngEnoughPeople = people.Where(IsYoungEnough);


bool IsYoungEnough(Person person)
{
    return person.Age < 20;
}

```


```csharp
Func<Person, bool> oldEnough = person => person.Age > 100;

var oldEnoughPeople = people.Where(oldEnough);
```



