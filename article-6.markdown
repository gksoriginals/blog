# Tuples in Swift

> [In mathematics, a tuple is a finite ordered list (sequence) of elements.](https://en.wikipedia.org/wiki/Tuple)

In Swift, a tuple is also a finite ordered list of elements.

```swift
let person = ("Oliver", 32)
```

A tuple can contain one or more values, of any type. It's syntax is a comma seperator list of elements, enclosed in parentheses.

```swift
person.0 // "Oliver"
person.1 // 32
```

The values in a tuple can be accessed with numeral indices accessed as properties.

```swift
let person = (name: "Oliver", age: 32)
person.name // "Oliver"
person.age // 32
```

A tuple can have labels for its elements, which means its values can be accessed with named properties.

```swift
let (name, age) = person
```

A tuple can be deconstructed into it's component parts through assignment, where the left hand side, is a tuple-like structure of identifiers.

```swift
let (_, age) = person
```

A value can be ignored when deconstructing a tuple, by using an underscore (`_`).

```swift
let person: (String, Int)?
```

A tuple, like any type in Swift, can be made `Optional`, and can therefore be conditionally- or force- unwrapped. 

```swift
("Hello, World!", (1, 2))
```

A tuple can contain, as one of its values, a tuple, which can contain, as one of its values, a tuple, which...

```swift
func parse(data: Data) -> (results: [String], end: Bool)
```

Tuples are designed for returning multiple values from a function. Arguably, a `struct` would be better a solution in most cases, but in cases where that might be considered overkill, a tuple is a quick, informal way of returning muliple values.

```swift
public typealias Void = ()
```

Fun fact, `Void` is actually an empty tuple.

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