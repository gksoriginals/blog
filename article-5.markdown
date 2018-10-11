# Initialization in Swift

In Swift, all values must be initialized before they are accessed.

```swift
let age: Int
print(age)  // compiler error!
```

```swift
let age: Int = 32
print(age) // 👍
```

A value does not, however, have to be initialized immediately upon being declared.

```swift
let message: String
if age < 18 {
    message = "No beer for you."
}
else {
    message = "beer for you?"
}
print(message)
```

If you declare an optional variable (`var`), the compiler will initialize the value to `nil` for your ~~pleasure~~ convenience. The compiler will not do the same for a constant (`let`).

```swift
var name: String?
if let name = name {
    print("Hello, \(name)!")
}
else {
    print("Who are you?")
}
```

```swift
let name: String?
if let name = name { // compiler error!
    print("Hello, \(name)!")
}
else {
    print("Who are you?")
}
```

## Failable Initializers

An initialization function ("initializer"), can be declared to be optional (`init?`). You must `return nil` if the initialization process fails.

```swift
struct TMNT {
    let name: String
    init?(name: String) {
        switch name {
        case "Leonardo", "Donatello", "Michelangelo", "Raphael":
            self.name = name
        default:
            return nil
        }
    }
}
```

## Enumerations

An enumeration (`enum`) does not require an explicit initializer, but enumerations can be extended to have custom initializers.

An enumeration initializer must set `self` to one of the enumeration cases.

```swift
enum Human {
    case child
    case teenager
    case adult
}
extension Human {
    init(age: Int) {
        if age < 13 {
            self = .child
        }
        if age < 18 {
            self = .teenager
        }
        self = .adult
    }
}
```

## Structures

A structure (`struct`) is a group of values, so each value must be initialized before the structure can be accessed.

```swift
struct Person {
    let name: String
    let age: Int
}
```

If you do not provide an initializer in the initial declaration of the structure, the Swift compiler will synthesize a memberwise initializer.

```swift
init(name: String, age: Int) {
    self.name = name
    self.age = age
}
```

A structure initializer can either set all of the properties, call another initializer, or set `self` to an instance of the type.

```swift
init(firstName: String, lastName: String, age: Int) {
    self.name = "\(firstName) \(lastName)"
    self.age = age
}
```

```swift
init(firstName: String, lastName: String, age: Int) {
    self.init(name: "\(firstName) \(lastName)", age: age)
}
```

```swift
init(firstName: String, lastName: String, age: Int) {
    self = Person(name: "\(firstName) \(lastName)", age: age)
}
```

## Classes

A class is similar to a structure, but there are rules for class initialization which do not exist for structures.

A class must provide a designated initializer; an initializer will never be synthesized for a class.

A designated initializer is one declared as part of the initial class declaration and initializes all of the classes properties.

```swift
class Person {
    let name: String
    init(name: String) {
        self.name = name
    }
}
```

An initializer in an extension is a `convenience` initializer and must call a designated initializer.

```swift
extension Person {
    convenience init(firstName: String, lastName: String) {
        self.init(name: "\(firstName) \(lastName)")
    }
}
```

A subclass must first initialize it's own properties, then call a super initializer.

```swift
class Person {
    let name: String
    init(name: String) {
        self.name = name
    }
}
class Employee: Person {
    let job: String
    init(name: String, job: String) {
        self.job = job
        super.init(name: name)
    }
}
class Executive: Employee {
    let department: String
    init(name: String, job: String, department: String) {
        self.department = department
        super.init(name: name, job: job)
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