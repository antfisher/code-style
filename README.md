## Guiding Tenets

* This guide is in addition to the official [Swift API Design Guidelines](https://swift.org/documentation/api-design-guidelines/). These rules should not contradict that document.
* These rules should not fight Xcode's <kbd>^</kbd> + <kbd>I</kbd> indentation behavior.

## Table of Contents

1. [Xcode Formatting](#xcode-formatting)
1. [Naming](#naming)
1. [Style](#style)
    1. [Functions](#functions)
    1. [Closures](#closures)
    1. [Operators](#operators)
1. [Patterns](#patterns)
1. [File Organization](#file-organization)
1. [Objective-C Interoperability](#objective-c-interoperability)


## Xcode Formatting

*  **Each line should have a maximum column width of 150 characters.**

  <details>

  #### Why?
  Due to larger screen sizes, we have opted to choose a page guide greater than 80

  </details>

*  **Use 4 spaces to indent lines.**

* **Trim trailing whitespace in all lines.**

## Naming

*  **Use PascalCase for type and protocol names, and lowerCamelCase for everything else.**

  <details>

  ```swift
  protocol SpaceThing {
    // ...
  }
  class SpaceFleet: SpaceThing {
    enum Formation {
      // ...
    }
    class Spaceship {
      // ...
    }
    var ships: [Spaceship] = []
    static let worldName: String = "Earth"
    func addShip(_ ship: Spaceship) {
      // ...
    }
  }
  let myFleet = SpaceFleet()
  ```

  </details>

  _Exception: You may prefix a private property with an underscore if it is backing an identically-named property or method with a higher access level_

  <details>

  #### Why?
  There are specific scenarios where a backing a property or method could be easier to read than using a more descriptive name.

  - Type erasure

  ```swift
  public final class AnyRequester<ModelType>: Requester {
    public init<T: Requester>(_ requester: T) where T.ModelType == ModelType {
      _executeRequest = requester.executeRequest
    }
    @discardableResult
    public func executeRequest(
      _ request: URLRequest,
      onSuccess: @escaping (ModelType, Bool) -> Void,
      onFailure: @escaping (Error) -> Void) -> URLSessionCancellable
    {
      return _executeRequest(request, session, parser, onSuccess, onFailure)
    }
    private let _executeRequest: (
      URLRequest,
      @escaping (ModelType, Bool) -> Void,
      @escaping (NSError) -> Void) -> URLSessionCancellable
  }
  ```

  - Backing a less specific type with a more specific type

  ```swift
  final class ExperiencesViewController: UIViewController {
    // We can't name this view since UIViewController has a view: UIView property.
    private lazy var _view = CustomView()
    loadView() {
      self.view = _view
    }
  }
  ```

  </details>

* **Protocols that describe what something is should read as nouns** (e.g. Collection).
* **Protocols that describe a capability should be named using the suffixes able, ible, or ing** (e.g. Equatable, ProgressReporting).
* **The names of other types, properties, variables, and constants should read as nouns.**

<details>

```swift
  // WRONG
  protocol MainCoordinatorType {
  }
  
  class MainCoordinator: MainCoordinatorType {
  }
  
  // RIGHT
  protocol MainCoordinator {
  }
  
  class DefaultMainCoordinator: MainCoordinator {
  }
  ```
 </details>
 
*  **Enums should be named as as description of its single case**

<details>

```swift
  // WRONG
  enum Orientations {
    case north
    case westh
    case south
    case east
  }
  
  // RIGHT
  enum Orientation {
    case north
    case westh
    case south
    case east
  }
  ```
 </details>

*  **Name booleans like `isSpaceship`, `hasSpacesuit`, etc.** This makes it clear that they are booleans and not other types.

*  **Acronyms in names (e.g. `URL`) should be all-caps except when it’s the start of a name that would otherwise be lowerCamelCase, in which case it should be uniformly lower-cased.**

  <details>

  ```swift
  // WRONG
  class UrlValidator {
    func isValidUrl(_ URL: URL) -> Bool {
      // ...
    }
    func isUrlReachable(_ URL: URL) -> Bool {
      // ...
    }
  }
  let URLValidator = UrlValidator().isValidUrl(/* some URL */)
  // RIGHT
  class URLValidator {
    func isValidURL(_ url: URL) -> Bool {
      // ...
    }
    func isURLReachable(_ url: URL) -> Bool {
      // ...
    }
  }
  let urlValidator = URLValidator().isValidURL(/* some URL */)
  ```

  </details>

*  **Names should be written with their most general part first and their most specific part last.** The meaning of "most general" depends on context, but should roughly mean "that which most helps you narrow down your search for the item you're looking for." Most importantly, be consistent with how you order the parts of your name.

  <details>

  ```swift
  // WRONG
  let rightTitleMargin: CGFloat
  let leftTitleMargin: CGFloat
  let bodyRightMargin: CGFloat
  let bodyLeftMargin: CGFloat
  // RIGHT
  let titleMarginRight: CGFloat
  let titleMarginLeft: CGFloat
  let bodyMarginRight: CGFloat
  let bodyMarginLeft: CGFloat
  ```

  </details>

*  **Include a hint about type in a name if it would otherwise be ambiguous.**

  <details>

  ```swift
  // WRONG
  let title: String
  let cancel: UIButton
  // RIGHT
  let titleText: String
  let cancelButton: UIButton
  ```

  </details>

*  **Event-handling functions should be named like past-tense sentences.** The subject can be omitted if it's not needed for clarity.

  <details>

  ```swift
  // WRONG
  class ExperiencesViewController {
    private func handleBookButtonTap() {
      // ...
    }
    private func modelChanged() {
      // ...
    }
  }
  // RIGHT
  class ExperiencesViewController {
    private func didTapBookButton() {
      // ...
    }
    private func modelDidChange() {
      // ...
    }
  }
  ```

  </details>

*  **Avoid Objective-C-style acronym prefixes.** This is no longer needed to avoid naming conflicts in Swift.

  <details>

  ```swift
  // WRONG
  class RQAccount {
    // ...
  }
  // RIGHT
  class Account {
    // ...
  }
  ```

  </details>

*  **Avoid `*Controller` in names of classes that aren't view controllers.**
  <details>

  #### Why?
  Controller is an overloaded suffix that doesn't provide information about the responsibilities of the class.

  </details>

## Style

*  **Don't include types where they can be easily inferred.**

  <details>

  ```swift
  // WRONG
  let host: Host = Host()
  // RIGHT
  let host = Host()
  ```

  ```swift
  enum Direction {
    case left
    case right
  }
  func someDirection() -> Direction {
    // WRONG
    return Direction.left
    // RIGHT
    return .left
  }
  ```

  </details>

*  **Don't use `self` unless it's necessary for disambiguation or required by the language.**

  <details>

  ```swift
  final class Listing {
    init(capacity: Int, allowsPets: Bool) {
      // WRONG
      self.capacity = capacity
      self.isFamilyFriendly = !allowsPets // `self.` not required here
      // RIGHT
      self.capacity = capacity
      isFamilyFriendly = !allowsPets
    }
    private let isFamilyFriendly: Bool
    private var capacity: Int
    private func increaseCapacity(by amount: Int) {
      // WRONG
      self.capacity += amount
      // RIGHT
      capacity += amount
      // WRONG
      self.save()
      // RIGHT
      save()
    }
  }
  ```

  </details>

*  **Name members of tuples for extra clarity.** Rule of thumb: if you've got more than 3 fields, you should probably be using a struct.

  <details>

  ```swift
  // WRONG
  func whatever() -> (Int, Int) {
    return (4, 4)
  }
  let thing = whatever()
  print(thing.0)
  // RIGHT
  func whatever() -> (x: Int, y: Int) {
    return (x: 4, y: 4)
  }
  // THIS IS ALSO OKAY
  func whatever2() -> (x: Int, y: Int) {
    let x = 4
    let y = 4
    return (x, y)
  }
  let coord = whatever()
  coord.x
  coord.y
  ```

  </details>

*  **Use constructors instead of Make() functions for CGRect, CGPoint, NSRange and others.**

  <details>

  ```swift
  // WRONG
  let rect = CGRectMake(10, 10, 10, 10)
  // RIGHT
  let rect = CGRect(x: 0, y: 0, width: 10, height: 10)
  ```

  </details>

*  **Place the colon immediately after an identifier, followed by a space.**

  <details>

  ```swift
  // WRONG
  var something : Double = 0
  // RIGHT
  var something: Double = 0
  ```

  ```swift
  // WRONG
  class MyClass : SuperClass {
    // ...
  }
  // RIGHT
  class MyClass: SuperClass {
    // ...
  }
  ```

  ```swift
  // WRONG
  var dict = [KeyType:ValueType]()
  var dict = [KeyType : ValueType]()
  // RIGHT
  var dict = [KeyType: ValueType]()
  ```

  </details>

*  **Place a space on either side of a return arrow for readability.**

  <details>

  ```swift
  // WRONG
  func doSomething()->String {
    // ...
  }
  // RIGHT
  func doSomething() -> String {
    // ...
  }
  ```

  ```swift
  // WRONG
  func doSomething(completion: ()->Void) {
    // ...
  }
  // RIGHT
  func doSomething(completion: () -> Void) {
    // ...
  }
  ```

  </details>

*  **Omit unnecessary parentheses.**

  <details>

  ```swift
  // WRONG
  if (userCount > 0) { ... }
  switch (someValue) { ... }
  let evens = userCounts.filter { (number) in number % 2 == 0 }
  let squares = userCounts.map() { $0 * $0 }
  // RIGHT
  if userCount > 0 { ... }
  switch someValue { ... }
  let evens = userCounts.filter { number in number % 2 == 0 }
  let squares = userCounts.map { $0 * $0 }
  ```

  </details>

*  **Omit enum associated values from case statements when all arguments are unlabeled.**

  <details>

  ```swift
  // WRONG
  if case .done(_) = result { ... }
  switch animal {
  case .dog(_, _, _):
    ...
  }
  // RIGHT
  if case .done = result { ... }
  switch animal {
  case .dog:
    ...
  }
  ```

  </details>

### Functions

*  **Omit `Void` return types from function definitions.**

  <details>

  ```swift
  // WRONG
  func doSomething() -> Void {
    ...
  }
  // RIGHT
  func doSomething() {
    ...
  }
  ```

  </details>

### Closures

*  **Favor `Void` return types over `()` in closure declarations.**

  <details>

  ```swift
  // WRONG
  func method(completion: () -> ()) {
    ...
  }
  // RIGHT
  func method(completion: () -> Void) {
    ...
  }
  ```

  </details>

*  **Name unused closure parameters as underscores (`_`).**

    <details>

    #### Why?
    Naming unused closure parameters as underscores reduces the cognitive overhead required to read
    closures by making it obvious which parameters are used and which are unused.

    ```swift
    // WRONG
    someAsyncThing() { argument1, argument2, argument3 in
      print(argument3)
    }
    // RIGHT
    someAsyncThing() { _, _, argument3 in
      print(argument3)
    }
    ```

    </details>

*  **Single-line closures should have a space inside each brace.**

  <details>

  ```swift
  // WRONG
  let evenSquares = numbers.filter {$0 % 2 == 0}.map {  $0 * $0  }
  // RIGHT
  let evenSquares = numbers.filter { $0 % 2 == 0 }.map { $0 * $0 }
  ```

  </details>

### Operators

*  **Infix operators should have a single space on either side.** Prefer parenthesis to visually group statements with many operators rather than varying widths of whitespace. This rule does not apply to range operators (e.g. `1...3`) and postfix or prefix operators (e.g. `guest?` or `-1`).

  <details>

  ```swift
  // WRONG
  let capacity = 1+2
  let capacity = currentCapacity   ?? 0
  let mask = (UIAccessibilityTraitButton|UIAccessibilityTraitSelected)
  let capacity=newCapacity
  let latitude = region.center.latitude - region.span.latitudeDelta/2.0
  // RIGHT
  let capacity = 1 + 2
  let capacity = currentCapacity ?? 0
  let mask = (UIAccessibilityTraitButton | UIAccessibilityTraitSelected)
  let capacity = newCapacity
  let latitude = region.center.latitude - (region.span.latitudeDelta / 2.0)
  ```

  </details>

## Patterns

*  **Avoid performing any meaningful or time-intensive work in `init()`.** Avoid doing things like opening database connections, making network requests, reading large amounts of data from disk, etc. Create something like a `start()` method if these things need to be done before an object is ready for use.

*  **Extract complex property observers into methods.** This reduces nestedness, separates side-effects from property declarations, and makes the usage of implicitly-passed parameters like `oldValue` explicit.

  <details>

  ```swift
  // WRONG
  class TextField {
    var text: String? {
      didSet {
        guard oldValue != text else {
          return
        }
        // Do a bunch of text-related side-effects.
      }
    }
  }
  // RIGHT
  class TextField {
    var text: String? {
      didSet { updateText(from: oldValue) }
    }
    private func updateText(from oldValue: String?) {
      guard oldValue != text else {
        return
      }
      // Do a bunch of text-related side-effects.
    }
  }
  ```

  </details>

*  **Extract complex callback blocks into methods**. This limits the complexity introduced by weak-self in blocks and reduces nestedness. If you need to reference self in the method call, make use of `guard` to unwrap self for the duration of the callback.

  <details>

  ```swift
  //WRONG
  class MyClass {
    func request(completion: () -> Void) {
      API.request() { [weak self] response in
        if let strongSelf = self {
          // Processing and side effects
        }
        completion()
      }
    }
  }
  // RIGHT
  class MyClass {
    func request(completion: () -> Void) {
      API.request() { [weak self] response in
        guard let strongSelf = self else { return }
        strongSelf.doSomething(strongSelf.property)
        completion()
      }
    }
    func doSomething(nonOptionalParameter: SomeClass) {
      // Processing and side effects
    }
  }
  ```

  </details>

*  **Prefer using `guard` at the beginning of a scope.**

  <details>

  #### Why?
  It's easier to reason about a block of code when all `guard` statements are grouped together at the top rather than intermixed with business logic.

  </details>

*  **Access control should be at the strictest level possible.** Prefer `public` to `open` and `private` to `fileprivate` unless you need that behavior.

*  **Avoid global functions whenever possible.** Prefer methods within type definitions.

  <details>

  ```swift
  // WRONG
  func age(of person, bornAt timeInterval) -> Int {
    // ...
  }
  func jump(person: Person) {
    // ...
  }
  // RIGHT
  class Person {
    var bornAt: TimeInterval
    var age: Int {
      // ...
    }
    func jump() {
      // ...
    }
  }
  ```

  </details>

*  **Prefer putting constants in the top level of a file if they are `private`.** If they are `public` or `internal`, define them as static properties, for namespacing purposes.

  <details>

  ```swift
  private let privateValue = "secret"
  public class MyClass {
    public static let publicValue = "something"
    func doSomething() {
      print(privateValue)
      print(MyClass.publicValue)
    }
  }
  ```

  </details>

*  **Use caseless `enum`s for organizing `public` or `internal` constants and functions into namespaces.** Avoid creating non-namespaced global constants and functions. Feel free to nest namespaces where it adds clarity.

  <details>

  #### Why?
  Caseless `enum`s work well as namespaces because they cannot be instantiated, which matches their intent.

  ```swift
  enum Environment {
    enum Earth {
      static let gravity = 9.8
    }
    enum Moon {
      static let gravity = 1.6
    }
  }
  ```

  </details>

*  **Use Swift's automatic enum values unless they map to an external source.** Add a comment explaining why explicit values are defined.

  <details>

  #### Why?
  To minimize user error, improve readability, and write code faster, rely on Swift's automatic enum values. If the value maps to an external source (e.g. it's coming from a network request) or is persisted across binaries, however, define the values explicity, and document what these values are mapping to.

  This ensures that if someone adds a new value in the middle, they won't accidentally break things.

  ```swift
  // WRONG
  enum ErrorType: String {
    case error = "error"
    case warning = "warning"
  }
  enum UserType: String {
    case owner
    case manager
    case member
  }
  enum Planet: Int {
    case mercury = 0
    case venus = 1
    case earth = 2
    case mars = 3
    case jupiter = 4
    case saturn = 5
    case uranus = 6
    case neptune = 7
  }
  enum ErrorCode: Int {
    case notEnoughMemory
    case invalidResource
    case timeOut
  }
  // RIGHT
  enum ErrorType: String {
    case error
    case warning
  }
  /// These are written to a logging service. Explicit values ensure they're consistent across binaries.
  // swiftlint:disable redundant_string_enum_value
  enum UserType: String {
    case owner = "owner"
    case manager = "manager"
    case member = "member"
  }
  // swiftlint:enable redundant_string_enum_value
  enum Planet: Int {
    case mercury
    case venus
    case earth
    case mars
    case jupiter
    case saturn
    case uranus
    case neptune
  }
  /// These values come from the server, so we set them here explicitly to match those values.
  enum ErrorCode: Int {
    case notEnoughMemory = 0
    case invalidResource = 1
    case timeOut = 2
  }
  ```

  </details>
  
*  **Use nested enum/struct/class for domain object**

  <details>
  
  ```swift
  // WRONG
  enum UserType: String {
    case guest
    case admin
  }
  struct User {
    var type:  UserType
  }
  // RIGHT
  struct User {
    enum `Type`: String {
      case guest
      case admin
    }
    var type: Type
  }
  ```
  </details>
  
*  **Use optionals only when they have semantic meaning.**

*  **Prefer immutable values whenever possible.** Use `map` and `compactMap` instead of appending to a new collection. Use `filter` instead of removing elements from a mutable collection.

  <details>

  #### Why?
  Mutable variables increase complexity, so try to keep them in as narrow a scope as possible.

  ```swift
  // WRONG
  var results = [SomeType]()
  for element in input {
    let result = transform(element)
    results.append(result)
  }
  // RIGHT
  let results = input.map { transform($0) }
  ```

  ```swift
  // WRONG
  var results = [SomeType]()
  for element in input {
    if let result = transformThatReturnsAnOptional(element) {
      results.append(result)
    }
  }
  // RIGHT
  let results = input.compactMap { transformThatReturnsAnOptional($0) }
  ```

  </details>

*  **Handle an unexpected but recoverable condition with an `assert` method combined with the appropriate logging in production. If the unexpected condition is not recoverable, prefer a `precondition` method or `fatalError()`.** This strikes a balance between crashing and providing insight into unexpected conditions in the wild. Only prefer `fatalError` over a `precondition` method when the failure message is dynamic, since a `precondition` method won't report the message in the crash report.

  <details>

  ```swift
  func didSubmitText(_ text: String) {
    // It's unclear how this was called with an empty string; our custom text field shouldn't allow this.
    // This assert is useful for debugging but it's OK if we simply ignore this scenario in production.
    guard text.isEmpty else {
      assertionFailure("Unexpected empty string")
      return
    }
    // ...
  }
  func transformedItem(atIndex index: Int, from items: [Item]) -> Item {
    precondition(index >= 0 && index < items.count)
    // It's impossible to continue executing if the precondition has failed.
    // ...
  }
  func makeImage(name: String) -> UIImage {
    guard let image = UIImage(named: name, in: nil, compatibleWith: nil) else {
      fatalError("Image named \(name) couldn't be loaded.")
      // We want the error message so we know the name of the missing image.
    }
    return image
  }
  ```

  </details>

*  **Default type methods to `static`.**

  <details>

  #### Why?
  If a method needs to be overridden, the author should opt into that functionality by using the `class` keyword instead.

  ```swift
  // WRONG
  class Fruit {
    class func eatFruits(_ fruits: [Fruit]) { ... }
  }
  // RIGHT
  class Fruit {
    static func eatFruits(_ fruits: [Fruit]) { ... }
  }
  ```

  </details>

*  **Default classes to `final`.**

  <details>

  #### Why?
  If a class needs to be overridden, the author should opt into that functionality by omitting the `final` keyword.

  ```swift
  // WRONG
  class SettingsRepository {
    // ...
  }
  // RIGHT
  final class SettingsRepository {
    // ...
  }
  ```

  </details>

*  **Never use the `default` case when `switch`ing over an enum.**

  <details>

  #### Why?
  Enumerating every case requires developers and reviewers have to consider the correctness of every switch statement when new cases are added.

  ```swift
  // WRONG
  switch anEnum {
  case .a:
    // Do something
  default:
    // Do something else.
  }
  // RIGHT
  switch anEnum {
  case .a:
    // Do something
  case .b, .c:
    // Do something else.
  }
  ```

  </details>

*  **Check for nil rather than using optional binding if you don't need to use the value.**

  <details>

  #### Why?
  Checking for nil makes it immediately clear what the intent of the statement is. Optional binding is less explicit.

  ```swift
  var thing: Thing?
  // WRONG
  if let _ = thing {
    doThing()
  }
  // RIGHT
  if thing != nil {
    doThing()
  }
  ```

  </details>


## File Organization

*  **Alphabetize module imports at the top of the file a single line below the last line of the header comments. Do not add additional line breaks between import statements.**

  <details>

  #### Why?
  A standard organization method helps engineers more quickly determine which modules a file depends on.

  ```swift
  // WRONG
  //  Copyright © 2019 Requestum. All rights reserved.
  //
  import DLSPrimitives
  import Constellation
  import Epoxy
  import Foundation
  //RIGHT
  //  Copyright © 2019 Requestum. All rights reserved.
  //
  import Constellation
  import DLSPrimitives
  import Epoxy
  import Foundation
  ```

  </details>

  _Exception: `@testable import` should be grouped after the regular import and separated by an empty line._

  <details>

  ```swift
  // WRONG
  //  Copyright © 2019 Requestum. All rights reserved.
  //
  import DLSPrimitives
  @testable import Epoxy
  import Foundation
  import Nimble
  import Quick
  //RIGHT
  //  Copyright © 2019 Requestum. All rights reserved.
  //
  import DLSPrimitives
  import Foundation
  import Nimble
  import Quick
  @testable import Epoxy
  ```

  </details>

*  **Limit empty vertical whitespace to one line.** Favor the following formatting guidelines over whitespace of varying heights to divide files into logical groupings.

*  **Files should end in a newline.**


## Objective-C Interoperability

*  **Prefer pure Swift classes over subclasses of NSObject.** If your code needs to be used by some Objective-C code, wrap it to expose the desired functionality. Use `@objc` on individual methods and variables as necessary rather than exposing all API on a class to Objective-C via `@objcMembers`.

  <details>

  ```swift
  class PriceBreakdownViewController {
    private let acceptButton = UIButton()
    private func setUpAcceptButton() {
      acceptButton.addTarget(
        self,
        action: #selector(didTapAcceptButton),
        forControlEvents: .TouchUpInside)
    }
    @objc
    private func didTapAcceptButton() {
      // ...
    }
  }
  ```

  </details>
