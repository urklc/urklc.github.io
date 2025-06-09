---
layout: post
title: Step-by-Step Guide to Sendable in Swift 6
author: Uğur Kılıç
---

## 1\. Introduction

It all started when this tiny piece of code started to fail after Swift 6 concurrency updates:

```swift
final class MyClass {
  var value: Int = 0
}

func hello() {
  let myInstance = MyClass()
  Task.detached {
    myInstance.value += 1
  }
}
```

The problem with above code is that `MyClass` is not `Sendable` so any instance of it cannot cross _isolation domain boundaries_. Isolation domains are areas that the mutable state is guaranteed to be safe from concurrent access.

After this short introduction I am going to talk about the ways that we can handle the transition to **Swift 6** concurrency safely.

## 2\. Sendable Types

First of all I want to talk about different types and how the most straightforward ways to make them conform to `Sendable`.

1. `Structs` and `enums` (value types) are implicitly `Sendable` when they are composed entirely of `Sendable` value types.

```swift
enum Ripeness { 
  case hard
  case perfect
  case mushy (daysPast: Int)
}

struct Pineapple {
  var weight: Double
  var ripeness: Ripeness
}
```

2. `Reference` types cannot be implicitly `Sendable`. A class must satisfy these:  
   * It must not contain any **mutable state**.  
   * All immutable properties of it must also be `Sendable`.  
   * It must be defined as **final**.  
   * **Cannot inherit** from another class other than NSObject.

```swift
final class Chicken: Sendable {
  let name: String
}
```

Above example hints that a class that conforms to `Sendable` might have been a value type instead. But that is not the case most of the time when we need to preserve value semantics.

3. `Protocols` may have isolation domain definitions. However, the effect of the protocol conformance on isolation depends on how the conformance is applied to that concrete type.

```swift
@MainActor
protocol Feedable {
  func eat(food: Pineapple)
}

// actor isolation applies to the entire Chicken type
class Chicken: Feedable { }

// actor isolation only applies to the extension of Pirate
extension Pirate: Feedable { }
```

## 3\. Main Approaches

Let's see the main approaches to make a type `Sendable` when we encounter problems related to crossing isolation domains.

### **Global Isolation**

```swift
@MainActor
public struct ColorComponents {
  // ...
}
```

Either a **main actor** or any custom **global actor** will make the type implicitly `Sendable` easily. With this way any accesses from other isolation domains must be done asynchronously. Note that above example will execute internal functions on main thread which is still not a problem for most of the cases when we still have the power of the suspension for long running tasks. The critical point here is _the isolation of internal state._

### **Actors**

```swift
actor Style {
  private var background: ColorComponents
}
```

`Actors` are implicitly `Sendable` because of obvious reasons: their state is protected by _actor isolation_. 

Updating a type as an actor is a change to be made with care. Because an actor's isolated public methods and properties will require async context. This might create a _domino effect_ and force you to create lots of `Sendable` conformances during the process to satisfy the callers of the actor.

But as a good thing, actors have their own isolation domain, so they can freely work any **non-Sendable** types internally.

```swift
actor Island {
  var flock: [Chicken] // non-Sendable Chicken
  var food: [Pineapple] // Sendable Pineapple
}
```

### **Manual Synchronization**

The good old ways of manual synchronization is another way. You can use `DispatchQueues`, `Locks` or other forms of manual synchronization to create a safe isolated area yourself. But with manual synchronization we have to make compiler aware of that the current type is `Sendable` with `@unchecked` since there's no mechanism to detect that automatically.

```swift
class Style: @unchecked Sendable {
  private var background: ColorComponents
  private let queue: DispatchQueue
}
```

### **Retroactive Conformance**

Sometimes the source of the problem is not your code, but external dependencies. If you know that such private types are safe for concurrent access, you can mark those `Sendable` with: 

```swift
extension ColorComponents: @retroactive @unchecked Sendable {
}
```

