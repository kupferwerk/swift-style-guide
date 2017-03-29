# Code Style
<br>

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

If you have an identifier `foo` of type `Optional<FooType>`, don't force-unwrap it to get to the underlying value (`foo!`) if possible.

Instead, prefer one of the two following approaches:

- Use if-let when the most of your code is executed *outside* your if-clause

```swift
if let foo = foo {
    // Use unwrapped `foo` value in here
} else {
    // If appropriate, handle the case where the optional is nil
}
```
- Use guard-let when most of your code would be executed *inside* your if-clause

```swift
guard let foo = foo else {
	// handle ase where optional is nil
}

// use unwrapped `foo` value here
```

Generally perfer using guard-let to if-let. Only use if-let if you have to.

Alternatively, you might want to use Swift's Optional Chaining in some of these cases, such as:

```swift
// Call the function if `foo` is not nil. If `foo` is nil, ignore we ever tried to make the call
foo?.callSomethingIfFooIsNotNil()
```

or Swift's default values for optionals such as:

```swift
// use value of foo if it is not nil use value after ?? if foo is nil
let x = foo ?? bar
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

#### Choosing between structs and classes

As a general guideline, consider creating a structure when one or more of these conditions apply:
* The structure’s primary purpose is to encapsulate a few relatively simple data values.
* It is reasonable to expect that the encapsulated values will be copied rather than referenced when you assign or pass around an instance of that structure.
* Any properties stored by the structure are themselves value types, which would also be expected to be copied rather than referenced.
* The structure does not need to inherit properties or behavior from another existing type.

Examples of good candidates for structures include:
* The size of a geometric shape, perhaps encapsulating a width property and a height property, both of type Double.
* A way to refer to ranges within a series, perhaps encapsulating a start property and a length property, both of type Int.
* A point in a 3D coordinate system, perhaps encapsulating x, y and z properties, each of type Double.

In all other cases, define a class, and create instances of that class to be managed and passed by reference. In practice, this means that most custom data constructs should be classes, not structures.

Note that inheritance is (by itself) usually _not_ a good reason to use classes, because polymorphism can be provided by protocols, and implementation reuse can be provided through composition.

For more details look at: https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Properties.html

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
    
    func maximumTotalTirePressure(vehicle: Vehicle, pressurePerWheel: Float) -> Float
}

extension Vehicle {
    func maximumTotalTirePressure(vehicle: Vehicle, pressurePerWheel: Float) -> Float {
        return pressurePerWheel * vehicle.numberOfWheels
    }
}

struct Bicycle: Vehicle {
    let numberOfWheels = 2
}

struct Car: Vehicle {
    let numberOfWheels = 4
}
```

_Rationale:_ This may seem simliar to base classes or even abstract classes but there are a few key differences:
* Value types can be simpler, easier to reason about, and behave as expected with the `let` keyword.
* Types can conform to more than one protocol, they can be decorated with default behaviors from multiple protocols. 
* Protocols can be adopted by classes, structs and enums. Base classes and inheritance are restricted to class types.

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

#### Don't write Swift as Objective-C with new syntax

Make use of features in swift to reduce boilerplate code.

For example, to select elements of an array that pass a certain condition, instead of:

```swift
var filteredArray: [Int] = []

for money in moneyArray {
    if (money > 30) {
      filteredArray += [money]
    }
}
```

use the `filter` method:

```swift
let filteredArray = moneyArray.filter({$0 > 30})
```
where {$0 > 30} is the supplied filter closure.

Note that parameters are omitted using the default $0 parameter name and the return type is implicitly assumed to be Bool.


To transform elements of an array, for example to convert an array of Integers to an array of Strings and append a suffix on each element, instead of:

```swift
var stringsArray: [String] = [] // Note that we have to specify the type of the array or else we'll get an type error

for money in moneyArray {
    stringsArray += "\(money)€"
}
```
use the `map` method:

```swift
let stringsArray = moneyArray.map({"\($0)€"})
```

_Rationale:_ Less code leads to less bugs and unnessecary boilerplate code increases the time needed to read and understand the code.

#### Multiple Optional Binding

Multiple Optional Binding is possible since Swift 1.2 and is recommended to avoid the "pyramid of doom".

Instead of:

```swift
if let widget = widget {
  if let url = options.url {
    if let host = options.host {
      // widget, url and host are all non-optionals
    }
  }
}
```

use:

```swift
if let widget = widget, url = options.url, host = options.host {
  // widget, url and host are all non-optionals
}
```

_Rationale:_ Multiple nested statements make it harder to follow along the initial intention and unnessecarily blow up methods.
