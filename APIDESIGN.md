# API Design
<br>

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

_Rationale:_ Assertions strongly indicate that the returned value of a function or property is of type `Bool` for the reader of the code.
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
