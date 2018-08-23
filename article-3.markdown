# Decoding JSON in Swift with JSONDecoder and Decodable

`JSONDecoder` is a class in Apple's Foundation framework to decode JSON data into Swift types. It requires the types conform to a protocol called `Decodable`.

If the structure of the JSON and the structure of the Swift type are aligned, and if the JSON keys and Swift property names and data types are equal, then no additional code is required other than explicitly declaring conformance to `Decodable`.

```json
{
    "name": "Oliver",
    "job": "Software Engineer & Tutor"
}
```

```swift
struct Person: Decodable {
    let name: String
    let job: String
}
```

```swift
let person = try JSONDecoder().decode(Person.self, from: data)
// => person.name == "Oliver"
// => person.job == "Software Engineer & Tutor"
```

In this example, the JSON contains a single object with two keys, `name` and `job`, which are both strings. The Swift type, `Person`, is a struct (object) with two properties, `name` and `job`, which are both strings.

Here, both the structures match, as do the property names and types.

Decoding the `Person` from the data, then, is as simple as asking `JSONDecoder` to decode the type from the data.

The code to decode the type is generated at compile time by the Swift compiler.

##

If the JSON keys and Swift property names don't match, you have to provide the compiler with the information it needs to map the keys to the property names.

```swift
struct Person: Decodable {

    private enum CodingKeys: String, CodingKey {
        case name
        case job = "occupation"
    }

    let name: String
    let job: String
}
```

We must declare a `private` `enum` called `CodingKeys`, which is backed by a `String` and conforms to `CodingKey`. We can then enumerate our Swift property names and associated JSON keys.

Here `name` doesn't require an explicit `rawValue` because it is the same as the `case` name. And we have defined the JSON key "occupation" to be used for the Swift property name `job`.

##

If the structure of the JSON and the structure of the Swift type don't match, you won't be able to rely on the compiler to generate the decoding code for you, and you will have to write it yourself.

```json
{
    "data": {
        "title": "Hello, World!"
    }
}
```

```swift
struct Post {
    let title: String
}
```

```swift
extension Post: Decodable {

    private enum CodingKeys: String, CodingKey {
        case data
    }

    private enum DataCodingKeys: String, CodingKey {
        case title
    }

    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        let dataContainer = try container.nestedContainer(keyedBy: DataCodingKeys.self, forKey: .data)
        self.title = try dataContainer.decode(String.self, forKey: .title)
    }
}
```

The first thing to note is that we are now declaring the conformance to `Decodable` in an extension. This is not required.

Conformance to `Decodable` must be declared when the type is being defined (`struct Person: Decodable`), for the compiler to generate the decoding code.

However, as the structures don't match and we have to write the code ourselves anyway, we are free to declare conformance in an extension.

Conformance to `Decodable` required implementing one method:

```swift
init(from decoder: Decoder) throws
```

First, we need to ask the decoder to find a (keyed) container, which is the way a `Decoder` refers to a `Dictionary`, i.e. a container / object with keys and values. And the keys are defined in our `CodingKeys` enumeration.

```swift
let container = try decoder.container(keyedBy: CodingKeys.self)
```

We are free to name our enumerations whatever we like here, but it seems prudent to follow the existing pattern.

```swift
private enum CodingKeys: String, CodingKey {
    case data
}
```

The `CodingKeys` enumeration has a single key, `data`, as the JSON object we are decoding has only one key that we are interested in.

```swift
let dataContainer = try container.nestedContainer(keyedBy: DataCodingKeys.self, forKey: .data)
```

The JSON `data` key contains a single object, so we must ask the container we just decoded (which represents the top level element in the JSON structure), to decode a nested container associated with the key `.data` from our `CodingKeys` enumeration. This nested container should contain the keys in our `DataCodingKeys` enumeration.

```swift
self.title = try dataContainer.decode(String.self, forKey: .title)
```

Now that we have the `dataContainer`, we can finally decode the `String` value for the key `.title` from our DataCodingKeys enumeration.

##

It may seem like a great deal of work, and far more preferable to have the compiler write the code for you, but your Swift code shouldn't be forced to conform to the structure of your JSON code, just to avoid writing decoding code.

Consider the following example of the JSON data structure of the Reddit API.

```json
{
    "data": {
        "children": [
            {
                "data": {
                    "title": "Hello, World!"
                }
            }
        ]
    }
}
```

It would require the following ridiculous Swift type:

```swift
struct Listing {
    struct Data {
        struct Child {
            struct Data {
                let title: String
            }
            let data: Data
        }
        let children: [Child]
    }
    let data: Data
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