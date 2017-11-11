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
//  Created by Developer on 02.10.17.
//  Copyright © 2017 intive_Kupferwerk. All rights reserved.
//

class ViewController {

    // ...

    func lastMethodInThisScope() {
        // method stuff
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

#### An Example Class
```swift
//
//  BasketViewController.swift
//  Esprit
//
//  Created by Marco Oliva on 25.07.17.
//  Copyright © 2017 intive_Kupferwerk. All rights reserved.
//

import Foundation
import UIKit
import XCGLogger

class BasketViewController: EspritViewController, AppFeatures {

    fileprivate struct EditingDataKeys {
        static let productData = "productData"
        static let colorSelection = "colorSelection"
    }

    fileprivate enum State {

        /// An error occured while loading the basket.
        case error

        /// A loading operation that is related to the whole view is in progress (like loading the basket or authenticating a user).
        case loading

        /// The basket does not contain any products.
        case emptyBasket

        /// The basket contains products and should be displayed.
        case filledBasket

        /// There is currently no logged in user.
        case notAuthenticated
    }

    fileprivate lazy var productsView: BasketProductsView = {
        let productsView: BasketProductsView = .fromNib()
        productsView.delegate = self
        return productsView
    }()

    /// The current state of the `BasketViewController`.
    /// When mutated, the view automatically updates to display the new state.
    fileprivate var state: State = .loading {
        didSet {
            updateUserInterface(for: state)
        }
    }

    // MARK: - View Lifecycle

    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        navigationController?.setNavigationBarHidden(false, animated: false)

        if LoginService.shared.state != .loggedOut {
           loadBasket()
        } else {
           state = .notAuthenticated
        }
    }

    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)

        trackPageView()
    }
}

// MARK: - Basket Updates

extension BasketViewController {

    /// Fetches the basket from the backend.
    fileprivate func loadBasket() {
        state = .loading

        BasketService.shared.getBasket { [weak self] (_, basket, invalidToken) in
            guard !invalidToken else {
                self?.basketProcessFailedDueToInvalidToken()
                return
            }

            self?.updateBasket(to: basket)
        }
    }

    /// Updates the display of basket items and performs operations depending on a new basket state.
    /// Note: This method does _not refresh_ or _reload_ the basket from the backend,
    /// but rather should be called to update the local state.
    ///
    /// - Parameter basket: The updated state of the basket.
    private func updateBasket(to basket: NativeBasket?) {
        if let basket = basket {
            state = basket.items.isEmpty ? .emptyBasket : .filledBasket
        } else {
            state = .error
        }

        productsView.basket = basket
    }
}
```
