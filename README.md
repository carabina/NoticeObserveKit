# NoticeObserveKit

[![Version](https://img.shields.io/cocoapods/v/NoticeObserveKit.svg?style=flat)](http://cocoapods.org/pods/NoticeObserveKit)
[![License](https://img.shields.io/cocoapods/l/NoticeObserveKit.svg?style=flat)](http://cocoapods.org/pods/NoticeObserveKit)
[![Platform](https://img.shields.io/cocoapods/p/NoticeObserveKit.svg?style=flat)](http://cocoapods.org/pods/NoticeObserveKit)

NoticeObserveKit is type-safe NotificationCenter wrapper that associates notice type with info type.

```swift
//UIKeyboardWillShow conforms to NoticeType protocol.
UIKeyboardWillShow.observe { keyboardInfo in
    //In this case, keyboardInfo is UIKeyboardInfo type.
    //It is inferred from UIKeyboardWillShow's InfoType.
    print(keyboardInfo)
}
//pool is NoticeObserverPool.
//If pool is released, NoticeObserves are automatically removed.
.addObserverTo(pool)
```

## Usage

First of all, you need to implement `InfoType` and `name` with `NoticeType` protocol.
`InfoType` means value type of element in `userInfo` of `Notification`.

```swift
struct UIKeyboardWillShow: NoticeType {
    typealias InfoType = UIKeyboardInfo
    static let name: Notification.Name = .UIKeyboardWillShow
}
```

If you define custom object as `InfoType`, you need to implement that with `NoticeUserInfoDecodable` protocol. To confirm this protocol, you must implement `init?(info: [AnyHashable : Any])` and `func dictionaryRepresentation() -> [AnyHashable : Any]`.

```swift
struct UIKeyboardInfo: NoticeUserInfoDecodable {
    let frame: CGRect
    let animationDuration: TimeInterval
    let animationCurve: UIViewAnimationOptions

    init?(info: [AnyHashable : Any]) {
        guard
            let frame = (info[UIKeyboardFrameEndUserInfoKey] as? NSValue)?.cgRectValue,
            let duration = info[UIKeyboardAnimationDurationUserInfoKey] as? TimeInterval,
            let curve = info[UIKeyboardAnimationCurveUserInfoKey] as? UInt
        else {
            return nil
        }
        self.frame = frame
        self.animationDuration = duration
        self.animationCurve = UIViewAnimationOptions(rawValue: curve)
    }

    func dictionaryRepresentation() -> [AnyHashable : Any] {
        return [:]
    }
}
```

## Customization

If you want to receive `Notification`, you can use receiving parameter.

```swift
UIKeyboardWillShow.observe(recieving: { notification in
    print(notification)
}) { keyboardInfo in
    print(keyboardInfo)
}.addObserverTo(pool)
```

If you want to get specific value from `userInfo` of `Notification`, please implement `infoKey`.

```swift
struct UIKeyboardWillShow: NoticeType {
    typealias InfoType = NSValue
    static let infoKey: String = UIKeyboardFrameEndUserInfoKey
    static let name: Notification.Name = .UIKeyboardWillShow
}
```

If you can post custom Notification like this.

```swift
struct NavigationControllerDidShow: NoticeType {
    typealias InfoType = NavigationControllerContent
    static var name = Notification.Name("navigationControllerDidShow")
}

let content = NavigationControllerContent(viewController: viewController, animated: animated)
NavigationControllerWillShow.post(info: content)
```

You can dispose manually like this.

```swift
let observer = UIKeyboardWillShow.observe { keyboardInfo in
    print(keyboardInfo)
}

observer.dispose()
```

## Requirements

- Swift 3.0
- Xcode 8.1 or greater
- iOS 8.0 or greater

## Installation

NoticeObserveKit is available through [CocoaPods](http://cocoapods.org). To install
it, simply add the following line to your Podfile:

```ruby
pod "NoticeObserveKit"
```

## Author

marty-suzuki, s1180183@gmail.com

## License

NoticeObserveKit is available under the MIT license. See the LICENSE file for more info.