# Data Sources and Delegates for Views in Swift

MVC is an architectural pattern for building software, usually with a user interface. There are three components: Model, Controller and View.

- A model encapsulates data and logic.
- A view renders a user interface, usually with data from the models, and accepts user input.
- A controller responds to the user input, which is delegated from the view, and interacts with the models.

*This is intended to be a simplified overview of MVC. I know the definition of MVC is triggering for some people. But please — please — don’t email me.*

So, a view has two responsibilities:

1. To render a user interface, which may require a data source.
2. To accept user input, which may require a delegate.

In Objective-C, data sources and delegates are usually defined with protocols, where an object can conform to the protocol. Everything in Objective-C is either an object or a protocol (except C scalar types).

In Swift, we have a much broader palette of tools. So, I want to explore the options for implementing data sources and delegates in Swift.

We will focus solely on data sources because the approaches will apply to delegates just the same.

## Data Sources

![PreviewView example](article4-image-1.png)

```swift
final class PreviewView: UIView {

    private let imageView: UIImageView
    private let textLabel: UILabel

    override init(frame: CGRect) {
        self.imageView = UIImageView(frame: .zero)
        self.imageView.translatesAutoresizingMaskIntoConstraints = false
        self.imageView.contentMode = .scaleAspectFill

        self.textLabel = UILabel(frame: .zero)
        self.textLabel.translatesAutoresizingMaskIntoConstraints = false
        self.textLabel.numberOfLines = 0

        super.init(frame: frame)

        self.layoutMargins = UIEdgeInsetsMake(15.0, 15.0, 15.0, 15.0)

        self.addSubview(self.imageView)
        self.addSubview(self.textLabel)

        NSLayoutConstraint.activate([
            self.imageView.topAnchor.constraint(equalTo: self.layoutMarginsGuide.topAnchor),
            self.imageView.leadingAnchor.constraint(equalTo: self.layoutMarginsGuide.leadingAnchor),
            self.imageView.heightAnchor.constraint(equalToConstant: 60.0),
            self.imageView.widthAnchor.constraint(equalTo: self.imageView.heightAnchor),

            self.textLabel.topAnchor.constraint(equalTo: self.layoutMarginsGuide.topAnchor),
            self.textLabel.leadingAnchor.constraint(equalTo: self.imageView.trailingAnchor, constant: 15.0),
            self.textLabel.trailingAnchor.constraint(equalTo: self.layoutMarginsGuide.trailingAnchor),
            self.textLabel.heightAnchor.constraint(greaterThanOrEqualTo: self.imageView.heightAnchor),

            self.layoutMarginsGuide.bottomAnchor.constraint(equalTo: self.textLabel.bottomAnchor),
            ])
    }

    required init?(coder _: NSCoder) {
        fatalError()
    }
}
```

### Option 1

The first option is to remove the `private` attribute from our subviews and let the whole world configure our subviews as they please.

This, however, is a terrible idea.

The fact that we’re using a text label and an image view to render the interface is an implementation detail. It may change in the future. We may elect to draw the user interface using CoreGraphics. So, the whole world would not longer be able to configure the text label and the image view, as they would not exist.

**The means by which a view is supplied with the data it requires to render itself, should by independent of the internal implementation.**

In other words, use should be able to refactor the internal implementation of a view, without breaking any other part of your application.

### Option 2: Protocols

```swift
protocol PreviewViewDataSource {
    func attributedText(for previewView: PreviewView) -> NSAttributedString?
    func image(for previewView: PreviewView) -> UIImage?
}

final class PreviewView: UIView {

    var dataSource: PreviewViewDataSource? {
        didSet {
            self.textLabel.attributedText = self.dataSource?.attributedText(for: self)
            self.imageView.image = self.dataSource?.image(for: self)
        }
    }
}
```

We could define a protocol, `PreviewViewDataSource `, with two methods to request the data required to render the user interface. And add a `dataSource` property to our view to configure subviews, when called.

We would also be required to implement the protocol somewhere, most likely in a controller.

```swift
extension ViewController: PreviewViewDataSource {

    func attributedText(for previewView: PreviewView) -> NSAttributedString? {
        let attributedString = NSMutableAttributedString()
        attributedString.append(
            NSAttributedString(string: "Probably Science", attributes: [
                .font: UIFont.preferredFont(forTextStyle: .headline),
                .foregroundColor: UIColor.black,
                ])
        )
        attributedString.append(
            NSAttributedString(string: "\nAndy Wood, Matt Kirshen", attributes: [
                .font: UIFont.preferredFont(forTextStyle: .subheadline),
                .foregroundColor: UIColor.lightGray,
                ])
        )
        return attributedString
    }

    func image(for previewView: PreviewView) -> UIImage? {
        return UIImage(named: "art.jpeg")
    }
}
```

This approach is not ideal as the data transformation code (the implementation of the protocol) does not belong in the controller.

*And, it’s triggering the MVVM people. Please don’t email me.*

### Option 3: Structs

```swift
struct PreviewViewDataSource {
    let attributedText: NSAttributedString?
    let image: UIImage?
}

final class PreviewView: UIView 
{
    var dataSource: PreviewViewDataSource? {
        didSet {
            self.textLabel.attributedText = self.dataSource?.attributedText
            self.imageView.image = self.dataSource?.image
        }
    }
}
```

We could define a `struct`, `PreviewViewDataSource `, which contains properties for the data required by the view.

This approach is better, and allows us to solve the problem of where the transformation code should live, because we can extend our data source and add custom initialisers for our models.

```swift
struct Podcast {
    let title: String
    let author: String
    let artwork: UIImage
}

extension PreviewViewDataSource {
    init(podcast: Podcast) {
        let attributedText = NSMutableAttributedString()
        attributedText.append(
            NSAttributedString(string: podcast.title, attributes: [
                .font: UIFont.preferredFont(forTextStyle: .headline),
                .foregroundColor: UIColor.black,
                ])
        )
        attributedText.append(
            NSAttributedString(string: "\n" + podcast.author, attributes: [
                .font: UIFont.preferredFont(forTextStyle: .subheadline),
                .foregroundColor: UIColor.lightGray,
                ])
        )
        self.attributeText = attributedText
        self.image = podcast.artwork
    }
}
```

So, our controller can simply create an instance of our data source, using a model.

```swift
previewView.dataSource = PreviewViewDataSource(podcast: podcast)
```

This is my preferred approach because it achieves all my architectural goals,

1. The view’s implementation is private.
2. The view is responsible for configuring itself, as only it can do with a private implementation.
3. The view is explicitly defining the data it requires.
4. We have isolated the transformation code, into a testable component (the data source custom initialiser).
5. The controller is now doing only the work it should be doing (coordinating the flow of data between the model layer and the view layer).
6. There is no tight coupling between any of the components.

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