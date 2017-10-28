---
layout: post
title: "example tests vs generated tests"
date: "2017-10-28 08:10:02 +0200"
---

example:

```nim
check("xz".reverse() == "zx")
```

generated:

```nim
quicktest "reverse" do(s: string(symbols={'a'..'z'}, min=0, max=20)):
  check(s.reverse().reverse() == s)
```

example tests are good, because they're deterministic, they hardcode edge cases
and bugs and they are simpler to write.

however, they're .. booring to write. honestly most people don't go and think hard for 
every possible edge case and write well distributed statistically 20 example cases, they just go
and test several obvious scenarios and then some known problem situations.

you can still have the best of both worlds: e.g. in python's hypothesis your failed generated examples are automatically added to a database. so you can reproduce testing for a known bug and still find new ones.

those reverse examples seem lame, but I've used this kind of testing for way more realistic problems involving starting other processes, intercommunicating services and a lot of weird stuff happening, not just 2-line methods. and it works surprisingly well if you can easily generate the initial conditions: you can autogenerate bug tasks when things crash and test for properties.

that's the most powerful thing about quickcheck-style libraries: they make you think about what `properties` you want to be true, which is more general and insightful.

of course they are just an additional tool. normal example cases (or hardcoded generated ones) must be your first line of defense.

if you're interested, check out the original [haskell quickcheck](https://hackage.haskell.org/package/QuickCheck) and [python's hypothesis](http://hypothesis.works/)

they show well two main approaches: generate most of the stuff based on types or based on custom generator definitions. that's something I am aiming to combine in [nim](https://github.com/alehander42/nim-quickcheck)

if you have better examples, please write me


