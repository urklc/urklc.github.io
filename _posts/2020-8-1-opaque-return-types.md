---
layout: post
title: Opaque Return Types in Swift
author: Uğur Kılıç
---

Swift 5 introduces a great feature for people who use protocol oriented programming: **Opaque return types**, which will help you through the problems of creating a protocol with an associated type or self requirements.

You already know the reason of infamous error that constraints you to use protocols that have associated types or Self requirements as generic constraints if you read my previous post: [Using protocols that have associated type requirements as a generic constraint ](2019-12-26-Using%20protocols%20that%20have%20associated%20type%20requirements%20as%20a%20generic%20constraint.md)

Opaque return types help you to overcome this issue. Let's see with a real world example, not a hypotetical one. Let's say we are creating a network api and the Request structure is as follows:

```swift
protocol Request {
    associatedtype ResponseModel

    init()
    func parseResponseModel(data: Data) -> ResponseModel?
}
```

`Request`s will have an associated response model type within their declaration. Let's start with getting the requirement error:

```swift
func createRequest() -> Request { }
```
>**Error:** Protocol 'Request' can only be used as a generic constraint because it has Self or associated type requirements

As I mentioned in the above blog post, we can clearly see the reason of error here: When someone creates a `Request` and makes a call to `parseResponseModel(data: Data)` function, what will be the result value? No one has an idea of it. Function will return an instance conforming to `Request` but it has no use since;

1. We don't have any knowledge about the implementation.  
2. Compiler won't know the returning type either. 

The basic solution lies in the error message. We will use the `Request` protocol as a generic constraint, so the caller will dictate the return type.

```swift
// `Request` as a generic constraint
func createRequest<T: Request>() -> T {
    return T.init()
}

struct LoginRequest: Request {
    func parseResponseModel(data: Data) -> User? {
        return nil // Parse and return response model
    }
}
let loginRequest: LoginRequest = createRequest()
```

Using the generics, we are forcing the caller to set the return type. Now we can use the `loginRequest` instance resolved from any associated type requirements, because it has `User?` as a `ResponseModel` from its implementation.

***Here comes the Opaque Return Types***

Now I will change the use case a bit, because we will create a problem that needs to be solved with opaque types. Let's update our objective to keep our request implementation details private while still returning a `Request` instance.

I will have a network manager that takes `Request` as a parameter and return the associated `ResponseModel` as a result. I know I can't use the `Request` protocol as a function parameter, so again with generics:

```swift
func send<T: Request>(request: T) -> T.ResponseModel? {
    // Some setup and the network call is here
    // No type info, just Request properties (url, method, etc.)
    let data = Data() // Response data
    return request.parseResponseModel(data: data)
}
```

There are 2 possible ways to call this function:  
1. It's completely fine to make this call; but we are leaking the request type which conflicts with our objective.
```swift
let request = LoginRequest()
send(request: request)
```
2. Finally, an opaque type is being returned here: Internal function implementation decides the `Request` instance's type and compiler will know this function only returns a `LoginRequest`. This time -opposite to generics- caller have no idea about the details of return type, but still is able to use it with a generic constrained function. Cool.
```swift
func createLoginRequest() -> some Request {
    return LoginRequest()
}
send(request: createLoginRequest())
```

You can say that *we would achieve the same functionality exposing the `LoginRequest` struct and make use of access modifiers to prevent leaking internal details*. But the best part is here: Let's say you are writing this network mapping for other developers to use and you need to update your `LoginRequest` with another implementation (maybe `Login2Request`) you will just change the return type of `createLoginRequest` function. **Not any extra work for anyone else & not incrementing the major semantic version.**

I personally don't think you will use opaque return types (**SwiftUI** exception) as much as generics, or some other language feature; but especially if you are developing APIs I strongly suggest you to practice more of it for the good of your users.

>For this example, I'm assuming the network functionality will be provided by your external developers. They might use `URLSession` or a 3rd party framework, who knows.
