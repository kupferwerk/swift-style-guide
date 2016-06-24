A guide to our Swift style and conventions.

This is an attempt to encourage patterns that accomplish the following goals (in
rough priority order):

 1. Increased rigor, and decreased likelihood of programmer error
 1. Increased clarity of intent
 1. Reduced verbosity
 1. Fewer debates about aesthetics

If you have suggestions, please see our [contribution guidelines](CONTRIBUTING.md),
then open a pull request. :zap:

----

#### Whitespace

 * Tabs, not spaces.
 * End files with a newline.
 * Make liberal use of vertical whitespace to divide code into logical chunks.
 * Don’t leave trailing whitespace.
   * Not even leading indentation on blank lines.


#### Prefer `let`-bindings over `var`-bindings wherever possible

Use `let foo = …` over `var foo = …` wherever possible (and when in doubt). Only use `var` if you absolutely have to (i.e. you *know* that the value might change, e.g. when using the `weak` storage modifier).

_Rationale:_ The intent and meaning of both keywords is clear, but *let-by-default* results in safer and clearer code.

A `let`-binding guarantees and *clearly signals to the programmer* that its value is supposed to and will never change. Subsequent code can thus make stronger assumptions about its usage.

It becomes easier to reason about code. Had you used `var` while still making the assumption that the value never changed, you would have to manually check that.

Accordingly, whenever you see a `var` identifier being used, assume that it will change and ask yourself why.

#### Avoid Using Force-Unwrapping of Optionals

If you have an identifier `foo` of type `FooType?` or `FooType!`, don't force-unwrap it to get to the underlying value (`foo!`) if possible.

Instead, prefer this:

```swift
if let foo = foo {
    // Use unwrapped `foo` value in here
} else {
    // If appropriate, handle the case where the optional is nil
}
```

Alternatively, you might want to use Swift's Optional Chaining in some of these cases, such as:

```swift
// Call the function if `foo` is not nil. If `foo` is nil, ignore we ever tried to make the call
foo?.callSomethingIfFooIsNotNil()
```

_Rationale:_ Explicit `if let`-binding of optionals results in safer code. Force unwrapping is more prone to lead to runtime crashes.

#### Avoid Using Implicitly Unwrapped Optionals

Where possible, use `let foo: FooType?` instead of `let foo: FooType!` if `foo` may be nil (Note that in general, `?` can be used instead of `!`).

_Rationale:_ Explicit optionals result in safer code. Implicitly unwrapped optionals have the potential of crashing at runtime.

#### Prefer implicit getters on read-only properties and subscripts

When possible, omit the `get` keyword on read-only computed properties and
read-only subscripts.

So, write these:

```swift
var myGreatProperty: Int {
	return 4
}

subscript(index: Int) -> T {
    return objects[index]
}
```

… not these:

```swift
var myGreatProperty: Int {
	get {
		return 4
	}
}

subscript(index: Int) -> T {
    get {
        return objects[index]
    }
}
```

_Rationale:_ The intent and meaning of the first version is clear, and results in less code.

#### Always specify access control explicitly for top-level definitions

Top-level functions, types, and variables should always have explicit access control specifiers:

```swift
public var whoopsGlobalState: Int
internal struct TheFez {}
private func doTheThings(things: [Thing]) {}
```

However, definitions within those can leave access control implicit, where appropriate:

```swift
internal struct TheFez {
	var owner: Person = Joshaber()
}
```

_Rationale:_ It's rarely appropriate for top-level definitions to be specifically `internal`, and being explicit ensures that careful thought goes into that decision. Within a definition, reusing the same access control specifier is just duplicative, and the default is usually reasonable.

#### When specifying a type, always associate the colon with the identifier

When specifying the type of an identifier, always put the colon immediately
after the identifier, followed by a space and then the type name.

```swift
class SmallBatchSustainableFairtrade: Coffee { ... }

let timeToCoffee: NSTimeInterval = 2

func makeCoffee(type: CoffeeType) -> Coffee { ... }
```

_Rationale:_ The type specifier is saying something about the _identifier_ so
it should be positioned with it.

Also, when specifying the type of a dictionary, always put the colon immediately
after the key type, followed by a space and then the value type.

```swift
let capitals: [Country: City] = [ Sweden: Stockholm ]
```

#### Only explicitly refer to `self` when required

When accessing properties or methods on `self`, leave the reference to `self` implicit by default:

```swift
private class History {
	var events: [Event]

	func rewrite() {
		events = []
	}
}
```

Only include the explicit keyword when required by the language—for example, in a closure, or when parameter names conflict:

