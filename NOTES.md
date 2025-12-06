# PlumJam's Haskell Learning Note

Haskell for Imperative Programmers by Philipp Hagenlocher:
<https://www.youtube.com/watch?v=Vgu82wiiZ90&list=PLe7Ei6viL6jGp1Rfu0dil1JH1SHk9bgDV>

Throughout this document, return values are indented and prefixed with `=>`.

I do not recommend learning from this document. Instead, go and watch the
excellent series linked above. These are my personal notes - they will be wrong
in places and they will change.

## Video #1 - Basics

In functional programming (more specifically Haskell), we are interested in pure
mathematical functions.

Functions have inputs, outputs and nothing else.

Data is completely immutable meaning it can not be changed once it is defined.
This leads to no/less side-effects.

Declarative, not imperative.

It's easier to verify a pure function because it can be proven mathematically.

A simple example of functional vs imperative can be seen below:

Functional:

```haskell
sum [] = 0
sum (x:xs) = x + sum xs

-- or

sum = foldr (+) 0
```

Imperative:

```c
int sum (int *arr, int length) {
  int i, sum = 0;

  for(i = 0; i < length; i++) {
    sum += arr[i]
  }

  return sum;
}
```

Haskell is lazy evaluated, as opposed to strict evaluation.

Examples of both approaches are below. Assume each function takes 1 year to
finish.

In Haskell (with lazy evaluation) `x`, `y` and `z` are assigned the result of
the corresponding functions but are not evaluated right when they occur in the
code because they don't have to be right now. Haskell observes what we NEED
which is `z` in this case, otherwise nothing else can be done. So `z` would be
evaluated and take 1 year. Once `z` is known we only need 1 of the other
functions `x` or `y` to finish the program so it would only take 2 years. It's
very important to understand how lazy evaluation works. Haskell only evaluates
what is needed.

```haskell
func0 arg =
    let x = func1 arg
        y = func2 arg
        z = func3 arg
    in
    if z then x else y
```

In a strictly evaluated language, the following example would take 3 years. Note
that it can obviously be done better and also only take 2 years but this example
helps demonstrate lazy evaluation.

```c
int func(int arg) {
  int x = func1(arg);
  int y = func2(arg);
  int z = func3(arg);

  if (z) {
    return x;
  } else {
    return y;
  }
}
```

## Video 2 - Functions, Types, let and where

### Functions

Function definitions look something like this:

```haskell
name arg1 arg2 ... argn = <expr>
```

Arguments are not wrapped in parentheses or separated by commas and the
arguments are followed by `=` and an expression that returns. There are no
return statements in the traditional sense and the expression IS the return
value.

Function applications look like this:

```c
name arg1 arg2 ... argn
```

Examples:

```haskell
in_range min max x =
  x >= min && x <= max

in_range 0 5 3
  => True

in_range 4 5 3
  => False
```

### Types

Haskell is statically and strictly typed.

#### Basic

```haskell
x :: Integer
x = 1

y :: Bool
y = True

z :: Float
z = 3.1415
```

#### Function Types

Function types are written as a arrow-connected line of types for your arguments
and finally the return value. For example:

```haskell
in_range :: Integer -> Integer -> Integer -> Bool
in_range min max x = x >= min && x <= max

in_range 0.5 1.5 1 -- Type error
in_range 0 5 3     -- Correct
```

#### Functions (let)

Sometimes we want to save the result of an expression in a variable and return a
final result like this:

```haskell
in_range min max x =
  in_lower_bound = min <= x;
  in_upper_bound = max >= x;
  return (in_lower_bound && in_upper_bound);
```

This is *imperative* NOT *declarative*! Also note this is not proper Haskell
syntax and is only provided as an example. How can we achieve a similar approach
in Haskell? With `let` bindings, like this:

```c
in_range min max x =
  let in_lower_bound = min <= x
      in_upper_bound = max >= x
  in
  in_lower_bound && in_upper_bound
```

We bind the result of an expression to a name like `in_lower_bound` and then
produce a final result (the last expression) in relation to whatever names we
defined. "Let this be this in this."

#### Functions (where)

Another approach is to use `where` bindings to achieve the same as the `let`
bindings seen in the previous section:

```haskell
in_range min max x = ilb && iub -- in lower bound && in upper bound
  where
    ilb = min <= x
    iub = max >= x
```

