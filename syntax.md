

# Introduction to Purescript

- Purescript is:
  - A purely functional programming language based on haskell
  - compiles to/interacts with Javascript.
  - Strong, statically typed
  - Whitespace driven (kinda like python)
  - Expression language
  - Immutable, Pure, and Total by default

- Advantages over JavaScript:
  - Turn runtime errors into compile time errors
  - Easier to make changes without breaking code
  - Fully declarative (get more done with less)

---

# Basic Syntax

Values/constants are declared with an `=`

```purescript
x = 5

y = x * 2
```

Functions are also declared using an `=` 

```purescript
add a b = a + b

addOne b = add 1 b

two = addOne (3 - 2)
```
**JS equivalents:**
```javascript
const add = (a, b) => a + b

const addOne = b => add(1, b)

const two = addOne(3 - 2)
```

Purescript looks a bit like coffeescript.

Each function parameter/argument is seperated only by spaces.

A function identifier on the left of `=` will always return the results on the right.

A function or value should always be equivalent to it's representation - This is called referential transparency.

The language has many inline operators, and you can create your own

lightweight, customizable syntax 
  + referential transparency 
  + types, currying
  + Equational (algebraic) reasoning

---

# Function composition  

`<<<`

If we have a function `f` and another function `g`

AND the output type of `f` matches the input type of `g`, then we can compose them as below

```purescript
f (g x)

f <<< g
```

`<<<` takes to functions and returns a new function with their composition

the resulting function must be _commutative_

---

# Function application

`$`

A function to the left of the operator is applied to the result of the expression on the right.

```purescript
sqrt (add 10.0 (round 15.25)))
```
can be rewritten as
```purescript
sqrt $ add 10.0 $ round 15.25
```


---

# Curry By Default

each function in purescript takes exactly one input argument

rather than taking additional arguments. it is automatically curried and returns a new function which itself takes one argument, and so on until the result is not a function, so

```purescript
addThenSub :: Int -> Int -> Int -> Int
addThenSub x y z = (x + y) - z
```
can become a new function 
```purescript

add5ThenSub = addThenSub 5

subFromSeven = addThenSub 5 2
--or
subFromSeven' = add5ThenSub 2
```

and finally...
```purescript
result = subFromSeven 2
-- or
result' = addThenSub 5 2 2
```

JS equivalents
```javascript
const addThenSub = x => y => z => (x + y) - z

const add5ThenSub = addThenSub(5)

const subFromSeven = addThenSub(5)(2)

const result = addThenSub(5)(2)(2)
```
note the use of `'` in identifiers,  this is completely legal and you will see it a lot. it is commonly pronounced `prime`.

---

# Equational Reasoning, Everything is an Expression

- Nothing gets mutated silently
- `if true then 1 else 2` evaluates to 1
- `+` is a function, so is `-`

we can directly compose operators as functions because they are just functions.

we can use the Anonymous Argument `_` to acheive this:

```purescript
twoPlusOneMinusOne = (_ - 1) <<< (_ + 1) $ 2
```

While this allows for a huge amount of abstraction, it also allows us to write non-repetitive, type safe code with maximum expressiveness.

In combination with the type checker, we can safely write polymorphic code without fear of exceptions caused by missing object fields, `null`, undefined, `NaN`, and mutation.

---

# FFI

Occasionally, working with unsafe javascript is more convenient, such as working with a library written in JS.

Purescript offers typed FFI to javascript to allow for flexibility and broader ecosystem integration.

Purescript also offers tools necessary to make an unsafe JS call into a safe Purescript function.

---

# The Type Checker

- Purescript is a compiled, Strongly Typed, and Statically Typed language:

In this paradigm, types are powerful enough to be the specification for our program, and often provide tests _for free_. 

Strong : types do not change/coerce at runtime silently

`1 + "1"` is a type error

Statically typed : types are known at _compile time_

A type inference algorithm, in addition to a few explicit type signatures, is used by the compiler to determine whether or not a program is correct.

Type inference means that the type checker is _often_ able to know the type of a variable without an
explicit annotation.  However there are times when you have more information than the compiler, so some occasional type signatures are necessary

