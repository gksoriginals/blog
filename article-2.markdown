# Result: The Swift Standard Library's Missing Type

The Swift Standard Library has a type called `Optional`.

```swift
enum Optional<Wrapped> {
    case none
    case some(Wrapped)
}
```

`Optional` is an enumeration with two cases, `some` and `none`, where `some` has an associated value which can be of `Any` type.

The purpose of `Optional` is to be able to *explicitly* declare that a value may or may not exist.

The compiler can then validate your use of the value and emit an error if you fail to check whether the value exists before it is used.

When `Optional` is used as a function return value, it expresses that the function may not return a value, which is an *implicit* expression of failure.

```swift
func parse(string: String) -> Optional<Int>
```

If we have a function, `parse`, which has a `String` parameter (input), and returns an `Optional<Int>` (output), we can intuit that the function may fail to return an output, if the input is invalid.

If we wanted to *explicitly* express a failure, or express a failure not of the input, we could `throw` an error instead.

```swift
func parse(string: String) throws -> Int
```

This approach is perfect for a synchronous function, but doesn't work for an asynchronous function, as we must pass the error to the callback function as a parameter, rather than throwing it.

<hr>

A synchronous function is one that returns a value immediately. An asynchronous function is one that returns a value in the future, typically through use of a function parameter, often referred to as a callback.

```swift
// synchronous
func foo() -> String
// asynchronous
func foo(callback: (String) -> Void)
```

<hr>

```swift
func foo(callback: (Optional<String>, Optional<Error>) -> Void)
```

If we were to declare our callback function to accept two parameters, `Optional<String>` and `Optional<Error>`, we would end up with four possible outcomes.

1. `Optional<String> == some, Optional<Error> == some`
2. `Optional<String> == some, Optional<Error> == none`
3. `Optional<String> == none, Optional<Error> == some`
4. `Optional<String> == none, Optional<Error> == none`

We, obviously, only want two possible outcomes: success or failure.

## Result

So, let's follow the lead of `Optional`...

```
enum Result<Wrapped> {
    case success(Wrapped)
    case failure(Error)
}
```

`Result` is an enumeration with two cases, `success` and `failure`, which both have associated values, `Any` and `Error`, respectively.

```swift
func foo(callback: (Result<String>) -> Void)
```

We can now declare our callback function as returning a `Result<String>`, which perfectly expresses the two possible outcomes of our function `foo`.

## Result in Use

So, let's say we have a function `foo`, which has a `URL` parameter, and a callback function parameter which accepts a `Result<Data>`. The purpose of this function is to fetch `Data` from a `URL`.

```swift
func foo(url: URL, callback: @escaping (Result<Data>) -> Void) {
    let task = URLSession.shared.dataTask(with: url) { data, response, error in
        guard let data = data, data.count > 0, let response = response as? HTTPURLResponse, response.statusCode == 200, error == nil else {
            callback(.failure(error ?? URLError(.unknown)))
            return
        }
        callback(.success(data))
    }
    task.resume()
}
```

This approach works, but is a little cumbersome. Firstly, we have to execute `callback` mutiple times, and secondly, we have to explicitly deal with the `Result` cases.

```swift
extension Result {
    init(wrap: () throws -> Wrapped) {
        do {
            let wrapped = try wrap()
            self = .success(wrapped)
        }
        catch {
            self = .failure(error)
        }
    }
}
```

If we extend `Result`, we can create an initializer that accepts a function which we can invoke inside a `do-catch` expression, which is how the code would have been structured had we been writing synchronous code.

```swift
func foo(url: URL, completion: @escaping (Result<Data>) -> Void) {
    let task = URLSession.shared.dataTask(with: url) { data, response, error in
        let result = Result<Data>(wrap: {
            guard let data = data, data.count > 0, let response = response as? HTTPURLResponse, response.statusCode == 200, error == nil else {
                throw error ?? URLError(.unknown)
            }
            return data
        })
        completion(result)
    }
    task.resume()
}
```

So, now we have eliminated our multiple invokations of the callback function, and our code is much more natural, using `throw` and `return` within the `Result` initializer parameter function.

We can even write the code in a much more succinct way, by eliminating the unnecessary parts of the initializer and closure syntax, and allowing Swift to use its type inference.

```swift
func foo(url: URL, completion: @escaping (Result<Data>) -> Void) {
    let task = URLSession.shared.dataTask(with: url) { data, response, error in
        completion(Result {
            guard let data = data, data.count > 0, let response = response as? HTTPURLResponse, response.statusCode == 200, error == nil else {
                throw error ?? URLError(.unknown)
            }
            return data
        })
    }
    task.resume()
}
```

Now, let's say we are the consumer of `foo`...

```swift
foo(url: url) { result in
    switch result {
    case .success(let data):
        print(data.count)
    case .failure(let error):
        print(error)
    }
}
```

Again, this approach works, but is a little cumbersome as we have the boilerplate of the switch statement and the unnatural syntax of not catching the error.

```swift
extension Result {
    func unwrap() throws -> Wrapped {
        switch self {
        case .success(let data):
            return data
        case .failure(let error):
            throw error
        }
    }
}
```

So, we have extended `Result` to add a function to unwrap the value or error, which allows us to use much more natural syntax again.

```swift
foo(url: url) { result in
    do {
        let data = try result.unwrap()
        print(data.count)
    }
    catch {
        print(error)
    }
}
```

<table>
<tr>
<td><img src="https://oliverrussellwhite.github.io/hero.png"></td>
<td>
<p>Hello! I'm Oliver. I'm a software engineer, and tutor.</p>
<p>I've been building apps for &#63743; platforms for ten years, and tutoring software developers in Swift and iOS for over a year.</p>
<p><a href="mailto:fortandlangley@gmail.com">I would ♥︎ to work with you.</a></p>
</td>
</tr>
</table>
