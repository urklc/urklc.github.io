---
layout: post
title: Type Erasure
author: Uğur Kılıç
---

I like writing about generic protocol types, and you like reading them. But now with another fancy concept **Type Erasure** I want to end current series. If you haven't read them yet, here are the links:

1- [Using protocols that have associated type requirements as a generic constraint](../Using-protocols-that-have-associated-type-requirements-as-a-generic-constraint/)  
2- [Opaque return types](../opaque-return-types/)

If you understand the problems and solutions in first two posts, type erasure will be very easy to get familiar with.  

Let's start coding for the problem that we will solve with type erasure. First, we will create different types to use them on a generic protocol as an associated type:

```swift
protocol Currency {
    var amount: Double { get }
}

struct Gold: Currency {
    let amount: Double
}

struct Dollar: Currency {
    let amount: Double
}
```

Then, we will create an `Account` protocol and related implementations as follows:

```swift
protocol Account {
    associatedtype AccountCurrency
    var deposit: AccountCurrency { get }
}

struct GoldAccount: Account {
    let deposit: Gold
}

struct DollarAccount: Account {
    let deposit: Dollar
}
```

Great! `Account` structs seems like overkill, but think about how a real world implementation can make these structs more complicated. We will only need such few properties to create a better understanding of type erasure.

This time I want to create an array of `Account`s for a customer and calculate how much gold that customer has:

```swift
let accounts = [GoldAccount(deposit: Gold(amount: 250)),
                GoldAccount(deposit: Gold(amount: 150))]
let totalGold = accounts
    .filter { $0.deposit is Gold }
    .reduce(0) { $0 + $1.deposit.amount }
print(totalGold) // prints 400.0
```

Everything seems fine. I created an array of `GoldAccount`s and filtered them with their `deposit` type, then calculated the sum of filtered values. The problem with this code is that `accounts` property is inferred as a `[GoldAccount]` array. That means we haven't yet implemented a generic usage for multiple `Account` types. Let's proceed with adding a `DollarAccount` to our records:

```swift
let accounts = [GoldAccount(deposit: Gold(amount: 250)),
                GoldAccount(deposit: Gold(amount: 150)),
                DollarAccount(deposit: Dollar(amount: 50))]]

```
***`Error`: Heterogeneous collection literal could only be inferred to `[Any]`; add explicit type annotation if this is intentional Insert `as [Any]`.***

The error says it all: We have a heterogeneous array and it will be inferred as `[Any]` array. Compiler wants us to show our intention with marking the return value as `[Any]` array. But this time since we want to filter gold accounts from the array, our calculation will not work on `[Any]`:

```swift
let totalGold = accounts
    .filter { $0.deposit is Gold } // `Any` has no member `deposit`
    .reduce(0) { $0 + $1.deposit.amount } // `Any` has no member `deposit`
```

We can conclude that our intention was wrong, we actually need an array with `Account` instances, not `Any`.

***Here comes Type Erasure***

```swift
struct AnyAccount {
    let amount: Double
    let deposit: Currency
    init<T: Account>(_ account: T) where T.AccountCurrency: Currency {
        amount = account.deposit.amount
        deposit = account.deposit
    }
}
```

This is the helper struct we will use to wrap generic protocols and hide their unwanted details. `AnyAccount` can be initialized with an `Account` argument as a generic constraint. We also constrain the `Account`'s (`T` in this case) `AccountCurrency` type to a `Currency` which we will use for filtering.

And the final piece of code is a little messy, but we are dealing with protocols with associated types. So, workarounds deserve to be messy:

```swift
let accounts = [
    AnyAccount(GoldAccount(deposit: Gold(amount: 100.0))),
    AnyAccount(GoldAccount(deposit: Gold(amount: 150.0))),
    AnyAccount(DollarAccount(deposit: Dollar(amount: 50.0)))
]
let totalGold = accounts
    .filter { $0.deposit is Gold }
    .reduce(0) { $0 + $1.deposit.amount }
```

When you want to work with heterogeneous arrays which contain generic types, you can use this method. But keep in mind that you can still cast the filtered array to `[GoldAccount]` and make specific calls to its `GoldAccount` elements.