It's almost a reversed and more concise approach. In my opinion, I prefer the
`let...in` approach.

It's a good time to re-highlight that lazy evaluation can be seen here too. The
bindings are only evaluated when they are needed.

#### Functions (if)

We obviously need a way to control the flow of the program. In this example,
we're achieving the same effect as the boolean `&&` but it's important to know
it exists.

```haskell
in_range min max x =
    if ilb then iub else False
    where
      ilb = min <= x
      iub = max >= x
```

#### Functions (infix)

Infix functions are when functions are written *between* arguments. Using `ghci`
we can pass `:t <function_name>` and get it's type.

```sh
$ ghci> :t (+)
(+) :: Num a => a -> a -> a
```

This looks confusing as fuck but apparently it will be covered more later.

It's good to know that functions can be written this style too.

```haskell
add a b = a+b -- Definition.

add 10 20   -- Is equivalent to...

10 `add` 20 -- ...this.
```

## Video 3 - Recursion, Guards, Patterns

### Recursion

While loops, for loops, and any other loops do not exist in Haskell.

If we want something to *loop* we have to use recursion, aka a function calling
itself with different arguments (or the same arguments but that doesn't make
much sense).

Basic boilerplate of what recursion looks like in Haskell.

```haskell
name <args> = .. name <args`> ...
```

An example using prime numbers and a factorial function:

```haskell
fac n =
    if n <= 1 then
        1
    else
      n * fac (n-1)
```

If `n` is less than or equal to 1 return 1. However, if `n` is greater than 1
(implied) then we recursively call `fac` (short for factorial) with `(n-1)`.

```haskell
fac 3
-- 3 * (fac 2)
--- 3 * (2 * (fac 1))
---- 3 * (2 * 1)
----- 3 * 2
  => 6
```

In plain English:

- 3 is not \<= 1 so call 3 * fac 2 (3 - 1 = 2)
- 2 is also not \<= 1 so call 3 * 2 * fac 1 (2 - 1 = 1)
- 1 is \<= 1 so no more recursion is needed and 1 is returned
- Finally left with 3 * 2 * 1
- => 6

### Guards

`if...then...else` is just one expression and there are other ways to do the
same thing. Another good way for handling a function that works with boolean
expressions is by using guards. Another benefit of guards is that you are not
limited only to `if...then...else`, you can add as many guards as you need.

```haskell
fac n
  | n <= 1    = 1             -- boolean expression
  | otherwise = n * fac (n-1) -- otherwise
```

Otherwise is a constant that always evaluates to true and thus will always be
matched and evaluated if the execution reaches it.

This may be incorrect but from my understanding it is effectively 2 function
definitions in 1. If `n` is \<= 1, the first guard runs and the expression
evaluates to 1. If `n` is not \<= 1, the second guard runs and the expression
evaluates to `n * fac (n-1)`.

### Pattern Matching

For integers, pattern matching isn't that interesting, as you can see here:

```haskell
is_zero 0 = True
is_zero _ = False
```

These implementations are partial because the first definition for 0 *only*
works for 0. So the other definition is needed to handle all other cases with a
wildcard.

The underscore acts as a wildcard, similarly to Rust and Nu and is used to match
any other pattern that is not covered by a prior pattern.

### Accumulators

Another way of implementing recursive functions is with accumulators.

In this example we say that the `fac` function is the *return* value of an
auxillary `aux` function:

```haskell
fac n = aux n 1
  where
    aux n acc
      | n <= 1    = acc
      | otherwise = aux (n-1) (n*acc)
```

In the previous Guards section, we first made the recursive call and *then*
performed a computation. In this example, we compute `n` - 1 and `n` * `acc`
first and *then* make the recursive call. This is called a tail recursive call
because there is no operation after the recursive call.

Tail recursive algorithms should always be our goal. Non-tail recursive
algorithms can cause a stack overflow because for every new recursive call that
is made, you have to put a new stack frame on the stack. For this example, we
obviously don't want our `fac` function to be bounded by the size of our stack.
The reason the example above does not end up causing a stack overflow is because
a compiler, after the first call, ends up in a `while(True)` loop, like this:

```c
fac n:
  acc = 1;

  while (True) {
    if (n <= 1) {
      return acc
    } else {
      acc = n*acc
      n = n-1
    }
  }
