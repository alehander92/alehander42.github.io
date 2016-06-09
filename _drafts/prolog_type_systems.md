---
layout: post
title:  "writing custom type systems for python in prolog"
date:   2016-06-09
categories: python
---

# writing custom type systems for python in prolog

## tl;dr

I play with Prolog, I find out I can easily model a simple type system in it and after that I feed it with some Python.

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

