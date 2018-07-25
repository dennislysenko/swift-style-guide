A guide to our Swift style and conventions.

This is an attempt to encourage patterns that accomplish the following goals (in
rough priority order):

 1. Increased rigor, and decreased likelihood of programmer error
 1. Increased clarity of intent
 1. Reduced verbosity

#### Whitespace

 * Just use the default editor settings in Xcode.
 * End files with a newline.
 * Make concise use of vertical whitespace to divide code into logical chunks. Prefer MARK comments over multiple newlines, to denote sections.
 * Don’t leave trailing whitespace.
   * Not even leading indentation on blank lines.

#### Method Naming Guidelines ####

Refer to https://swift.org/documentation/api-design-guidelines/

#### Prefer `let`-bindings over `var`-bindings wherever possible

Use `let foo = …` over `var foo = …` wherever possible (and when in doubt). Only use `var` if you absolutely have to (i.e. you *know* that the value might change, e.g. when using the `weak` storage modifier).

_Rationale:_ The intent and meaning of both keywords are clear, but *let-by-default* results in safer and clearer code.

A `let`-binding guarantees and *clearly signals to the programmer* that its value will never change. Subsequent code can thus make stronger assumptions about its usage, especially with value types.

It becomes easier to reason about code. Had you used `var` while still making the assumption that the value never changed, you would have to manually check that.

Accordingly, whenever you see a `var` identifier being used, assume that it will change and ask yourself why.

#### Return and break early

When you have to meet certain criteria to continue execution, try to exit early. So, instead of this:

```swift
if n.isNumber {
    // Use n here
} else {
    return
}
```

use this:
```swift
guard n.isNumber else {
    return
}
// Use n here
```

You can also do it with `if` statement, but using `guard` is preferred, because `guard` statement without `return`, `break` or `continue` produces a compile-time error, so exit is guaranteed.

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

_Rationale:_ The intent and meaning of the first version are clear, and results in less code.

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
let capitals: [Country: City] = [sweden: stockholm]
```

#### Explicitly refer to `self`, except when it adversely impacts readability

When accessing properties or methods on `self`, make the reference to `self` explicit by default:

```swift
private class History {
	var events: [Event]

	func rewrite() {
		self.events = []
	}
}
```

Only omit the explicit keyword when it hinders readability:

```swift
private class CustomView {
	let subview = UIView()
	let otherSubview = UIView()
	let thirdSubview = UIView()
	
	init() {
		[subview, otherSubview, thirdSubview].forEach(self.addSubview)
		// looks much better than [self.subview, self.otherSubview, self.thirdSubview].forEach(self.addSubview)
	}
}
```

_Rationale:_ This makes the semantics of instance vars more explicit, while the caveat that you can omit `self` at your discretion when it impacts readability allays verbosity concerns. 

**You can be liberal with the "adversely impacts readability" rule.** However, when writing instance variables, you should generally explicitly specify `self` as that will prevent any potenetial issues with in-scope name shadowing, and generally should not introduce verbosity the way variable reads do.

#### Decide whether to use structs or classes based on whether you want value semantics

Unless you require functionality that can only be provided by a class (like identity or deinitializers), implement a struct instead.

Note that inheritance is (by itself) usually _not_ a good reason to use classes, because polymorphism can be provided by protocols, and implementation reuse can be provided through composition.

However, note that _not_ using inheritance is also _not_ a good reason to use structs, as structs have value semantics, which can be expensive, and lead to unpleasant surprises.

_Rationale:_ If you understand value semantics, and can reason about both class and value types, you are better placed to make a decision than if we just made one arbitrary guideline.

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

#### Do not use extra spaces around method calls

Do not use extra spaces around the parentheses in method calls. Commas should have no space before, and one space after.

**Undesirable**: 

```sin ( a, b )```

```sin( a, b )```

```sin (a, b)```

**Desirable**:

```sin(a, b)```

#### Use lowerCamelCase naming for all methods and variables, but namespace when it makes sense

Prefer to use lowerCamelCase naming for all methods and variables, even global functions, constants, and global variables.

However, when you have several variables that have the same role within a few different concepts, e.g. model, view, and projection matrices for two cameras, feel free to use snake case to break them up and namespace them.

`sceneSpace_modelViewProjectionMatrix` and `screenSpace_modelViewProjectionMatrix` 

read better than:

`sceneSpaceModelViewProjectionMatrix` and `screenSpaceModelViewProjectionMatrix`.

Generally this exception should only be made in graphics and algorithm code. Consider a GPU operation where you are operating on groups of images at two or three defined resolutions (e.g. to save on heavier kernel execution time by making heavier kernels execute on downscaled textures). You may want to define a variable naming scheme that reads like this:

```swift
let upscaled_brightmapTexture = ...
let downscaled_blurTexture = ...
let downscaled_brightmapTexture = ...
let upscaled_brightmapTexture = ...
let upscaled_finalTexture = ...
```

In this scheme, the snake case helps to group the textures and allows the user to determine which textures have similarity across groups, drastically reducing the potential for programmer error when changing code in the future. 

#### Do not abbreviate variable or method names unless they significantly impact readability

Terseness does not confer any performance benefit. Xcode's fuzzy autocomplete is good enough that you can afford to name your variables with fully-readable names. If you are having readability issues because of long variable names, first check that you are not organizing poorly. For example, you may be tempted to write shorter variable names so you avoid lines like:

```
let transformIntoWorldSpace = ...
let finalTransform = SCNMatrix4Mult(transformIntoWorldSpace, SCNMatrix4Rotate(SCNMatrix4Invert(SCNMatrix4Mult(cameraProjection, cameraTransform), Double.pi / 2, 0, 0, 1)))
```

If you rewrote it with shorter variable names, it would be much smaller and could read something like this:

```
let tw = ...
let t = SCNMatrix4Mult(tw, SCNMatrix4Rotate(SCNMatrix4Invert(SCNMatrix4Mult(cp, ct), Double.pi / 2, 0, 0, 1)))
```

But realistically, you should first check if you can fix this by refactoring the code itself:

```
let transformIntoWorldSpace = ...
let composedCameraTransform = SCNMatrix4Mult(cameraProjection, cameraTransform)
let landscapeTransformFromCamera = SCNMatrix4Invert(composedCameraTransform)
let portraitTransformFromCamera = SCNMatrix4Rotate(landscapeTransformFromCamera, Double.pi / 2, 0, 0, 1)
let finalTransform = SCNMatrix4Mult(transformIntoWorldSpace, portraitTransformFromCamera)
```

The final code essentially documents itself and is significantly easier to reason about and refactor if needed.