```

Thus we do not end up with a possible stack overflow because no new stack frame
needs to be added to the stack per recursive call. NOTE: This is still a bit
hazy to me, need to learn more.

Most algorithms in Haskell are recursive so it's important to understand
recursion.

## Video 4 - Lists and Tuples

Lists in Haskell can only be of 1 type (as it should be). For example:

```haskell
[1,2,3,4,5] :: Integer
```

This is not possible:

```haskell
[1,2,"hello",false] :: WhatTheFuckAreYouEvenDoing
```

Lists can be constructed by constructors of which there are 2. The first one is
an empty list, the second is with "prepend" or "colons":

```haskell
[]

x : xs
```

The colon separates `x` which will be prepended to an existing list `xs`. It
doesn't have to be `x` and `xs` but it's the standard way of doing it.

We can do the following which prepends 1-5 to an empty existing list:

```haskell
-- [1,2,3,4,5]
1 : 2 : 3 : 4 : 5 : []
```

If the amount of elements in a list is finite or small enough, you can simply
create a list like this:

```haskell
[1,2,3,4,5]
```

### Generating a List

It can be done with a function like this where `asc` refers to ascending:

```haskell
asc :: Int -> Int -> [Int]
asc n m
  | m < n = []
  | m == n = [m]
  | m > n  = n : asc (n+1) m

asc 1 3
  => [1,2,3]
```

So as we can see, we take 2 inputs `n` and `m` and then output a list of
ascending numbers from `n` to `m`.

- If `m < n` it doesn't make sense so we just return an empty list.
- If `m == n` we return a list containing that single number.
- If `m > n` we recursively construct the new list by prepending `n` and then
  recursively calling `asc` with an incremented `n`.

This isn't necessary but I took note of it for learning. There are many
functions for lists defined in the `Data.List` module which can be imported like
this:

```haskell
import Data.List
```

### Functions on Lists

`head` gives the first element of a list:

```haskell
head :: [a] -> a
head [1,2,3,4,5]
  => 1
```

`tail` gives the list without the first element:

```haskell
tail :: [a] -> [a]
tail [1,2,3,4,5]
  => [2,3,4,5]
```

This is exactly the split that the colon constructor does - we have a head that
was prepended to the list and then the tail which is the rest of the list.

`length` gives the length of a list:

```haskell
length :: [a] -> Int
length [1,2,3,4,5]
  => 5
```

`init` gives the list with the last element removed:

```haskell
init :: [a] -> [a]
init [1,2,3,4,5]
  => [1,2,3,4]
```

Removed in this cases does not mean that the element is deleted, instead it is
just not displayed because the rest of the list is *copied* but with the last
element removed. Remember that every data type in Haskell is immutable.

`null` tells us whether a list is empty or not:

```haskell
null :: [a] -> Bool
null []
  => True

null [1,2,3,4,5]
  => False
```

This is a very important function because something like `head` can not be
called on an empty list because there is no first element to return. So it's
good to check if the list is empty before performing other operations on the
list.

`a` is a polymorphic type here which allows the list to be of any type so we can
use it for a list of strings, integers, and even other lists.

### Functions on Lists of Booleans

`and` gives a boolean indicating if a list contains only `True`.

```haskell
and :: [Bool] -> Bool
and [True, False, True]
  => False
```

`or` gives a boolean indicating if a list contains both `True` or `False`.

```haskell
or :: [Bool] -> Bool
or [True, False, True]
  => True
```

### List Comprehension

List comprehension allow us to cleanly build new lists from an existing list or
lists.

The structure looks like this:

```haskell
[ <gen> | <elem> <- <list>, ..., <guard>, ... ]
```

In this example we take each element in the `[1,2,3]` list, multiply it by 2 and
get a new list `[2,4,6]`:

```haskell
[ 2*x | x <- [1,2,3] ]
  => [2,4,6]
```

In this example we also take each element in the `[1,2,3]` list and multiply it
by 2 but only perform this multiplication on elements where `x > 1` which acts
as a guard:

```haskell
[ 2*x | x <- [1,2,3], x > 1 ]
  => [4,6]
```

Based on the structure above, you can see it is possible to do this on multiple
lists with multiple guards in one expression:

```haskell
[ (x,y) | x <- [1,2,3], y <- ['a','b'] ]
  => [(1,'a'),(1,'b'),(2,'a'),(2,'b'),(3,'a'),(3,'b')]
