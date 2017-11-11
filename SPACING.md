# Spacing Style Guide
When and where to use spaces and empty lines.


 _Note: when this guide says __one__ blank line then __exactly one__ blank line is meant!_

#### Closing Scopes
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

#### Property Observers & Getters/Setters
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

#### Inside Protocol Definitions (and Enums or Structs without implementations)

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


#### Marks
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

#### Inside Methods and other Blocks of Code
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

#### Switch Statements
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

_Exception:_ `switch` statements with a low number of cases and only one line of code pre case may be concentrated.
```swift
switch self {
case .production:
    return "abc"
case .staging:
    return "test.abc"
}
```

#### Overview
```swift
import Framework

// MARK: SomeDelegate

protocol SomeDelegate {
    func one()
    func two()
}

// MARK: - SomeClass

class SomeClass {

    var a: TypeA
    var b: TypeB

    private let c: TypeC
    private let d: TypeD

    private var e: TypeE {
        willSet {
            // code
        }
        didSet {
          switch e {
          case a:
              // one line of code
          case b:
              // one line of code
          }
        }
    }

    // MARK: - Topic

    func method() {
        guard condition else {
            // code
            return
        }

        do {
            // code
        } catch let error {
            // code  
        }

        switch thing {

          case a:
              // multiple lines of code

          case b:
              // multiple lines of code

          default:
              // code
        }
    }
}
```