See: [Proposal-0364-retroactive-conformance](https://github.com/apple/swift-evolution/blob/main/proposals/0364-retroactive-conformance.md)

---

> **Combining Techniques**

You can use multiple techniques together to satisfy `Sendable` conformance.

```swift
final class Style: @unchecked Sendable {
  private var background: ColorComponents
  private let queue: DispatchQueue

  @MainActor
  private var foreground: ColorComponents
}
```

In the implementation code, you can think as if _background_ property is protected with the _queue_ and _foreground_ property is isolated by **MainActor**.

### Handling Global Variables

Beyond these examples sometimes we have to deal with global variables.

```swift
var supportedStyleCount = 42
```

The global variable above is both non-isolated and mutable from any isolation domain. It is not safe from concurrent access. We can change it to a **constant** or a **computed variable**. Then since it won't be mutable, the problem will be gone. But instead what if we have to define it as a variable?

1. **Isolate by actor**: This will use the main actor isolation domain to create a safe boundary for the property on _main thread_.

```swift
@MainActor
var supportedStyleCount = 42
```

2. **Manual synchronization**: Similar to `@unchecked`, we are telling the compiler that this property is protected manually by a _queue_ or a _lock_.

```swift
nonisolated(unsafe) var supportedStyleCount = 42
```

---

In cases where the global variable is a reference type instead of a value type:

```swift
class WindowStyler {
  var background: ColorComponents

  static let defaultStyler = WindowStyler()
}
```

Here the problem is the mutable background property of the **non-Sendable** `WindowStyler` type. We are completely fine with static let declaration.

This time we can apply `@MainActor` isolation to the type but another problem arises then:

```swift
@MainActor
class WindowStyler {
  init() { }
}
```

`WindowStyler` type is now `Sendable` and protected by actor isolation, but this means the initializer access also happens in the actor domain. So we cannot just go and initialize the instance globally as:

```swift
struct Stylers {
  static let window = WindowStyler() ❌
}
```

Luckily most of the times globally isolated types don't actually need to reference any mutable state in their initializers. By making the init method `nonisolated`, we allow it to be called from any isolation domain. This is completely safe since only initializer is effected from this change, _actor mutable state is still protected_.

```swift
nonisolated init() { } ✅
```

### Techniques for Crossing Isolation Boundaries

And now to the 2 great ways to allow **non-Sendable** values cross an isolation boundary. Sometimes these are more handy and even appropriate than trying to add a `Sendable` conformance for a type.

1. **Computed Value**

```swift
func updateStyle(colorProvider: @Sendable () -> ColorComponents) async {
  await applyBackground(using: colorProvider)
}
```

Here you can simply use a _Sendable function_ that creates the needed values. So the type doesn't have to be Sendable because the `ColorComponents` instance will be created in the isolated domain which starts at applyBackground already.

2. `sending` **Argument**

```swift
func updateStyle(color: sending ColorComponents) async {
  await applyBackground(color)
}
```

With `sending` argument, you are proving to the compiler that `ColorComponents` **non-Sendable** type is safe to cross boundaries. You guarantee that you won't make any changes to the instance. This works as `@unchecked` keyword we saw earlier to tell a compiler that the type is manually synchronized. Still _if you go against your promise things might start to break_.

## 4\. Region-Based Isolation

[Proposal-0414-region-based-isolation](https://github.com/apple/swift-evolution/blob/main/proposals/0414-region-based-isolation.md)

This is a smart behavior of compiler that will prevent unnecessary `Sendable` conformance requirements. There are situations when a particular instance of a **non-Sendable** type is being used in a safe way. The compiler is capable of inferring this safety through _region-based isolation_.

```swift
func populate(island: Island) async {
  let chicken = Chicken()
  await island.adopt(chicken)
}
```

In this example compiler is smart enough to understand that **non-Sendable** `Chicken` instance is safe to cross isolation domains, it cannot introduce data races. This works automatically.

## 5\. Final Thoughts

While there are several options ensuring the safe crossing isolation boundaries for types, it still _requires a careful thought process_. With existing codebase, it is not a good approach to simply move a type into actor-isolated space or use a `DispatchQueue` to secure it from data races as these often requires updates at call sites and change behavior. Sometimes using `sending` arguments or `computed values` to avoid conforming a type to `Sendable` is a better approach.

Our goal is to write code that is both safe and maintainable. Keep your designs as simple as possible and don't try to over-engineer as this is a complex enough topic already.

---

> ##### _The examples in this post are from **Swift 6 Migration Guide.**_ 