```

This ends up working like a nested for loop where you first take the first index
of the first list, then go through the second list entirely, then repeat again.
Although, in the Haskell example we are using tuples `(x,y)` but I don't care,
it was just a useful comparison for me.

```javascript
let list1 = [1,2,3]
let list2 = ['a','b']
let list3 = []

for (let i = 0; i < list1.length; i++) {
  for (let j = 0; j < list2.length; j++) {
    list3.push((list1[i],list2[j]))
  }
}
  => [[1,'a'],[1,'b'],[2,'a'],[2,'b'],[3,'a'],[3,'b']]
```

This works for any number of lists.

### List Patterns

On lists, unlike integers we checked earlier, pattern matching is way more
interesting.

```haskell
sum :: [Int] -> Int
sum [] = 0
sum (x:xs) = x + sum xs
```

```haskell
sum [1,2,3]
-- 1 + sum [2,3]
--- 1 + (2 + sum [3])
---- 1 + (2 + (3 + sum []))
----- 1 + (2 + (3 + 0))
------ 1 + (2 + 3)
------- 1 + 5
  => 6
```

The base case is an empty list `[]` which evaluates to 0.

Then we match a non-empty list with any number of elements where the first
element is `x` and the rest of the list is `xs` think of `head` and `tail` from
earlier. When we match a non-empty list, we take `x` and add a recursive call of
`sum xs` to it. This recursion takes place until an empty list is found `[]` and
the base case is satisfied.

```haskell
evens :: [Int] -> [Int]
evens []         = []
evens (x:xs)
  | mod x 2 == 0 = x : evens xs
  | otherwise    = evens xs
```

```haskell
evens [1,2,3,4]
-- evens [2,3,4]           discard 1
--- 2 : evens [3,4]
---- 2 : evens [4]         discard 3
----- 2 : (4 : evens [])
------ 2 : (4 : [])
------- 2 : [4]
  => [2,4]
```

The base case is an empty list `[]` which evaluates to an empty list `[]`.

Then we match a non-empty list with any number of elements where the first
element is `x` and the rest of the list is `xs` - again, remember `head` and
`tail` from before. When we match a non-empty list we take `x`, see if it is
divisible by 2 and if it is, we prepend with `:` to the rest of the expression.
We then make a recursive call of `evens xs` which is calling `evens` on the rest
of the list. Odd numbers are discarded and in the end we get a line of
prepending `:` that ends with an empty list `[]`. This then builds a final list
containing only even numbers when the expression is evaluated by prepending each
number to the empty list at the end.

### Tuples

Tuples are a way to have multiple values of potentially differing types in a
single value. Unlike lists, tuples have a fixed number of elements (immutable)
so they can't be prepended to like we can using `:` for lists. Their length is
also finite for the same reason.

```haskell
(1, 2) :: (Int, Int)
(1, 1.5) :: (Int, Float)
("Hello world", True) :: (String, Bool)
-- etc...
```

Pattern matching on tuples can be seen in the example below. Note that `fst` and
`snd` are part of Haskell, thus you don't need to create them yourself.

`fst` - First

```haskell
snd :: (a,b) -> b
snd (x,_) = x
```

`snd` - Second

```haskell
snd :: (a,b) -> b
snd (_,y) = y
```

It can also be done in bindings such as `let` which can be useful for splitting
a tuple:

```haskell
let (x,y) = (1,2) in x
  => 1
```

To wrap up everything from this video, this function provides the sum of every
tuple in a list using list comprehension.

```haskell
addTuples :: [(Int, Int)] -> [Int]
addTuples xs = [ x+y | (x,y) <- xs ]

addTuples [(1,2), (2,3), (100,100)]
-- [ x+y | (x,y) <- [(1,2), (2,3), (100,100)] ]
--- [ (1+2), (2+3), (100+100) ]
  => [3,5,200]
