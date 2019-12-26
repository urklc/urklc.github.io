---
layout: post
title: Using protocols that have associated type requirements as a generic constraint 
author: Uğur Kılıç
---

Whenever you go beyond classical protocol oriented programming practices, and use protocols for a variety of functionality across your code; you face this noted error with Swift. Next, you suddenly change most of your implementation or try to workaround it by searching solutions online.

I guess understanding the reasoning behind this error leads to better software design before implementation. Let's say we have a `Comparable` protocol as: 

```swift
protocol Comparable {
    func compare(other: Self) -> Bool
}
```

`Self` is an associated type. Associated types (as an instance variable or a function argument) are decided by the implementation that associates with them. `Comparable` protocol has appearently **Self (or associated type) requirement**. You cannot have a direct reference to it and call its functions.

To understand why, we will create `Animal` and `Person` structs conforming the protocol:

```swift
struct Animal: Comparable {
    let age: Int

    func compare(other: Self) -> Bool {
        return age > other.age
    }
}

struct Person: Comparable {
    let name: String

    func compare(other: Person) -> Bool {
        return name > other.name
    }
}
```

Animals are being compared by their ages and Person instances are being compared by alphabetical positions. 
Totally makes sense for our example. Finally if you want to write a function to sort any `Comparable` list with comparing them:

```swift
struct Utility {
    static func sortedComparables(_ comparables: [Comparable]) -> [Comparable] {
        return comparables.sorted {
            return $0.compare(other: $1)
        }
    }
}
```

Here we get a compiler error as **error: protocol 'Comparable' can only be used as a generic constraint because it has Self or associated type requirements**. Although it's not visible at first look, the reason of this error is that if you have a reference to an object via a `Comparable` pointer instance, you will have an opportunity to call its functions.

Within the sort function let's take the first element in array to operate on it:

```swift
let firstComparable = comparables.first

// What `Comparable` instance can you pass as an argument to this function? 
// It has to be guaranteed to compare two instances with same 
// concrete type (`Animal` or `Person`). There's no chance to have it here!
firstComparable.compare(...)
```

Any implementation that results in a reference pointer of a `Comparable` instance directly will produce this error.

### Solution

See the first section of the error: **protocol 'Comparable' can only be used as a generic constraint**. Generics allow us to decide the function/class parameters at function call time. Thus we will be able to force any function parameter to a specific concrete type. Let's fix the issue in `Utility` function:

```swift
struct Utility {
    static func sortedComparables<T: Comparable>(_ comparables: [T]) -> [T] {
        return comparables.sorted {
            return $0.compare(other: $1)
        }
    }
}
```

Here whenever you make a call to `sortedComparables` function, compiler will force you to pass an array of `Comparables` with the same concrete type: only `Animal` or only `Person` list. Now we are using `Comparable` as a generic constraint and satisfy the compiler.
