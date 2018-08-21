# Safeguarding Against Stringly-typed APIs in Swift

A Stringly-typed API is an API (function) which accepts a `String` as a parameter, which is used as an identifier / key.

```swift
UserDefaults.standard.bool(forKey: "enableSecretFeatures")
```

```swift
try AVAudioRecorder(url: url, settings: [
    AVFormatIDKey : kAudioFormatMPEG4AAC,
    AVSampleRateKey : 12000,
    AVNumberOfChannelsKey: 1,
])
```

`UserDefaults` uses the `String` key as an opaque identifier. In other words, `UserDefaults` doesn't care about the actual contents of the string, the string is simply used to store and retrieve the associated value.

For example, at a coffee shop, when they ask for your name, you don't *have* to say "Emma", you could say "Custard". You will still receive your coffee when they shout "Custard". The name you give is an opaque identifier to the barista — it only means something to you.

`AVAudioRecorder` uses `String` keys for the `settings` `Dictionary` parameter . The keys are **not** opaque.  `AVAudioRecorder` only understands specific values.

For example, back at the coffee shop, now you've identified yourself as "Custard", you can only order items that the coffee shop sells, using the names (keys) that the coffee shop defines. 

## The Problem

In the first example, if you use an incorrect `String` — whether it's a different `String` ("highestScore" vs. "highScore"), or a `String` with a typo ("hightestScore") — you will not receive the value you're expecting. However, no error has occured. The API has been used correctly and it's expected that a key may not be found. You now have a silent bug.

In the second example, `AVAudioRecorder` will only search the dictionary for keys that it is expecting. Therefore, if you use an incorrect `String` as a key, the key / value pair will be ignored, and the desired setting will not be applied. But again, no error has occured. You have another silent bug.

## The Solution

So, we want a solution that avoids using strings, to safeguard against all the problems we have identified, so the compiler can validate our API usage.

```swift
extension UserDefaults {

    enum Key: String {
        case enableSecretFeature
    }

    func object(forKey key: Key) -> Any? {
        return self.object(forKey: key.rawValue)
    }
}

UserDefaults.standard.object(forKey: .enableSecretFeature)
```

Here, we have defined an `enum` called `Key`, which is backed by a `String` (which means every `case` has a `rawValue` property which is a `String` which is the same as the `case` name).

```swift
UserDefaults.Key.enableSecretFeature.rawValue == "enableSecretFeature"
```

Additionally, we have defined a `func`tion that accepts a `Key` value and uses an existing API to return the value, using the `enum`'s `rawValue`.

We now get the benefit of the compiler's type-checking and auto-complete to show the possible `Key` `case`s. 

## The Conclusion

The solution presented is not the only solution. For example,

```swift
extension UserDefaults {
    var enableSecretFeatures: Bool {
        get {
            return self.bool(forKey: "enableSecretFeatures")
        }
        set {
            self.set(newValue, forKey: "enableSecretFeatures")
        }
    }
}
```

*Note: a previous version of this article suggested using `#function`, but as pointed out by [thisischemistry](https://www.reddit.com/user/thisischemistry), this is dangerous as the function name may be refactored in the future changing the value of `#function`. Lesson learned: don't add code examples when editing a final draft.*

However, regardless of the solution, the goal should be to avoid creating stringly-typed APIs yourself, and to abstract  stringly-typed APIs that you can't avoid using.

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
