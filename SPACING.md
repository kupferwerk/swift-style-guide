# Spacing Style Guide
When and where to use spaces and empty lines.


 _Note: when this guide says __one__ blank line then __exactly one__ blank line is meant!_

### Closing Scopes
Simply __never__ include any whitespace before closing a scope and always __one__ blank line afterwards.
```swift
        return x
    }
}

func nextFunc() {}
```

### Classes & Extensions
Always use exactly __one__ empty line above and below classes and between every one of its methods.

```swift
//
//  ViewController.swift
//  Swift Styleguide
//
//  Created by Developer on 10.11.17.
//  Copyright © 2017 intive_Kupferwerk. All rights reserved.
//

class ViewController {

    // properties

    func lastMethodInThisScope() {
        // code
    }
}

```

### Properties

- cluster variables in groups (static, class, public, internal, private, fileprivate, computed, observed, etc.) and separate them by __one__ blank line
- always surround properties with closures (e.g. observers or lazy) or documentation with __one__ blank line

```swift
struct SomeResource: Resource {

    static let path = "/resource"

    var internalPropertyOne: String
    var internalPropertyTwo: String

    var coolDictionary: [String: String] {
        return ["key": "value"]
    }

    lazy var something: Type = {
        return Type(dict: coolDictionary)
    }()

    private var someNumber: Int
    private var isSomething: Bool
}
```

### Property Observers & Getters/Setters
- use __no__ blank lines at all

```swift
var name: String {
    willSet {
        loadViewIfNeeded()
    }
    didSet {
        nameLabel?.text = name
    }
}
```

### Overridden Functions
- include __one__ blank line _after_ calling the super method if you add _multiple_ lines to your method
- include __no__ blank line _after_ calling the super method if you only add _one_ line

```swift
override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)

    if let parent = parent {
      // code
    }    
}

override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)
    trackPageView()
}
```

### Inside Protocol Definitions (and Enums or Structs without implementations)

- __one__ blank line between the protocol definition and its first property/method, as well as between every property/method signature and (the documentation of) the following one
- __one__ space between a property's type and every used item in `{ get set }`, `{ get }` or `{ set }`

```swift
protocol SomeDelegate: class {

    /// Documentation
    var property: Type { get set }

    /// Documentation
    func methodOne()

    /// Documentation
    func methodTwo()
}

struct Address {

    /// Documentation
    var street: String

    /// Documentation
    var city: String
}
```

- if no documentation is required, use __no__ blank lines at all

```swift
enum Side {
    case left
    case right
}
```

__Exception:__ Structs or enums declared _inside_ classes may occupy too much vertical space so that the blank lines could be omitted (however one might argue that such nested types should be declared in a separate extension anyway)

```swift
class BasketViewController: UIViewController {

    fileprivate enum State {
        /// An error occured while loading the basket.
        case error
        /// A loading operation that is related to the whole view is in progress.
        case loading
        /// The basket does not contain any products.
        case emptyBasket
        /// The basket contains products and should be displayed.
        case filledBasket
        /// There is currently no logged in user.
        case notAuthenticated
    }
}
```


### Marks
Marks should always be surrounded by __exactly one__ blank line above and below.

```swift
      ...
      return something
    }
}

// MARK: - SomeDelegate

extension ViewController: SomeDelegate {

    func delegateMethod() {}
}
```

### Inside Methods and other Blocks of Code
- __do not__ start with a blank line (exceptions may exist where adding __one__ blank line at the beginning might increase readability)
- __one__ blank line after a potential `guard` statement or between multiple ones
- __one__ blank line above and below an `if` statement (except it's the first statement in this method)
- if not eligible to be extracted in a single method, try to __group statements__ that share a topic by not including any whitespace between them and surround the groups with exactly __one__ blank line

```swift
func doStuff(with item: Item) {
    guard operationIsAllowed(with: item) else {
        // handle the case
        return
    }

    if itsTrue {
        // cool
    } else {
        // also fine
    }

    methodWithTopicA()
    anotherMethodWithTopicA()

    methodWithTopicB()
    anotherMethodWithTopicB()
}
```

### Switch Statements
- include __one__ blank line before every case

```swift
var state: State {
    didSet {
        switch state {

        case .loading:
            display(view)
            startActivityIndicator()

        case .notAuthenticated:
            display(loginView)
            track(.unauthenticatedEmptyBasket)

        case .error:
            display(view)
            showDefaultMessage(with: "Error Message", type: .error)

        case .emptyBasket:
            display(recommendationsView)
            track(.authenticatedEmptyBasket)

        case .filledBasket:
            display(productsView)
        }
    }
}
```

__Exception:__ `switch` statements with a low number of cases and only one line of code per case may be concentrated.
```swift
switch self {
case .production:
    return "abc"
case .staging:
    return "test.abc"
}
```

### Overview
```swift
import Framework

// MARK: SomeDelegate

protocol SomeDelegate {

    /// Documentation
    func one()

    /// Documentation
    func two()
}

// MARK: - SomeClass

class SomeClass {

    var a: TypeA
    var b: TypeB

    private let c: TypeC
    private let d: TypeD

    private var state: State {
        willSet {
            // code
        }
        didSet {
          switch state {
          case a:
              // one line of code
          case b:
              // one line of code
          }
        }
    }

    // MARK: - Topic

    override func method(with item: Item) {
        super.method()

        guard condition else {
            // code
            return
        }

        do {
            // code
        } catch let error {
            // code  
        }

        // code
        // code
        // code

        switch thing {

          case a:
              // multiple lines
              // of code

          case b:
              // multiple lines
              // of code

          default:
              // code
        }
    }
}

// MARK: Delegate

extension SomeClass: Delegate {

    // code
}
```