Types do not exist at runtime.

We will see the `::` operator which you can read as '<identifier> Has Type <type>'

a given `Type` will generally correspond to some _term-level_ expression which can generally be verified to be of that type.

---

# Primitive Types

These are Javascript types with safe wrappers available in `Prim`, which is secretly imported in every purescript file.

There are additional simple types that come from purescript in the `Prelude`

- Boolean
```purescript
myBool :: Boolean
myBool = true
```

- Int (32bit signed int)

```purescript
myInt :: Int
myInt = 8
```
- Number (64bit float)
```purescript
myNum :: Number
myNum = 8.0
```

- Char 
```purescript
myChar :: Char
myChar = 'a'
```

- String (not a list of Char)
```purescript
myString :: String
myString = "hello"
```

- Array a
```purescript
myIntArray :: Array Int
myIntArray = [1,2,3]
```

- Record
```purescript
myRec :: { field :: String, field2 :: Int }
myRec = {field: myString, field2: myInt }
```

---

# Prelude Types

These types must be explicitly imported with prelude:
`import Prelude`

These do not directly correspond with a javascript type, 

`unit :: Unit`
- this is like true or false but only one value
- used for functions that don't need a return value
- similar to haskell's `()`

`Void`
- used to denote a type that cannot exist, is is unused

`Ordering`
- has three values: `GT`, `LT`, and `EQ`
- used for comparison, drives `<` and `>` machinery

___

# How to Read a Simple Function Type signature

`simpleLT :: Int -> Int -> Boolean`

`->` is the inline operator to construct a function, it's _right associative_ so brcket behaviour goes to the right.

This is equivalent:

`simpleLT :: Int -> (Int -> Boolean)`

---

### Functions can also be arguments

You can use round braces to accept a function argument such as this
```purescript
doesItPass :: (Int -> Boolean) -> Int -> Boolean
doesItPass f i = f i
-- or
doesItPass = $
```

In this situation the braced area appears to the _left_, letting us know that our function takes another function of the type `Int -> Boolean` as an argument.

This is a simple example of taking a function as an argument however there are many more useful examples.

---

# Type variables and Ad-hoc polymorphism

Up until now all of the types we have used have been fully known, This is known as a concrete type such as `String` However, writing all of our code this way will be repetitive.

We can write code that operates on multiple types _polymorphically_
```purescript
doesItPassPoly :: forall a. (a -> Boolean) -> a -> Boolean
doesItPassPoly p x = p x
```

`forall a.` introduces the type variable `a` to the current scope.

Notice that the _argument_ name `x` does not match the name of `a` - they can be named differently, but the type checker knows that `x` should have the type `a` and indeed that `p` takes an argument of type `a`

In this case, we've declared that our function can handle an `a` of ANY type.

This means that we can't actually do much of anything to `a` since we don't know what it is, which is a fairly strong guarantee that we won't do anything misleading with the _term_ corresponding to type `a`

---

# TypeClasses & Constraints

If we want to _do_ something with a type variable in our function, we can qualify it with a `Constraint`.

Constraints tell us that the type we are dealing with is a member of a club that allows use to perform certain actions, even if the type itself is not known. 
`Show` - allows us to print the data as a string
`Eq` - allows us to compare two values of that type for equality
`Ord` - allows us to rank values as Greater Than, Less than, Equal.

These can allow us to perform operations on a variable type
```purescript
equal :: forall a. (Eq a) => a -> a -> Boolean
equal x y = x == y
```

Here we can see that the `Eq` typeclass is allowing us to use `==` as a comparison operator.

The type checker can verify at compile time that any type we supply as an argument to `equal` is a member of the TypeClass `Eq`.

The type checker will also verify that the first and second argument above are of the same type. (type variables must match up, they are not a wildcard each time).


---

# Algebraic Data Types,   Type & Data Constructors
## Parameterized types (they're curried just like functions)

---

# Sum Types

---

# Product Types

---
# Row Types
---

# Type Synonyms

---

# newtypes

---

# data types

---

#function syntax
