---
title: Transducers: Why and How?
---

I encountered the notion of transducers about a year ago, but didn't really look into it.
Recently, I encountered it again in [a presentation about functional programming in C++](http://vitiy.info/Slides/MeetingCPP2015/MeetingCPP2015Complexity.pdf) (slide 37).

[The article on the Clojure website](http://clojure.org/transducers) didn't really help, and a Google search for "Haskell transducers" gave me discussions such as
["Clojure's Transducers are perverse lenses"](https://www.reddit.com/r/haskell/comments/2cv6l4/clojures_transducers_are_perverse_lenses/) *(What?)* and [transducers are functions a -> [b] cleverly transformed with a bit of continuation passing so that the Kleisli composition operator becomes (.)](http://stackoverflow.com/questions/26653829/how-is-a-transducer-different-from-a-partially-applied-function) ***(What?!?)***.

[Franklin Chen's blog post](http://conscientiousprogrammer.com/blog/2014/08/07/understanding-cloure-transducers-through-types/) made things a little clearer, but the *why* was still escaping me.
And with [this StackOverflow post](http://stackoverflow.com/questions/26317325/can-someone-explain-clojure-transducers-to-me-in-simple-terms), I think I got the gist of it.

## Why?

* Transducers are useful to compose transformations together:

> Transducers are composable algorithmic transformations. 
> *([Source](http://clojure.org/transducers))*

* They generalize over the input and the output containers, so a little like `fmap`:

> They are independent from the context of their input and output sources and specify only the essence of the transformation in terms of an individual element. 
> Because transducers are decoupled from input or output sources, they can be used in many different processes - collections, streams, channels, observables, etc. 
> *([Source](http://clojure.org/transducers))*

> Note that there is nothing that ties transducers to any concrete "collection" type. 
> We could write instances of `Foldable` and `Conjable` for some kind of "channel" abstraction, for example, and instantaneously be able to munge data coming from it and to another. 
> *([Source](http://conscientiousprogrammer.com/blog/2014/08/07/understanding-cloure-transducers-through-types/))*

* This allows for more code reuse.

## How?

In Clojure, some functions produce a transducer based on how many arguments you provide the function.

In Haskell, we have to define some functions that give us (or even: lift a function to) a `Transducer` (examples taken from [here](http://conscientiousprogrammer.com/blog/2014/08/07/understanding-cloure-transducers-through-types/):

```Haskell
-- Left reduce
type Reducer a r = r -> a -> r

type Transducer a b = forall r . Reducer a r -> Reducer b r

-- Produces a Transducer that maps stuff from a to b.
-- mapping :: (a -> b) -> (r -> b -> r) -> r -> a -> r
mapping :: (a -> b) -> Transducer b a
mapping f xf r a = xf r (f a)

-- A really simple 'reducer' to append something to a list.
-- conj :: [a] -> a -> [a]
conj :: Reducer a [a]
conj xs x = xs ++ [x]

-- Takes a transducer and a list, then applies conj to the transducer and folds the entire thing to list as the output.
-- xlist :: Transducer a c -> [a] -> [c]
xlist :: (([a] -> a -> [a]) -> [b] -> c -> [b]) -> [a] -> [c]
xlist xf = foldl (xf conj) []
```

`xlist` is very interesting, because it's what can be abstracted over: it is the container in which our data lives.
We could have an `xvector` instead, for example.

This also explains why, in the `Transducer` type, `r` is universally quantified: the transducer functions like `mapping` *are not allowed to know what the container is* for this generalization to work.

This is exactly the point the Clojure documentation makes:

> They are independent from the context of their input and output sources and specify only the essence of the transformation in terms of an individual element. 

Franklin Chen's blog post has some code that shows which typeclasses are necessary to abstract over the container type.
For our purposes, we won't do this exercise, but continue with the list representation.

### An example

Let's take `xlist ( mapping (+1) ) [1..5]` as an example.

`xlist` becomes:

```Haskell
-- xlist xf = foldl (xf conj) []
-- xf = mapping (+1)
xlist = foldl ( (mapping (+1)) conj ) [] [1..5]
```

`mapping` then becomes:

```Haskell
-- mapping f xf r a = xf r (f a)
-- f = (+1)
-- xf = conj
-- r = we don't know yet.
-- a = we don't know yet.
mapping = \r a -> conj r ( (+1) a )
```

Working out the iterative step of foldl:

```Haskell
-- foldl f z (x:xs) = foldl f (f z x) xs
-- f = (\r a -> conj r ( (+1) a ))
-- z = []
-- (x:xs) = [1..5]

f z x = ((\r a -> conj r ( (+1) a )) [] 1) = conj [] ((+1) 1) = conj [] 2 = [2]
```

So this is where it happens: the `(+1)` function given to the `mapping` transducer ends up in here.
The `foldl` function will collect it all into a list, giving us `[2,3,4,5,6]`.

To reiterate, the generalized transducer functionality allows you to abstract over the container and just say "I want to map over it and then filter it and then use it in a formula".

That is, we can compose "recipes" out of functions that then work on every type of container for which the appropriate typeclasses are defined.
Let's see how such a recipe works behind the scenes.

### Another example

```Haskell
-- A filter lifted to a Transducer.
filtering :: (a -> Bool) -> (r -> a -> r) -> (r -> a -> r)
filtering p xf r a = if p a then xf r a else r

-- concatMap lifted to a Transducer.
flatmapping :: (a -> [b]) -> (r -> b -> r) -> (r -> a -> r)
flatmapping f xf r a = foldl xf r (f a)

-- Three composed transducers.
xform :: (r -> Integer -> r) -> (r -> Integer -> r)
xform = mapping (+ 1) . filtering even . flatmapping (\x -> [0 .. x])

xlist :: (([a] -> a -> [a]) -> [b] -> c -> [b]) -> [a] -> [c]
xlist xf = foldl (xf conj) []
```

Let's write out `xlist xform [1..5]`.

First, `xlist` :

```Haskell
-- xlist xf = foldl (xf conj) []
-- xf = mapping (+ 1) . filtering even . flatmapping (\x -> [0 .. x])
xlist = foldl (mapping (+ 1) . filtering even . flatmapping (\x -> [0 .. x]) conj) [] [1..5]
```

Then, `flatmapping (\x -> [0 .. x]) conj`:

```Haskell
-- flatmapping f xf r a = foldl xf r (f a)
-- f = (\x -> [0 .. x])
-- xf = conj
-- r and a = don't know yet, they'll be given to us in the foldl call.
flatMapping = \r a -> foldl conj r ( (\x -> [0 .. x]) a ) = \r a -> foldl conj r [0..a]
```

Then, `filtering even (\r a -> foldl conj r [0..a])`:

```Haskell
-- filtering p xf r a = if p a then xf r a else r
-- p = even
-- xf = \r a -> foldl conj r [0..a]
-- r and a = don't know yet, they'll be given to us in the foldl call.
filtering = \r a -> if (even a) then (\r a -> foldl conj r [0..a]) r a else r
```

Then, `mapping (+1) (\r a -> if (even a) then (\r a -> foldl conj r [0..a]) r a else r)`:

```Haskell
-- mapping f xf r a = xf r (f a)
-- f = (+1)
-- xf = \r a -> if (even a) then (\r a -> foldl conj r [0..a]) r a else r
-- r and a = don't know yet, they'll be given to us in the foldl call.
mapping = \r a -> (\r a -> if (even a) then (\r a -> foldl conj r [0..a]) r a else r) r ((+1) a)
```

Finally, working out an iteration of the `foldl` step:

```Haskell
-- foldl f z (x:xs) = foldl f (f z x) xs
-- f = \r a -> (\r a -> if (even a) then (\r a -> foldl conj r [0..a]) r a else r) r ((+1) a)
-- z = []
-- (x:xs) = [1..5]

f z x = (\r a -> (\r a -> if (even a) then (\r a -> foldl conj r [0..a]) r a else r) r ((+1) a)) [] 1
 = (\r a -> if (even a) then (\r a -> foldl conj r [0..a]) r a else r) [] ((+1) 1)
 = if (even 2) then (\r a -> foldl conj r [0..a]) [] 2 else [])
 = if (even 2) then (foldl conj [] [0..2]) else []
 = foldl conj [] [0..2]
 = [0,1,2]
```

It is now obvious that each transducer results in a left-to-right embedding: 
while the definition of `xform` is `mapping (+ 1) . filtering even . flatmapping (\x -> [0 .. x])`, the functions are executed left-to-right:

* [1,2,3,4,5] becomes [2,3,4,5,6] *(`mapping` transducer)*
* [2,3,4,5,6] becomes [2,4,6] *(`filtering` transducer)*
* [2,4,6] becomes [0,1,2,0,1,2,3,4,0,1,2,3,4,5,6] *(`flatmapping` transducer)*

Hopefully this clarifies how transducers work!
