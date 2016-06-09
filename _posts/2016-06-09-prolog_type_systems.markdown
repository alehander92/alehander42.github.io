---
layout: post
title:  "writing custom type systems for python in prolog"
date:   2016-06-09
categories: python
---


## tl;dr

I play with Prolog, I find out I can infer custom type systems in under 50 lines of it and after that I create [hatlog](https://github.com/alehander42/hatlog) to feed it with some Python. In the end I find out my programs ~= 2 parallel programs: type inference / compile-time prolog and the normal program.

## prologue (heh)

I was playing with Prolog last weekend and I was wondering how can I apply it
for something different than the usual uni-solve-8-queens type of thing.
Prolog is often accepted as "that bizarre magic logical programming language from uni", so I wanted to actually build something real-world-y in it.

Most Prolog implementations have a repl, so it's pretty easy to just start modeling relationships between .. things. I am very interested in prog languages, so sooner or later I started trying to create rules for a simple language. I started describing the signatures of several "builtin" functions and it was pretty and neat:

```prolog
push(list, any, list).
```

It didn't take long to realize I can do

```prolog
?- push(list, any, X).
X = list.

?- push(Receiver, Element, list).
Receiver = list,
Element = any.
```

The builtin Prolog unification gave me type inference for free.
This looks even cooler when you realize you can do stuff like

```prolog
sub(int, int).
?- push(X, any, X), sub(X, int).
false
```

You can apply this to a whole "program" and basically type check it and infer all the types.

However list and any don't help much! So I decided to support generic types in my signatures. Accidentally Prolog's free variables seemed like a good solution:

```prolog
push([list, T], T, [list, T]).
?- push(X, z, X).
X = [list, z].
?- push(X, z, [list, v]).
false.
```

## my prolog milk brings all the pythons to the backyard

I had a very simple type inference engine but it worked only on some imaginary 
facts. Those facts did look suspiciously similar to real programs though. Actually if you just write a simple fibonacci in Python, you can easily map it to rules like these: `if` is `if(Test, IfBlock, ElseBlock, IfType)`, `==` is `eq(Left, Right, EqType)` and `+` is `bin_op(Left, Right, BiType)`. I mapped several simple functions to prolog rules like that and I realized the type inference was working ok for that too. 

So then `hatlog` was born. 

`hatlog` takes a one-function python program:
    
  * visits and flattens the ast in DFS order
  * annotates each node with its type: an atom(e.g. `str`) for literals and a free variable(e.g. `Z0`) for the other unknown types.
  * maps a prolog fact to each node with args its subnode types
  * generates and runs a prolog program with the rules, which shows the resulting type:

```bash
bin/hatlog examples/map.py
# A,B::Callable[[A],B] -> List[A] -> List[B]
```

The cool part is that all the Python-related type rules are isolated in `pythonTypeSystem.pl` and the generated programs just use that as a library.

That way you can easily write your own `custom.pl` and use your own swappable custom type system: you can have a simple declarative representation of it just like grammars can represent parsers!

## the type system

If you look at `pythonTypeSystem.pl` you will see ~50 lines of code, but most of the magic is happening from line 29 to 57: the lines before that define some common interfaces and the lines after 57 just contain signatures for several example standard library methods.


However just 1-2 simple relations are enough to describe the rules for most nodes:

```prolog
z_index([list, X], int, X).
z_index([dict, X, Y], X, Y).
z_index(str, int, str).
```

```prolog
z_for(Element, Sequence, _, void) :- sequence(Sequence), element_of(Element, Sequence).
```

the generated program just uses this rules with some free variables for unknown types:

```prolog
f(map, [Z0, Z1], X) :-
    z_list(Z3, Z4),
    z_assign(Z2, Z4, Z5),
    z_fcall(Z0, [Z6], Z7),
    z_method_call(Z2, append, [Z7], Z8),
    z_for(Z6, Z1, [Z8], Z9),
    =(Z2, X).
```

Here prolog originally unifies `Z0` with `[function, _G2, _G4]`, `Z1` with `[list, _G2]` and `X` with `[list, _G4]`. `_G..` are free variables and most of the time we try to bind constants to them. However in this case they can be used as solutins: they correspond to generic types, and you can see they repeat twice: we have unification of generic types.

We do some post-processing work behind the scenes with Prolog and reformat the result to 

```
A,B::Callable[[A],B] -> List[A] -> List[B]
```

I've tried to match the types with mypy's rules.

## why

In the end, I'll try to summarize my feelings about that. 

Everybody who does PL theory more seriously probably knows that, but I didn't realize how close type inference and prolog-like systems look. Of course this project is an oversimplification, I support only a subset of Python for a proof of concept. 

However now I can imagine a statically typed program like 2 parallel programs:

  * the type inference which works on type level. it matches and unifies types similarly to how prolog finds solutions for constraints
  * the normal program which works on value level.

Is that close to truth? Is there a deeper link between logical programming and type theory? Any experiments like this? I'd love to hear more about this kind of stuff

In the end I enjoyed that, Prolog is actually pretty ok even for writing "normal" code and this very high level type of rules are way better experience than e.g. implementing type inference for [Pseudo-Python](https://github.com/alehander/pseudo-python) in Python(for prototyping)
