## TOC

## Foreword

> The information in this document applies to version 3.0.0 of the compiler.


> `@oigroup/lightscript` adheres to semver regarding language syntax and
> semantics -- the same major compiler version should compile the same source
> to the same output, barring bug fixes and optimizations.


> Come hang out with us on Gitter: https://gitter.im/lightscript/Lobby


> Feel free to open an issue at https://github.com/wcjohnson/lightscript/issues
> if you encounter a problem with LightScript.

## Variables & Assignment

### `const`

LightScript is `const` by default; the keyword is not necessary:

    greeting = 'Hello, World!'

This is also true when destructuring:

    { color, owner } = lightsaber

Because LightScript is a (rough) superset of JavaScript,
the `const` keyword is also valid:

    const greeting = 'Hello, World!'

LightScript uses Facebook's [Flow](https://flow.org/) type syntax,
so you can optionally annotate types:

    greeting: string = 'Hello, World!'

As a rule of thumb, anywhere you can use Flow syntax in JavaScript,
you can use the same syntax in LightScript.

Note that, unlike in JavaScript, the `:` cannot be followed by a newline.

> Integration with the Flow typechecker has **not been built yet**,
> so while you can annotate your types, they will not yet be statically checked.
> This is blocking on [Flow accepting an AST as input](https://github.com/facebook/flow/issues/1515).

### `let` and `var`

`let` and `var` are the same as in JavaScript.

    let friendCount = 1

However, to reassign a variable, you must use the keyword `now`:

    let friendCount = 1
    // makes a friend...
    now friendCount = 2

### Updating values

Reassigning or updating a variable requires the `now` keyword.
This makes it more clear when you are reassigning
(which shouldn't be often), and enables the `const`-by-default syntax.

    let isDarkSide = false
    // ... gets stuck in traffic ...
    now isDarkSide = true

Assignments that update a variable also require the `now` keyword:

    let planetsDestroyed = 0
    // the death star is fully operational
    now planetsDestroyed += 1

However, assigning to an object's property allows, but does not require `now`:

    now milleniumFalcon.kesselRun = 'less than 12 parsecs!'

    milleniumFalcon.kesselRun = 'less than 12 parsecs!'

Similarly, non-assignment updates allow but do not currently require `now`:

    now planetsDestroyed++
    planetsDestroyed++

### Shadowing Variables

You cannot shadow a variable using `const` shorthand:

    let myVar = 2
    if (true) {
      myVar = 3
    }

The above is not allowed because it looks ambiguous and confusing;
did you mean to declare a new variable, or were you trying to update
the existing variable, and just forgot to use `now`?

Instead, you must explictly use either `now` or `const`:

    let myVar = 2
    if (true) {
      const myVar = 3
    }
    myVar === 2 // true

    let myOtherVar = 4
    if (true) {
      now myOtherVar = 5
    }
    myOtherVar === 5 // true

## Whitespace

Since (almost) all valid JavaScript is valid LightScript,
you can write loops and conditionals as you always would, complete with parens and curlies.

You can write `if`, `for`, etc without parens, but with curly braces:

    for const x of arr {
      if x > 10 {
        print('wow, so big!')
      } else {
        print('meh')
      }
    }

Or, if you prefer, use significant indentation:

    for const x of arr:
      if x > 10:
        print('wow, so big!')
      else:
        print('meh')

This "curly-or-whitespace" option is also available for class and function bodies.

Whitespace (significant indentation) is often more concise and readable for simple code,
but when blocks get very long or deeply nested, curly braces offer better visibility. Teams have the freedom to choose a style that feels comfortable.

### One-line syntax

Blocks that contain only a single statement can be written on the same line:

    for elem thing in list: print(thing)

    if x < 10: print('x is small')

One-line blocks can be combined:

    for elem x in arr: if x < 10: print(x)

You cannot, however, mix one-line and multiline syntax:

    for x of arr: print(x)
      print('This is broken.')

### What constitutes indentation?

An indent is two spaces, period.

LightScript only allows "normal" (ascii-32) whitespace;
tabs, non-breaking spaces, and other invisible characters raise a SyntaxError.
Similarly, only `\n` and `\r\n` are valid line terminators.

Overindentation is currently allowed, but discouraged. It may be made illegal in the future.

### When does an indent end?

An indented block is parsed until the indent level of a line is less than or equal to
the indent level of the line that started the block (the line with a `:` or `->`).

For example, this does not work:

    if treeIsPretty and
      treeIsTall:
      climbTree()

but this does (because the line with the `:` has one indent):

    if treeIsPretty and
      treeIsTall:
        climbTree()

However, that's a little ugly; the recommended style is:

    if (
      treeIsPretty and
      treeisTall
    ):
      climbTree()

### Mixing and matching

While LightScript allows both "curly blocks" and significantly-indented blocks,
they cannot be mixed for the same construct:

    if true:
      okay()
    else {
      thisIsNotAllowed()
    }

Furthermore, the indentation level must be the same across all branches of
`if`/`else`/`elif`, `try`/`catch`/`finally` and `do`/`while`.

### Objects and Blocks

In JavaScript, curly braces do double-duty, delimiting both objects and blocks of code. When you write code with curly braces, LightScript does its best to figure out what you mean.

The general rule is that if something enclosed with `{ }` looks like an object, it will be treated as such, otherwise it will be treated as a block of code.

For more specifics, check out [the gritty details](#something) -- though we hope you won't ever need to refer to them! It should "just work" the way you'd expect.

## Conditionals

### `elif`

The same as `else if`, which you can also use:

    if awesome:
      jumpForJoy()
    elif great:
      highFive()
    else if good:
      smileMeekly()
    else:
      cringe()

### `if` expressions

In LightScript, ternaries look like `if`s:

    animal = if canBark: 'dog' else: 'cow'

### Multiline `if` expressions
    animal =
      if canBark:
        'dog'
      elif canMeow:
        print('These ternaries can take multiple expressions')
        'cat'
      else:
        'cow'

Note that if you move the `if` to the first line, the rest of the code
must be dedented so that the `else`s have the same indent level as `animal`.

### `undefined`-default `if` expressions

If you don't include an `else`, it will be `undefined`:

    maybeDog = if canBark: 'dog'


## Logic and Equality

### `==`
    1 == 1

Both `==` and `===` compile to `===`, which is almost always what you want.

When you actually want to use a loose-equals (`==`), call the `looseEq()` function
from the [standard library](#standard-library):

    1~looseEq('1')

### `!=`
    1 != 0

Similarly, both `!=` and `!==` compile to `!==`.

When you actually want to use a loose-not-equals (`!=`),
call the `looseNotEq()` function from the [standard library](#standard-library):

    1~looseNotEq('2')

### `or`
    a or b

### `and`
    a and b

### `not`
    not c


## Functions and Methods

JavaScript has half a dozen ways to define a function;
LightScript unifies that to just one, consistent across contexts.

The basic syntax comes from stripping down the fat arrow:

    const myFunction = (x, y) => x + y

to the more minimal:

    myFunction(x, y) => x + y

Unbound functions use a skinny arrow (`->`),
async functions use a barbed arrow (`-/>` or `=/>`),
and methods look the exact same as top-level functions.

LightScript functions have implicit returns and optional curly braces:

    myFunction(x, y) =>
      print('multiplying is fun!')
      x * y

    myCurlyFunction(x, y) => {
      print('adding is fun!')
      x + y
    }

### Bound

    foo() => this.someProp

Compiles to ES6 fat arrows whenever possible,
and inserts the relevant `.bind()` call otherwise.
See also [bound methods](#bound-methods).

Note that when used in an expression, the name is discarded:

    runCallback(foo() => 1)

### Unbound

Skinny arrows (`->`) are unbound:

    ultimateQuestion() -> 6 * 9

    sillySumPlusTwo(a, b) ->
      now a++
      now b++
      a + b

While you're welcome to use `=>` pretty much everywhere, there are a few advantages
of using skinny arrows:

- `function` declarations are hoisted, meaning you can declare utility methods
  at the bottom of a file, and main methods at the top.
- Fat-arrow methods insert `.bind()` calls, which may be unncessary if the method
  doesn't actually need to be bound.

### Without implicit returns

To disable implicit returns for a method, give it a `void` type annotation.

    foo(): void ->
      1

LightScript does not add implicit returns:

- To functions with a `void` returnType annotation (eg; `fn(): void ->`).
- To setter methods (eg; `{ prop(newValue) -set> this._prop = newValue }`).
- To constructor methods (eg; `constructor() ->`), which generaly should not return.

### Empty functions

An arrow function without a body is illegal:

    ->

The standard way of writing an empty function in LightScript is:

    -> return

### Type Annotations

LightScript uses Facebook's [Flow](https://flow.org/) type syntax.

    foo(a: string, b: number): number ->
      a.length + b

Polymorphic:

    foo<T>(a: T): T -> a

### Anonymous

    runCallback(() -> 42)
    runCallback(param -> param * 2)
    runCallback(param => param * 2)

### Async

    foo() -/>
      Promise.resolve(42)

    boundFoo() =/>
      Promise.resolve(this.answer + 42)

See also [await](#await).

### Generators

    foo() -*>
      yield 3
      yield 4

Note that JavaScript does not support fat-arrow generator functions;
LightScript compiles them to bound functions:

    boundFoo() =*>
      yield 3
      yield 4

### Async Generators

If you are using the
[`async-generator-functions` babel transform](https://babeljs.io/docs/plugins/transform-async-generator-functions/),
you can define async generators with `-*/>` and `=*/>`.

>Note that this is (at time of writing) a stage 3 proposal
>and thus not yet part of `babel-preset-env` or any browers.

### Object Methods
    obj = {
      foo() -> 'hello'
      bar() ->
        'hi there'
    }

### Bound Methods
    obj = {
      name: 'Jack'
      loudName() => this.name.toUpperCase()
    }

### Getters and Setters
    obj = {
      get foo() -> this._foo
      set foo(newValue) -> this._foo = newValue
    }
    obj.foo = 'hi'
    obj.foo

See also [Classes](#classes).


## Await

```
getData(url) -/>
  response <- fetch(url)
  <- response.json()
```

### Await and Assign

To assign to a `const`, supply a variable name on the left side of the arrow:
```
getData(url) -/>
  response <- fetch(url)
  response
```
To reassign an existing variable, use `now`:
```
reassignData(data) -/>
  now data <- asyncTransform(data)
  data
```
If you are mutating an object's property, `now` is optional:
```
reassignDataProp(obj) -/>
  now obj.data <- process(obj.data)
  obj.data <- process(obj.data)
  obj
```
Note that in all cases, the `<-` must be on the same line as the variable.

### Await without Assign

A `<-` that begins a line is a "naked await":
```
delayed(action, delay) -/>
  <- waitFor(delay)
  action()
```
It can be implicitly returned like anything else:
```
getData(url) =/>
  response <- fetch(url)
  <- response.json()
```

### Await Array

When an `await` is followed by a `[`, it is wrapped in `Promise.all()`:
```
fetchBoth(firstUrl, secondUrl) -/>
  <- [fetch(firstUrl), fetch(secondUrl)]
```
You do not need to use the `<-` symbol to take advantage of this:
```
fetchBoth(firstUrl, secondUrl) -/>
  return await [fetch(firstUrl), fetch(secondUrl)]
```
You cannot pass a value that happens to be an array; it must be contained in `[]`:
```
awaitAll(promises) -/>
  <- promises
```
doesn't work, but this does:
```
awaitAll(promises) -/>
  <- [...promises]
```

### Safe Await

The most likely source of errors in any application should occur at I/O boundaries,
which are also typically crossed asynchronously. Any time you `fetch()` across a network,
you should expect it to fail some percentage of the time, and prepare accordingly.

In JavaScript, this can be inconvenient:

    getData(url) -/>
      let response
      try {
        now response = await fetch(url)
      } catch (err) {
        handle(err)
        return
      }
      return await response.json()

The small try/catch blocks are annoying and force you to use unnecessary `let`s.
The alternative is to put all logic using the `await`ed value into the `try` block,
which is also an anti-pattern.

In LightScript, you can easily wrap an await in a try/catch:
```
getData(url) -/>
  response <!- fetch(url)

  if isError(response):
    handle(response)
    return

  <- response.json()
```

This pattern is much closer to the "safely handle errors within normal control-flow"
philosophy of Rust, Haskell, and Go, which use `Result`, `Maybe`, or multiple return values.

Because LightScript uses Flow for static type checking, any code that follows a
`<!-` must handle the error case.


## Property Access

### Null-Safe Property Access

*(also known as safe navigation operator, optional chaining operator, safe call operator, null-conditional operator)*

    lacesOrNull = hikingBoots?.laces

This also works with computed properties:

    treeTrunk?.rings?[age]

This also works with function calls, terminating the chain if the callee is not a function:

    if someAnimal.getFins?(): "fish"

Calls and other complex expressions within safe chains are only evaluated once:

    getDancingQueen()?.feelTheBeat(tambourine)

If a safe chain expression terminates early, it will evaluate to `undefined`:

    (null)?.prop

> When using safe chaining, spaces are forbidden near the `?`:
> ```
> someAnimal.getFins? ()
> ```

### Numerical Index Access

    firstChance = chances.0
    secondChance = chances.1

This is a minor feature to make chaining more convenient, and may be removed in the future.

There is not a negative index feature (eg; `chances.-1` doesn't work),
but with the standard library you can write:

    lastChance = chances~last()

This uses the [Tilde Call](#tilde-calls) feature and the `lodash.last()` function,
which LightScript makes available in its [standard library](#standard-library).

### Property function definition

You can define properties that are functions using the standard LightScript arrow syntax:

    mammaMia.resistYou() -> false

    mammaMia.hereWeGo() => this.again

    Mamma.prototype.name() -> "Mia!"

## Tilde Calls

This is a headline feature of LightScript, and a slightly unique mix of
Kotlin's Extensions Methods, Ruby's Monkey Patching, and Elixir's Pipelines.

    subject~verb(object)

The underlying goal is to encourage the functional style of separating
immutable typed records from the functions that go with them,
while preserving the human-readability of "subject.verb(object)" syntax
that Object-Oriented methods provide.

It can enable more readable code:

    if response~isError():
      freakOut()

<!-- -->

    money = querySelectorAll('.money')
    money~map(mustBeFunny)

And makes chaining with functions much more convenient, obviating intermediate variables:

    allTheDucks
      .map(duck => fluffed(duck))
      ~uniq()
      ~sortBy(duck => duck.height)
      .filter(duck => duck.isGosling)

`~` can be thought of as roughly analogous to the proposed JavaScript bind operator `::`, except that `~` avoids the use of `this`, `Function.call`, and the JavaScript OO system in favor of purely functional code.

As part of a potential future JavaScript standard proposal, `~>` is supported as an alternative syntax for tilde calls, since `~` is a reserved JavaScript operator.

    subject~>verb(object)

## Bang Calls

Functions can be called without parenthesis using a `!` suffix:

    getSomething!
      .then! (something) ->
        print! something
      .catch! (err) ->
        print! "There was a problem!"

This syntax combines with [Tilde Calls](#):

    subject~verb! object

...and safe chaining:

    fish~hasPart?! "gills"

Arguments can be on one line, separated by commas:

    f! arg1, arg2, arg3

...or split between multiple lines, with optional commas.

    f!
      arg1
      arg2
      arg3

Since they omit so much punctuation, bang calls are particularly sensitive to
whitespace. If a bang call is split across multiple lines, all arguments must
occur at the same indent level as the first:

    f!
    arg1
    arg2
        arg3 // nope!

Subscripts (dots, brackets, and chained calls) that are aligned with
the arguments of the bang call will adhere to the bang call. Deeper subscripts
will adhere to the arguments.

    allAboard!
      theExpress!
        .train
      to
      .crazyTown!

Arguments must be separated from the `!` by whitespace. Anything not separated by whitespace will be interpreted as a subscript:

    callWithArray! [1]
    callThenIndexResult![1]


## Objects

See also [Methods](#methods).

### Single-Line Objects

The same as JavaScript, including ES7 syntax:

    obj = { a: 'a', b, [1 + 1]: 'two', ...spr }

### Multi-Line Objects

Commas are optional; newlines are preferred.

    obj = {
      a: 'a'
      b
      method() =>
        3
      [1 + 1]: 'two'
    }

## Arrays

### Single-Line Arrays

The same as JavaScript.

    arr = [1, 2, 3]

### Multi-Line Arrays

Again, commas are optional; newlines are preferred.

    arr = [
      1
      2
      2 + 1
      5 - 1
      5
    ]

### Safe Spread and Elision

When using ES2015 spread, a safety check is added to make sure you aren't spreading `undefined`, which would throw a `TypeError` at runtime. This also allows the use of `if` with spread to elide elements while assembling an array:

    arr = [
      1
      ...if shouldAddTwo: [2]
      3
    ]


## Loops

In JavaScript, there's really only one fast option for iteration: `for-;;`.
It's so ugly, though, that most developers avoid it in favor of more ergonomic
(but less performant and powerful) alternatives, like `.forEach()` and `for-of`.

With LightScript, you don't have to compromise.

### Iterating over Arrays

Iterate over indices and elements of an array:

    for idx i, elem x in arr:
      print(i, x)

Only indices:

    for idx i in arr:
      print(i)

Only elements:

    for elem x in arr:
      print(x)

Note that if you are iterating over something more complicated than a variable
(eg; a function call), it will be lifted into its own variable so as not to be called twice:

    for elem x in foo():
      print(x)

### Iterating over Objects

Iterate over keys and values of an object:

    for key k, val v in obj:
      print(k, v)

Only keys:

    for key k in obj:
      print(k)

Only values:

    for val v in obj:
      print(v)

> Note the use of `Object.keys()` under the hood, as this will only iterate over
> _own_ keys, not inherited ones. Use a [traditional `for-in`](#traditional-for-in)
> if you wish to iterate over inherited properties as well.

### Destructuring elements or values

You can destructure the elements of an Array or the values of an Object, similar to ES2015 JavaScript:

Array element destructuring:

    for elem { color } in [{ color: 'blue' }]:
      print(color)

Object value destructuring:

    for val [first, second] in { bases: ['who', 'what'] }:
      print(first, second)

The full power of [destructuring syntax](https://mdn.io/destructuring_assignment) is possible here:

    for elem { props: { color, size: [w, h] }  } in arr:
      print(color, w, h)

### Traditional `for-in`

If you wish to iterate over all owned _and inherited_ keys of an object,
use `for-in` with `const`, `let`, `var`, or `now`:

    for const k in obj:
      print(k)
<!-- -->

    var k;
    for now k in {a: 1, b: 2}:
      print()

    print(k)
    // "b"

Unfortunately, the more concise `for x in arr` form would be ambiguous
(is it keys of an object or values of an array?) and is not allowed:

    for x in arr:
      print(x)

### Traditional `for-of`

A plain JavaScript `for-of` loop can be used to iterate over anything exposing `[Symbol.iterator]` (eg; a generator function):

    for x of gen:
      print(gen)

> Note that `for..of` is [slower](https://jsperf.com/for-of-vs-for-loop/15)
> than `for (idx/elem/key/val) .. in` when iterating over plain JavaScript arrays
> and objects.

### Single-line `for`

    for elem x in stuff: print(x)

This syntax can be used with all `for` loops.

Note that you can combine this with single-line `if` statements:

    for elem x in stuff: if x > 3: print(x)


### Iterating over Numerical Ranges

With the large iteration toolbox available, numeric iteration is infrequently used, so there is no special syntax for ranges in LightScript. When numeric iteration is needed, the traditional `for` loop is up to the task:

    for let i = 0; i < n; i++:
      print(i)

In situations where the overhead of allocating a nonce object is acceptable,
use of `Array()` or the lodash `range()` method (provided by [the stdlib](#standard-library)) can make for cleaner-looking code:

    for idx i in Array(10):
      print(i)
<!-- -->

    for idx i in range(20, 100, 2):
      print(i)


### `while` loops

As in JavaScript, with the option to use LightScript syntax:

    while true:
      doStuff()

### `do-while`

As in JavaScript, with the option to use LightScript syntax:

    do:
      activities()
    while true

A newline or semicolon must follow the trailing `while` clause, so this is not legal in LightScript:

    do:
      activities()
    while (true) foo()

### `switch`

As in JavaScript. Curly braces around the `case`s are required;
parens around the discriminant are not:

    switch val {
      case "x":
        break
      case "y":
        break
    }

> `switch` may be changed in the future to support whiteblock syntax

## Spread Loops

Spread `for` is a feature in the vein of Comprehensions in other languages and prior versions of LightScript. They allow you to assemble an array or object by iterating over and transforming another data structure.

### Basic Syntax

In situations where an ES2015+ spread element would be permitted, the syntax `...for` may be used to introduce a spread loop:

    copyArray(src) ->
      [ ...for elem e in src: [ e ] ]

The spread loop can be thought of as introducing a single spread element for
each iteration of the loop. Whatever expression comes at the end of the loop
will be spread into the array, once for each iteration of the loop.

### Arrays

When spreading into an array in JavaScript, a layer of brackets is removed:

    oneOne = [ ...[ 1 ] ]
    // oneOne deepEquals [1]

The same is true with spread loops; the expression at the end of the spread loop
will be unwrapped and pushed to the array. In a typical case, this just means
wrapping the element you want to push with `[ ]`:

    input = [1, 2, 3]
    output = [ ...for elem e in input: [ e + 1 ] ]
    // this is equivalent to [...[1], ...[2], ...[3]]
    // so output deepEquals [1, 2, 3]

    // an implementation of Array.prototype.map:
    myMap(arr, f) ->
      [ ...for elem e in arr: [ f(e) ] ]

This also means that you can map a single element of the input array to
multiple elements of the output array:

    input = [1, 2, 3]
    output = [ ...for elem e in input: [e, e+1] ]
    // this is equivalent to [ ...[1, 2], ...[2, 3], ...[3, 4] ]
    // so output deepEquals [1, 2, 2, 3, 3, 4]

### Objects

Spreading into objects with spread loops works like regular object spread. the
expression at the end of the loop will be assigned into the resulting
object:

    flip(obj) ->
      { ...for key k, val v in obj: {[v]: k} }
<!-- -->
    mapObject(obj, f) ->
      { ...for key k, val v in obj: f(k, v) }

### JSX

Using a spread loop in a JSX `{ }`-delimited expression creates an array of JSX items, one for each time the loop reaches a terminal expression:

    <Items>
      { ...for idx i, elem item in items:
          if item.isInStock:
            <Item key={i} item={item} />
      }
    </Items>

Note that this is different than an array spread; you are not required to wrap terminal items in `[ ]` and multiple items cannot be created per iteraton.

### Elision

In other languages that have comprehensions -- CoffeeScript, for instance --
each iteration of the loop always produces an element of the output array.

In LightScript, spread `for` loops are *elisive*, meaning that if control flow
does not reach a terminal expression, no element will be added to the array.

In practice, this means you can use `if` statements inside the `for` loop to
perform filtering on the output array:

    evens(arr) ->
      [ ...for elem e in arr: if e % 2 == 0: [e] ]
    evens([1, 2, 3, 4, 5]) // [2, 4]

> By combining elision and spread, you can make each element of your input
> iterator generate zero, one, or more elements in the output object or array.
> Clever use of spread `for` can do more than `map`, `filter`, and company
> combined -- and more efficiently to boot. Promise you'll only use your new
> powers for good!

## Pattern Matching

Pattern matching is a powerful form of conditional branching that lets you branch on deep structural properties using concise syntax.

> The design for pattern matching is based loosely on [an early stage JS standard propsal](https://github.com/tc39/proposal-pattern-matching).
> Features may be added to track the proposal, and if it is adopted as a
> standard, it will naturally be supported.

<!-- -->

> As you look at the compiled code for pattern matching, you will notice a
> dependency on `@oigroup/lightscript-runtime` -- **you must add this package
> to your project's dependencies if you wish to use pattern matching!**

### Core Syntax

A pattern match begins with the keyword `match`, followed by a *discriminant*, and then a block of *cases*. The block of cases uses standard LightScript block syntax, meaning you can choose either `{ }` or `:` and whitespace to delimit your block.

The cases are set apart from each other using `|`. Each case consists of a *test* followed by a *consequent* block.

The match can end with an `else` case, which catches anything not caught by the other cases.

    DisplayedComponent = match product:
      | Shirt:
        <Shirt size={product.size} />
      | Pants:
        <Pants w={product.w} l={product.l} />
      | CoffeeMug:
        <CoffeeMug decal={product.decal} />
      | else:
        <UnknownProduct />

A `match` executes only the first matching case; there is no fallthrough. `match` can be used as an expression, in which case the value of the expression is the last value reached in the matching case, or `undefined` if no case matches.

### Cases

A case consists of an optional comma separated list of *atoms*, optionally followed by a *pattern*, optionally followed by a *guard*. Although all three of these things are optional when considered individually, at least one of the three must be present.

#### Atoms

Atoms perform basic tests on your discriminant. Multiple atoms can be separated by commas, and a discriminant that matches any one of them will pass the test.

    match something:
      | OneAtom: "it matches OneAtom"
      | Several, CommaSeparated, Atoms: "it matches at least one of the atoms"

##### Predicate Atoms

If an atom begins with a `~`, the [Tilde Call](#) syntax of LightScript is used to apply a function to the discriminant. If the function returns truthy, the atom is matched. The discriminant is implicitly placed on the left of the tilde call:

    match mellow:
      | ~isHarshed(): "not cool man"
      | else: "it's all good"

##### Class Atoms

If an atom is a class constructor, it will test whether the discriminant is
an `instanceof` the corresponding class. When `Number`, `String`, or `Boolean` are used as atoms, a corresponding `typeof` check is performed on the discriminant.

    match maybeComponentOrNumber:
      | React.Component: "it's a react component"
      | Number: "it's a number"
      | else: "it's not a react component or a number"

##### Regexp Atoms

If an atom is a RegExp, `RegExp.prototype.test` is used to test if the discriminant matches the RegExp:

    match str:
      | /^hello/: "starts with hello"
      | else: "nope"

##### Default Atoms

By default, any other expression appearing as an atom is tested for strict equality (`===`) against the discriminant.

#### Patterns

Each match case may contain a JavaScript destructuring pattern that the discriminant must match in order to satisfy the case. A pattern is introduced by `with`:

    match someProduct:
      | with { pageCount }: `it's a book with ${pageCount} pages`
      | else: "not a book"

Items matched in the destructuring are assigned to variables within the scope of the consequent.

Both object and array destructuring patterns work. Patterns can be nested or
have default values. Rest patterns can be used with arrays. A defaulted value will not be tested against.

    match mysteryItem:
      | with [ { pageCount }, ...rest ]:
        `it's an array starting with a book?`
      | with [first, second = 2]:
        "at least one and maybe two items"
      | with [plugin, { someOption } = defaultOptions]:
        "a babel plugin?"

#### Guards

A guard is a final `if` condition that must be passed for the case to be satisfied. The guard begins with the `if` keyword and can utilize variable bindings from the pattern.

    match product:
      | with { color, size, price } if size == "XL" and color == "red":
        "an extra large red shirt costs" + price
      | else:
        "something else"

### All Together Now

Combining atoms, patterns, and guards:

    shipping = match product:
      | ~isShirt() with { size } if size == "XXXXL":
        bigShirtShippingCost
      | ~isShirt():
        normalShirtShippingCost
      | ~isBook() with { pages }:
        bookShippingCostPerPage * pages
      | else:
        throw new Error("we only sell shirts and books")

### Advanced Matching Topics

#### Create-Your-Own Atoms

The behavior of atoms is determined by a new language feature,
`Symbol.matches` -- making the power of pattern matching essentially
unlimited. We outline only the default features of atoms here.

The `isMatch` function you see in the generated code actually just calls
`Atom[Symbol.matches](discriminant)` -- so by defining your own `Symbol.matches`
in your project, you can customize the behavior of pattern matching.

#### Pre-guards

You may be able to avoid expensive pattern matching checks using a pre-guard. Pre-guards are delineated using `if` and `when` at the beginning of a match case:

    result = match thing:
      | if shouldDoExpensiveStuff when ~expensiveTest():
        "yup"
      | else:
        "nope"

The rest of the tests will only run if the pre-guard passes.

#### Non-Assertive Destructuring

If you have an atom that tests whether your discriminant is a particular type of object, it may be redundant to perform additional checking for the presence of keys on that object. You can skip this check by using `as` instead of `with`.

For instance, if you know all shirts have a `size` field:

    shipping = match product:
      | ~isShirt() as { size } if size == "XXXXL":
        bigShirtShippingCost

Using `as` there skips a check for that field's presence.

## Classes

### Basic Classes

    class Animal {
      talk() -> 'grrr'
    }

    class Person extends Animal:
      talk() -> 'hello!'

### Bound Class Methods

    class Clicker extends Component:

      handleClick(): void =>
        this.setState({ clicked: true })

      render() ->
        <button onClick={this.handleClick}>
          Click me!
        </button>

Use a fat arrow (`=>`) to bind class methods.
This will be added to the constructor after a `super` call, which will be created if it does not exist.

You cannot use bound class methods if you `return super()`,
which is something you typically [shouldn't do anyway](TODO).

### Constructor and Super Insertion

If you define bound class methods with `=>`, a `constructor` method will be inserted
if one did not already exist.

LightScript will also insert `super()` for you if a `constructor` is defined
in a class that `extends` another class, and will pass along your parameters
to the base class.

    class Person extends Animal:
      constructor(foo) ->
        console.log("I forgot to call super!")

To disable `super`-insertion, define the constructor without a LightScript arrow:

    class NaughtyPerson extends Person:
      constructor(foo) {
        console.log("I don't want super called!")
      }

### Bound static methods

    class Animal:
      static kingdom() => this.name

In this example, `kingdom()` will always return `'Animal'`,
regardless of its calling context.

### Class Getters and Setters

    class Animal:
      get noise() ->
        this.sound or 'grrr'

      set noise(newValue) ->
        this.sound = newValue

See also [Object Methods](#object-methods).

### Class properties

As in ES7:

    class Animal:
      noise = 'grrr'

### Static properties and methods

As in ES7:

    class Animal:
      static isVegetable = false
      static canMakeNoise() -> true

### Decorators

As in ES7:

    @classDecorator
    class Animal:

      @methodDecorator
      talk() -> 'grrr'

## Standard Library

If you reference the name of a `lodash` method and you haven't defined that name as something else within scope, LightScript will automatically import the method from `lodash`:

    [0.1, 0.3, 0.5, 0.7]~map(round)~uniq()
    // [0, 1]

There are also methods for invoking the JavaScript operators whose syntax has been disabled in LightScript:

    looseEq(3, '3')
    // true

    2~looseNotEq('3')
    // true

    bitwiseNot(1)
    // -2


### Contents

- Every method in [Lodash v4](https://lodash.com/docs/4.17.4)
  - Note that LightScript **does not** install lodash for you;
    you must run `npm install --save lodash` if you wish to use these features.
- `looseEq(a, b)`, which uses the JavaScript loose-equality `==` to compare two variables
  (available because in LightScript, `==` compiles to `===`).
- `looseNotEq(a, b)`, which uses the JavaScript loose-inequality `!=` to compare two variables.
- all the JavaScript bitwise operators:
  - `bitwiseNot(x)`, compiles to `~x`.
  - `bitwiseAnd(a, b)`, compiles to `a & b`.
  - `bitwiseOr(a, b)`, compiles to `a | b`.
  - `bitwiseXor(a, b)`, compiles to `a ^ b`.
  - `bitwiseLeftShift(a, b)`, compiles to `a << b`.
  - `bitwiseRightShift(a, b)`, compiles to `a >> b`.
  - `bitwiseZeroFillRightShift(a, b)`, compiles to `a >>> b`.

### Overriding

If you define a variable named after a `lodash` method, your definition supersedes the automatic import:

    round(x) -> 100
    [0.1, 0.3, 0.5, 0.7]~map(round)~uniq()
    // [100]

### Disabling

To disable this feature, pass `stdlib: false` as a [compiler option](#).

To disable Lodash only, pass `stdlib: { lodash: false }` as a [compiler option](#).

### Using require instead of import

If you want `require()`  rather than `import` in the output code, add `useRequire: true` as a [compiler option](#):

```
'use @oigroup/lightscript with useRequire'
[0.5]~map(round)~uniq()
```

## Automatic Semicolon Insertion

**You never need semicolons in LightScript to separate statements.**

Instead, there are a few restrictions around edge cases:

1. For positive and negative numbers, use `+1` and `-1` instead of `+ 1` and `- 1`.
1. Regular expressions that begin with a space must use `/\ /` or `/\s/`, not `/ /`.
1. When starting a line with binary `+`, `-`, `\`, or `<`, the symbol must be followed by a space:

```
isOver100 = twoHundred
  / four
  + oneHundred
  - fifty
  < myNumber
```

More details about ASI are in [the gory details](#), which we hope you'll never need to consult.

## Tooling

### Syntax Highlighting

- Sublime Text:

    Install `LightScript` through Package Control.

- Atom:

    https://atom.io/packages/lightscript-atom

- VS Code:

    Install the `lightscript-syntax` extension:
    https://marketplace.visualstudio.com/items?itemName=lightscript.lsc

    You probably also want the `Babel ES6/ES7` extension which extends syntax highlighting to elements like JSX and Flow types:
    https://marketplace.visualstudio.com/items?itemName=dzannotti.vscode-babel-coloring

- GitHub:

    GitHub does not directly support LightScript syntax, but you can override with JavaScript highlighting, which works reasonably well. Add a `.gitattributes` to the root of your repo with the following content:
    ```text
    *.lsc linguist-language=javascript
    ```

### Linting

Linting for LightScript is implemented as an extension for the ESLint linter. First install the `@oigroup/lightscript-eslint` extension:

```sh
$ npm install --save-dev @oigroup/lightscript-eslint
```

Then configure the `.eslintrc` in the root of your project to use `@oigroup/lightscript-eslint` as the parser:

```json
// .eslintrc
{
  "parser": "@oigroup/lightscript-eslint",
  "parserOptions": {
    "sourceType": "module"
  },
  "extends": "eslint:recommended"
}
```

Then lint your code:

```sh
$ eslint --ext .js,.lsc src
```

#### Live Linting

##### Visual Studio Code

- Set up eslint for your project as above. Verify that eslint lints correctly from the CLI.
- Install the `ESLint` extension for VSCode: https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint
- Tell VSCode to live-lint LightScript files by adding the following entry to your VSCode options (workspace or global):
  ```
  "eslint.validate": ["javascript", "javascriptreact", "lightscript"]
  ```

##### Others

Only Visual Studio Code has been extensively tested at the time of this writing, however, live linting extensions for Atom and Sublime Text should work as well, as long as they are able to use local versions of `eslint`.

#### Linting with React

If you're coding with React and JSX, you will need to install `eslint-plugin-react` and add it to your `.eslintrc`:

```json
// .eslintrc
{
  "parser": "@oigroup/lightscript-eslint",
  "parserOptions": {
    "sourceType": "module"
  },
  "plugins": [
    "react"
  ],
  "extends": [
    "eslint:recommended",
    "plugin:react/recommended"
  ]
}
```

> The ESLint linter extension is still a bit immature. If you encounter any
> issues with linting, such as broken rules and crashes, feel free to open
> an issue at https://github.com/wcjohnson/lightscript/issues

## Porting from JavaScript

LightScript is a "rough superset" of JavaScript: *almost* all valid JavaScript
is valid LightScript. In a lot of cases, it should be possible to rename `.js`
to `.lsc`, run the linter, fix a few errors, and go.

This section aims to comprehensively document the cases where valid JavaScript
compiles differently (or breaks) in LightScript.

Most cases have been covered elsewhere in this documentation,
but are grouped here for convenience.

### Added keywords

`now`, `or`, `and`, and `not` are reserved words in LightScript.

### `==` and `!=`

Perhaps the biggest semantic change, `==` compiles to `===` and `!=` compiles to `!==`.

### Bitwise operators

All bitwise operators have been removed. The unary Bitwise NOT `~` has been repurposed
for [Tilde Calls](#tilde-calls).

Instead you may use the replacements (like `bitwiseNot(x)`) provided by the [standard library](#standard-library).

Bitwise assignment operators (`|=`, `&=`, `^=`, `<<=`, `>>=`, `>>>=`)
remain but may be removed as well in the future.

### ASI Fixes

See [ASI](#automatic-semicolon-insertion) for a handful of breaking syntax changes,
mainly requiring or disallowing a space after an operator.

### Do-While requires newline or semicolon

While in JavaScript, the following is valid:

    do {} while (x) foo()

It is illegal in LightScript. A `;` or newline must follow the `while` conidition,
regardless of whether parens are used.

### No newlines after `if`, `while`, `switch`, and `with`

While in JavaScript, the following are valid:

    if
      (condition) {
        result()
      }

    while
    (notDone())
    doStuff()

They are illegal in LightScript. Instead, put the condition on the same line as the `if`
or use parens like so:

    if (
      condition and
      otherCondition
    ):
      result()

    while (
      notDone() and
      notBoredYet()
    ):
      doStuff()

### Numbers with decimals

Numbers in LightScript cannot begin or end with a `.`:

    .5
<!-- -->

    5.

Instead use the explicit decimal forms:

    0.5
    5.0

### No Invisible Characters

While invisible characters are legal in strings, the only ones allowed in code
are ` ` (ascii-32), `\n` and `\r\n`. Tabs, non-breaking spaces, and exotic unicode
such as `\u8232` raise `SyntaxError`s.

### Sequence expressions require `( )`

In JavaScript, a comma-separated list of expressions with optional `( )` is parsed as a sequence expression.

The implicit parentheses introduce grammatical ambiguities with [Bang Calls](#). In LightScript, the `( )` is required:

    // illegal in LightScript
    a, b
<!-- -->
    // legal in LightScript
    (a, b)

### Labeled expressions are illegal

JavaScript [labeled statements](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/label) can no longer be used with expressions:

    label: expr

Using labeled jumps is bad practice and should be discouraged, but if you really need them, you can still label loop statements:

    outerLoop: while true:
      innerLoop: while true:
        continue outerLoop

## Compiler Options

Some functions of the compiler can be further configured using an options
interface. Whenever the docs refer to a compiler option, it can be configured
as shown

### Via `.babelrc`

A configuration object is passed to the compiler as the second element of the `preset` array. Additional keys added to this object convey compiler option settings.

```json
// babelrc
{
  "presets": [
    [
      "@oigroup/babel-preset-lightscript",
      {
        "env": { "targets": { "ie": 10 } },
        "placeholderArgs": true, // option
        "stdlib": false // option
      }
    ]
  ]
}
```

### Via directives

Compiler options can be configured for individual files by placing a JavaScript
directive string at the top of the file:

    // myapp.lsc
    'use @oigroup/lightscript with placeholderArgs'
    -> _

Naming an option sets it to `true`. `:` can be used to provide a value other than `true`:

    // myapp.lsc
    'use @oigroup/lightscript with placeholderArgs, placeholder: "$$"'
    -> $$1

## Non-Standard Features

The @oigroup branch contains features that the main LightScript language, maintained by @rattrayalex, has either rejected or not evaluated for inclusion. In order to maintain compatibility with core LightScript, these features are off by default and must be enabled via [compiler options](#).

### Existential Operator

Compiler option `existential: true` enables the existential operator. Similar to CoffeeScript, a suffix `?` will loosely compare any expression with `null`:

    'use @oigroup/lightscript with existential'
    if not entity.nonNullableField?:
      throw new Error("non-nullable violation")

As opposed to a check for falsiness, an existential check does not fail on `false`, `''`, or `0` -- it only fails when a value is `null` or `undefined`.

**Status:** This feature has been explicitly rejected by the upstream language, see https://github.com/lightscript/babylon-lightscript/pull/26

### Placeholder Args

Compiler option `placeholderArgs: true` enables placeholder arguments. An extension of features like Kotlin's `it`, they enable a terse syntax for functional programming.

A placeholder (by default, `_`) appearing in a function without arguments implicitly adds an argument to that function. The placeholder will be replaced with the argument when the function is called:

    'use @oigroup/lightscript with placeholderArgs'
    lastNames = people.map(-> _.lastName)

When a placeholder is followed by an integer literal `n`, it represents the `n`'th (zero-indexed) argument of the function.

    'use @oigroup/lightscript with placeholderArgs'
    add = -> _0 + _1

Placeholders with indices cannot be mixed with plain placeholders:

    'use @oigroup/lightscript with placeholderArgs'
    add = -> _ + _1

Placeholders can't be used on functions that already have explicit args:

    'use @oigroup/lightscript with placeholderArgs'
    add(x) -> x + _

The placeholder can be changed by passing a `placeholder` compiler option. (This is helpful if e.g. you use `_` for `underscore`):

    'use @oigroup/lightscript with placeholderArgs, placeholder: "$$"'
    add = -> $$0 + $$1

### Subscript Indentation

LightScript proper requires that subscripts on a new line must be indented from
the previous line, on pain of a syntax error:

    doAsyncThing()
    .then! (result) -> print! result
    .catch! (err) -> print! err

However, a lot of people use that coding style, and the compiler shouldn't be enforcing something else if it doesn't have to. Enter compiler option `noEnforcedSubscriptIndentation: true`, which disables that requirement:

    'use @oigroup/lightscript with noEnforcedSubscriptIndentation'
    doAsyncThing()
    .then! (result) -> print! result
    .catch! (err) -> print! err

When using this option, the ASI rules for brackets are similar to general ASI in LightScript -- the opening bracket must appear on the same line as the expression being indexed:

    a
      [0]
<!-- -->
    'use @oigroup/lightscript with noEnforcedSubscriptIndentation'
    a
    [0]
<!-- -->
    'use @oigroup/lightscript with noEnforcedSubscriptIndentation'
    a[
      0
    ]

Also note that because of the parsing conditions involving bang calls, even when this option is enabled, if you want to subscript an argument of a bang call, it must be indented more deeply than the argument:

    'use @oigroup/lightscript with noEnforcedSubscriptIndentation'
    bang!
      arg
      .sticksToBang

    bang!
      arg
        .sticksToArg


## The Gory Details

> This is the part of the manual we hope you don't have to read. If you're here
> because something went wrong for you and not out of morbid curiosity, we want to
> hear from you. Feel free to [join the LightScript Gitter chat](https://gitter.im/lightscript/Lobby) and mention me `@wcjohnson`.

### ASI

In vanilla JavaScript, ASI tends to fail on statements beginning with `[`, `(`, `/`, `+`, or `-`. ES6 and JSX each introduce an additional ambiguity: ``` ` ``` and `<`, which are handled as well.

LightScript solves each issue in a slightly different way,
though each fix is essentially an encoding of stylistic best-practice into the syntax itself.

In practice, if you stick to community-standard code style,
you should not have to worry about any of this; they are documented for completeness.

#### `+` and `-`: binary vs. unary

There are two possible interpretations of this code:

    1
    -1

It could either be a `1` statement followed by a `-1` (negative one)
statement, or a single `1 - 1` statement.
JavaScript chooses `1 - 1`, which is typically undesired.

This is because `+` and `-` take two forms in JavaScript:
*binary* (add or subtract two numbers) and *unary*
(make a number positive or negative).

To resolve this ambiguity, LightScript requires that *unary* `+` and `-`
are not separated by a space from their argument.
That is, `-1` is "negative one" while `- 1` is invalid LightScript.

With this restriction in place, it easy to give preference to the unary form
when a `+` or `-` begins a line:

    1
    + 1

    1+1

    1
    +1

Only the last example is a deviation from JavaScript,
which would interpret the two lines as `1 + 1`.

Again, beware that unary `+` and `-` cannot be followed by a space in LightScript:

    - 1

Without this fix, it would be difficult to implicitly return negative numbers:

    negativeThree() ->
      three = 3
      -three

(This would be `const three = 3 - three;` in JavaScript).

Similarly, it would be difficult to have lists with negative numbers:

    numbers = [
      0
      -1
      -2
    ]

(Without this ASI fix, that'd be `const numbers = [0 - 1 - 2];`).

#### `/`: division vs. regular expression

In JavaScript, the following code throws a SyntaxError:

    let one = 1
    /\n/.test('1')

This is because it tries to parse the `/` at the start of the second line as division.
As you can see, LightScript does not share this problem.

LightScript makes a slightly crude generalization that draws from the same strategy
as `+` and `-` (above): Regular Expressions can't start with a space character (` `):

    / \w/.test(' broken')

This doesn't happen very often, and when it does, can be trivially fixed
by escaping the space or using a `\s` character:

    /\ \w/.test(' not broken')

    /\s\w/.test(' not broken')

Similary, a division `/` that starts a line cannot be followed by a space:

    1
    /2

This space is not required when the `/` does not start a line:

    1/2

    1
    / 2

#### `(`: expressions vs. function calls

This is perhaps the most frequently problematic ASI failure, and the most easily fixed.

In JavaScript, the following code would try to call `one(1 + 1)`, which is not what you want:

    two = one + one
    (1 + 1) / 3

In LightScript, the opening paren of a function call must be on the same line as the function:

    doSomething(
      param)

    doSomething (
      param
    )

    doSomething
      (param) // oops!

#### `[`: index-access vs. arrays

In JavaScript, the following code would try to access `one[1, 2, 3]` which isn't what you want:

    two = one + one
    [1, 2, 3].forEach(i => console.log(i))

That's because you often do see code like this:

    firstChild = node
      .children
      [0]

In LightScript, accessing an index or property using `[]` requires an indent:

    node
      .children
      [0]

    node
    .children
    [0] // oops!

The required indent is relative to the line that starts a subscript chain.

Note that this rule also applies to the "numerical index access" feature:

    node
      .children
      .0

    node
    .children
    .0 // oops!

#### `<`: less-than vs. JSX

In JavaScript, the following would be parsed as `one < MyJSX` and break:

```
two = one + one
<MyJSX />
```

LightScript solves this in a similar manner to `+`, `-`, and `/`:
a less-than `<` that starts a line must be followed by a space:
```
isMyNumberBig = bigNumber
  <myNumber
```
is broken, but this works:

    isMyNumberBig = bigNumber
      < myNumber

#### ``` ` ```: tagged vs. untagged templates

In JavaScript, the following would be parsed as ```hello`world` ```:

    hello
    `world`

As with function calls, a tagged template expression in LightScript must have
the opening ``` ` ``` on the same line as the tag.

Unfortunately, there are a few ambiguous corner-cases.
You are unlikely to hit them and there are easy fixes.

### Known Grammar Ambiguities

#### Colons, Arrow, and Flow Types

If you have an `if` whose test is a function call,
and whose consequent is an anonymous arrow function, eg;

    if fn(): (x) => 4

it will parse as a function `fn() => 4` with Flow return type annotation `x`,
and then throw a SyntaxError: `Unexpected token, expected :`.

This can be corrected by wrapping the `if` test in parens:

    if (fn()): (x) => 4


### Blocks and Objects

`{ }` does double duty in JavaScript, delimiting both blocks of code and objects. When LightScript encounters `{ }`, it tries to use context to determine whether you meant to write an object or a code block. We hope this process "just works" for everyone, but in the event it doesn't, here's a rundown of the detailed rules at play.

- **Whenever plain JavaScript syntax is used with `if`, `for`, `do`, or `while`, the following `{ }` is always treated as a block of code:**

    ```
    z = 3
    x = if (true) { z }
    // x === 3
    ```

- **Whenever LightScript colon syntax is used with `if`, `for`, `do`, or `while`, the following `{ }` will first be treated as an object. If it fails to parse as an object, it will be treated as a block of code:**

    ```
    z = 3
    x = if true: { z }
    // x deepEquals { z: 3 }
    ```

- **Whenever `{ }` is encountered elsewhere, it is first treated as an object. If it fails to parse as an object, it will be treated as a block of code:**

    ```
    a; { b } = c
    {
      d = e
    }
    ```

- **If something in `{ }` cannot be parsed as an object or a block, the error that allows the parser to get furthest in the source code is displayed.**

  In this example, `while` is illegal as an expression in an object, but `c: d()` is illegal in a block of code. Since that error is further along in the source, that is the error that gets reported.

    ```
    {
      a: while true: b
      c: d()
    }
    ```