```

## Video #5 - List Exercises

Don't worry about the `(Eq a)` type class yet, it will be covered later
according to the video.

### Exercise 1

TODO: Tidy up this example and explain it better.

Description:

"Create a function `elem` that returns True if an element is in a given list and
returns False otherwise."

This function exists in Haskell.

Solution:

```haskell
elem :: (Eq a) => a -> [a] -> Bool
elem _ []     = False
elem e (x:xs) = (e == x) || (elem e xs)
```

The wildcard `_` matches any element and returns false. We could name it but
this way it is clearly shown that the value is irrelevant. If the list is empty,
we return `False`.

If there is element `x` in the list, we compare it to element `e` and if they
are the same, we return `True`.If they are not the same `elem` is called again
with the current `e` and the rest of the list `xs`.

### Exercise 2

Description:

"Create a function `nub` that removes all duplicates from a given list."

This function exists in Haskell.

Solution:

```haskell
nub :: (Eq a) => [a] -> [a]
nub [] = []
nub (x:xs)
  | x `elem` xs = nub xs -- Or elem x xs -- Here we use infix notation that we discussed earlier.
  | otherwise   = x : nub xs
```

If the list is empty, return an empty list because there obviously is no
duplicates.

If the list contains data, we first check if `x` is an `elem` in `xs` and if it
is we have a duplicate, so we don't add it to our recursive call. If the element
is not a duplicate, we build a new list where `x` is prepended `:` to the
recursive call `nub xs`.

### Exercise 3

Description:

"Create a function `isAsc` that returns True if the list given to it is a list
of ascending order."

Solution:

```haskell
isAsc :: [Int] -> Bool
isAsc []  = True
isAsc [x] = True
isAsc (x:y:xs) =
  (x <= y) && isAsc (y:xs)