```swift
extension History {
	init(events: [Event]) {
		self.events = events
	}

	var whenVictorious: () -> () {
		return {
			self.rewrite()
		}
	}
}
```

_Rationale:_ This makes the capturing semantics of `self` stand out more in closures, and avoids verbosity elsewhere.

#### Prefer structs over classes

Unless you require functionality that can only be provided by a class (like identity or deinitializers), implement a struct instead.

Note that inheritance is (by itself) usually _not_ a good reason to use classes, because polymorphism can be provided by protocols, and implementation reuse can be provided through composition.

For example, this class hierarchy:

```swift
class Vehicle {
    let numberOfWheels: Int

    init(numberOfWheels: Int) {
        self.numberOfWheels = numberOfWheels
    }

    func maximumTotalTirePressure(pressurePerWheel: Float) -> Float {
        return pressurePerWheel * numberOfWheels
    }
}

class Bicycle: Vehicle {
    init() {
        super.init(numberOfWheels: 2)
    }
}

class Car: Vehicle {
    init() {
        super.init(numberOfWheels: 4)
    }
}
```

could be refactored into these definitions:

```swift
protocol Vehicle {
    var numberOfWheels: Int { get }
}

func maximumTotalTirePressure(vehicle: Vehicle, pressurePerWheel: Float) -> Float {
    return pressurePerWheel * vehicle.numberOfWheels
}

struct Bicycle: Vehicle {
    let numberOfWheels = 2
}

struct Car: Vehicle {
    let numberOfWheels = 4
}
```

_Rationale:_ Value types are simpler, easier to reason about, and behave as expected with the `let` keyword.

#### Make classes `final` by default

Classes should start as `final`, and only be changed to allow subclassing if a valid need for inheritance has been identified. Even in that case, as many definitions as possible _within_ the class should be `final` as well, following the same rules.

_Rationale:_ Composition is usually preferable to inheritance, and opting _in_ to inheritance hopefully means that more thought will be put into the decision.


#### Omit type parameters where possible

Methods of parameterized types can omit type parameters on the receiving type when they’re identical to the receiver’s. For example:

```swift
struct Composite<T> {
	…
	func compose(other: Composite<T>) -> Composite<T> {
		return Composite<T>(self, other)
	}
}
```

could be rendered as:

```swift
struct Composite<T> {
	…
	func compose(other: Composite) -> Composite {
		return Composite(self, other)
	}
}
```

_Rationale:_ Omitting redundant type parameters clarifies the intent, and makes it obvious by contrast when the returned type takes different type parameters.

#### Use whitespace around operator definitions

Use whitespace around operators when defining them. Instead of:

```swift
func <|(lhs: Int, rhs: Int) -> Int
func <|<<A>(lhs: A, rhs: A) -> A
```

write:

```swift
func <| (lhs: Int, rhs: Int) -> Int
func <|< <A>(lhs: A, rhs: A) -> A
```

_Rationale:_ Operators consist of punctuation characters, which can make them difficult to read when immediately followed by the punctuation for a type or value parameter list. Adding whitespace separates the two more clearly.

## Naming

#### Include all the words needed to avoid ambiguity for another person reading the code

For example, consider a method that removes the element at a given position within a collection.

```swift
extension List {
  public mutating func remove(at position: Index) -> Element
}

// Usage
items.remove(at: x)
```

If we left out the word ```at``` from the method signature, the reader of the code could think that the method searches for and removes an element equal to ```x``` rather than removing the element at the position ```x```.

```swift
employees.remove(x) // unclear: are we removing x?
```
_Rationale:_ Omitting words that add important information about what the function does creates ambiguity especially when the function name can have multiple meanings.

#### Omit needless words

On the other side, leave out words that doesn't convey salient information at the use site.
In particular, words that merely repeat type information can be omitted.

In the following example the word ```Element``` adds no additional information when using the method.

```swift
public mutating func removeElement(member: Element) -> Element?

// usage
views.removeElement(cancelButton)
```

Instead write:

```swift
public mutating func remove(member: Element) -> Element?

// Usage
views.remove(cancelButton) // clearer
```

Looking at the following example (in the perspective of the use site), which words are needed and which words should be left out?
```swift
mainView.addChild(sideBar, atPoint: origin)
```
The word 'Child' is needed here, because it adds relevant information about the role of the first parameter, it's establishing an hierarchy indicating that the sideBar (of which we know from the type system that it's a UIView) will be a child of the mainView.
<br>At the second argument we know from the type system that it's going to be a Point so 'atPoint' is only restating type information that the compiler will enforce anyway.

After removing redundant words, the call site looks like this:
```swift
mainView.addChild(sideBar, at: origin)
```

_Rationale:_ Including words that don't add important information increases redundancy and the volume of the code that must be read and understood by other programmers.

#### Strive for fluent usage

Name functions in a way that using them reads like grammatical English phrases, for example:

```swift
dataSource.insert(item, at: index)
sharingService.share(post, with: accessToken)
database.fetch(.Users, createdAfter: date)
```

Note: It is acceptable for fluency to degrade after the first or second argument when those arguments are not central to the call’s meaning:

```swift
AudioUnit.instantiate(with: description, options: [.inProcess], completionHandler: stopProgressBar)
```

_Rationale:_ Designing functions to read like grammatical English sentences at the use site drastically reduces the time needed to understand the meaning of the code and eliminates the need for describing comments.

#### Name functions according to their side-effects

Functions without side-effects should read as noun phrases where the <b>noun describes the returned result</b>, like:
```swift
origin.distance(to: target)
element.successor()
controller.managedSubviews()
```
Functions with side-effects should read as imperative verb phrases where the <b>verb describes the side effect</b>, like:
```swift
print(description)
names.sort()
dataSource.append(item)
```
<br>
Name mutating/nonmutating method pairs consistently. For example when an operation is naturally described by a verb, use the verbs imperative for the mutating method and apply the "ed" or "ing" suffix for its nonmutating counterpart:

| Mutating | Nonmutating |
|---|---|
| Mutates the receiver in place | Returns a mutated copy of the receiver, without changing the receiver itself |
| `names.sort()`  | `names.sorted()`|
| `results.reverse()`  | `results.reversed()`|
| `dataSource.append(item)` | `dataSource.appending(item)` |

_Rationale:_ Designing function names that indicate side effects raises the awareness of the user that the function may mutate the receiver or affect the application state in another way that may not be obvious otherwise.

#### Name Booleans as assertions

Using a method returning a `Bool` or a property of type `Bool` should read as an assertion about the receiver, like:
```swift
dataSource.isEmpty
name.contains(searchTerm)
gameObject.hasCollider
```

It is important to avoid negated assertions that may lead to double negatives when used in an if statement, like:
```swift
if !gameObject.hasNoCollider { ... } // pure chaos
```

_Rationale:_ Assertions strongly indicate that the returned value of a function or property is of type Bool for the reader of the code.
Furthermore, following the rule that functions should read as grammatically correct phrases, naming Booleans as assertions creates this effects when appended to a receiver.

#### Name Protocols according to what they describe

Protocols that <b>describe what something is</b> should read as nouns, e.g. `Collection`, `GameStateObserver`

Protocols that <b>describe a capability</b> should be named using the suffixes 'able', 'ible' or 'ing', e.g. `Parsable`, `ProgressReporting`

#### Parameters

Choose parameter names to make documentation easy to read. For example, these names make documentation read naturally:
```swift
// Return an `Array` containing the elements of `self`
// that satisfy `predicate`.
func filter(_ predicate: (Element) -> Bool) -> [Generator.Element]

// Replace the given `subRange` of elements with `newElements`.
mutating func replaceRange(_ subRange: Range, with newElements: [E])
```
_Rationale:_ Even though parameter names do not appear at the point of use, they play an important explanatory role.

##### Use default parameters to simplify common uses

Any parameter with a single commonly-used value is a candidate for a default.

For example in a custom view controller transition that usually has the same animation duration application-wide, default this parameter:
```swift
extension UIViewController {
  func magicallyPresent(viewController: UIViewController,
        withDuration duration: NSTimeInterval = MagicTransition.defaultDuration)  
}

_Rationale:_

// Usages
magicallyPresent(viewController) // using the default (MagicTransition.defaultDuration)
magicallyPresent(viewController, withDuration: 0.4) // setting a custom animation duration
```
_Note: In order to use a member as the default value, it has to be declared as a static property._

<br>
Furthermore replacing larger method families with the use of default parameters simplifies usage and reduces the complexity of the API.

For example:
```swift
extension String {
  // ...description...
  public func compare(other: String, options: CompareOptions = [],
                      range: Range? = nil, locale: Locale? = nil) -> Ordering
}
```

is preferred over:
```swift
extension String {
  // ...description 1...
  public func compare(other: String) -> Ordering
  // ...description 2...
  public func compare(other: String, options: CompareOptions) -> Ordering
  // ...description 3...
  public func compare(other: String, options: CompareOptions, range: Range) -> Ordering
  // ...description 4...
  public func compare(other: String, options: StringCompareOptions,
                      range: Range, locale: Locale) -> Ordering
}
```

_Rationale:_ Default arguments are generally preferable to the use of method families, because they impose a lower cognitive burden to anyone trying to understand the API.