```

If there are no or one elements in the list, the list is in ascending order.

Patterns can be finitely recursive so instead of needing to do `x:xs` and `x:xs`
again, we can chain them with `x:y:xs` where `x` is the first element and `y` is
the second element. So what we have is a list with at least 2 elements and the
rest of the list `xs`. We compare the first 2 elements, check if they are
ascending and if they are, we use the boolean `&&` to recursively call `isAsc`
with the second element `y` and the tail of the list `xs`.

### Exercise 4

NOTE: I need to come back to this, it was way too confusing. I took the solution
from the video anyway.

Description:

"Create a function `hasPath` that determines if a path from one node to another
exists with a *directed* graph."

Solution:

```haskell
hasPath :: [(Int, Int)] -> Int -> Int -> Bool
hasPath [] x y = x == y
hasPath xs x y
  | x == y    = True
  | otherwise =
    let xs' = [ (n,m) | (n,m) <- xs, n /= x ] in
    or [ hasPath xs' m y | (n,m) <- xs, n == x ]
```

The first argument is a list of the edges, second is the start node, third is
the end node, and it returns `True` or `False` if their is a direct path or not.

If we have no more paths `[]` we just have to check that the start node is equal
to the end node `x == y`, if so a path exists between them.

TODO: Finish describing the function.

## Video #6 - Higher Order Functions & Anonymous Functions

### Higher Order Functions

```haskell
app :: (a -> b) -> a -> b
app f x = f x
```

Here we can see that the first argument to the function `app` is another
function which is represented by the `()` which shows the first argument is
expected to be a function that takes `a` and returns `b`.

```haskell
add1 :: Int -> Int
add1 x = x+1
app add1 1
  => 2
```

### Anonymous Functions

```haskell
(\<args> -> <expr>)
```

Anonymous functions do not have a name when they are defined. This is nothing
new, just highlighting that the meaning is the same as it other languages.

They consist of a backslash `\`, a list of arguments `<args>` and and expression
`<expr>`.

A simple example of our `add1` function from earlier:

```haskell
(\x -> x+1)
```

Because functions are just values in Haskell, we can assign this anonymous
function to a variable like this:

```haskell
add1 = (\x -> x+1)
```

An anonymous function with multiple arguments would look like this:

```haskell
(\x y z -> x+y+z)
```

The application of anonymous functions is shown below:

```haskell
(\x -> x+1) 1
  => 2

(\x y z => x+y+z) 1 2 3
  => 6
```

So instead of something like `add1 1` or `sum3Nums`, we are instead replacing
the function name with the anonymous function itself.

### High Order + Anonymous Functions

```haskell
app :: (a -> b) -> a -> b
app f x = f x

app (\x -> x+1) 1
  => 2
```

Here we've just replaced the `add1` function that we make earlier, with an
anonymous function (see the start of the "Higher Order Functions" section).

### Map

Map is important to understand because it is frequently used in Haskell and is a
good example of a HOF.

```haskell
map :: (a -> b) -> [a] -> [b]

[a, b, ..., y,  z]
 |  |       |   |
[1, 2, ... n-1, n]
```

It maps a list of type `a` to a list of type `b`. It takes a function as it's
first argument `(a -> b)` meaning it is a HOF. The function passed as the first
argument is used to convert the list from type `a` to `b` and it's good to note
that this type can change.

Here is a simple example:

```haskell
map :: (a -> b) -> [a] -> [b]
map (\x -> x+1) [1,2,3,4,5]
  => [2,3,4,5,6]
```

Here is another example which is another way of handling the list comprehension
approach we used previously (see around line 680 where we covered tuples) for a
similar problem.

```haskell
map :: (a -> b) -> [a] -> [b]
map (\(x,y) -> x+y) [(1,2),(2,3),(3,4)]
  => [3,5,7]
```

The list of tuples are converted to a list of the sums of each tuple.

To me these look very similar to closures.

### Filter

```haskell
filter :: (a -> Bool) -> [a] -> [a]

[1,2,3,4,5,6,7,8,9]
     | | | | | | |
    [3,4,5,6,7,8,9]
```

`filter` is used to filter a list of type `a` to a list of type `a`. It also
takes a function as it's first argument and it is used to convert the list to a
new list with values filtered out. Note that with `filter` the type of the list
can not change. The function passed as an argument in this case is called a
predicate. When the predicate returns `True` the element will be in the new
list.

```haskell
filter :: (a -> Bool) -> [a] -> [a]
filter (\x -> x > 2) [1,2,3,4,5,6,7,8,9]
  => [3,4,5,6,7,8,9]
```

Here we filter out any numbers that are not `> 2`.

```haskell
filter :: (a -> Bool) -> [a] -> [a]
filter (\(x,y) -> x /= y) [(1,2),(2,2)]
  => [(1,2)]
```

This example can we used to eliminate loops in the "directed graphs" example
covered in video #4, according to the video. In this case, we are using it to
remove tuples where the value of each element in a pair are the same.

### Video #7 - Partial Function Application & Currying

#### Currying

Partial function application is a result of "currying".

Currying is the principle which tells us, for example in the following:

```haskell
f :: a -> b -> c -> d
```

The function takes 3 arguments and returns one value, we could rewrite it as:

```haskell
f :: a -> (b -> (c -> d))
```

Which tells us that instead, the function only takes one argument `a` and
returns a new function `b` which takes only one argument `c` which then returns
the final value `d`.

Functions that have more than 1 argument don't actually exist. Functions only
take 1 argument and then return another function or the end result. This is
broken down more clearly in the following example which shows how we can use
currying to rewrite functions:

```haskell
add :: Int -> Int -> Int
add x y = x+y


add :: Int -> Int -> Int
add x = (\y -> x+y)

add :: Int -> Int -> Int
add = (\x -> (\y -> x+y))
```

All of the 3 function definitions are equivalent.

#### Partial Function Application

In the 3rd definition above, we can guess that based on how languages typically
work, the function needs 2 numbers to perform an addition, and so, we expect an
error of some kind because we are only able to pass 1 `x`.

```haskell
add :: Int -> Int -> Int
add = (\x -> (\y -> x+y))

add 1 :: Int -> Int
 => add (\y -> 1+y)
```

In Haskell it doesn't work this way because `add` implicitly only takes 1
argument and instead a new function is returned. Where `x` was previously a free
variable, it is now fixed as `1`.

This displays how we can change the behaviour of functions and generate new
functions from old ones. This is called "partial function application" and it is
very common in functional programming.

`map` is a prime example for a function which can be used in such a way.

If we have a function called `doubleList` that we expect to double all the
elements in a new list. We can use partial function application on map such that
we only provide the first argument (the only real one), which is used to create
the new list.

```haskell
map :: (a -> b) -> [a] -> [b]

doubleList = map (\x -> 2*x)

doubleList [1,2,3]
  => [2,4,6]
```

`doubleList`'s type definition ends up looking like this:

```haskell
doubleList :: [a] -> [b]
```

`map (\x -> 2*x)` is a function that takes a list and returns another and it can
be saved as a new function in `doubleList`.

Using `map` normally would require passing both a function `(a -> b)` as well as
the list `[a]` like this:

```haskell
map (\x -> 2*x) [1,2,3]
  => [2,4,6]
```

We don't need an argument for `doubleList` because it gets it *implicitly*.
