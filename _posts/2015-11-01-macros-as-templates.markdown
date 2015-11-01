---
layout: post
title:  "macros expressed as templates"
date:   2015-11-01
categories: ast macros
---


in lisps we have traditional ast macros: they take a tree corresponding
to the structure of some lisp code and expand it to another tree evaluating
a lisp function in compile time:

{% highlight clojure %}

(defmacro until [t & b]
  `((fn []
   (if-not ~t
     (do ~@b (recur))))))

{% endhighlight %}

<!--  `-->

however most languages are not homoiconic so if we have syntax macros,
they don't look so natural, cause unquotes should work on some kind of
"artificial" ast structures(code != data):

an example with an imaginary macro system in ruby

{% highlight ruby %}

defmacro until(t, *b)
  Quote(While(BooleanUnary(:!, t), b))
end

{% endhighlight %}

that look surprisingly ok, but still its foreign, you should know the
exact node names and the order of ast nodes, it's not natural

what if we could have something like a ruby-like template language expanding
on compile time, with additional syntax for unquoting and quoting syntax elements

{% highlight ruby %}
a until(t, *b)
  %quote(
    while !%u(t)
      %ul(b)
    end
  )
end
{% endhighlight %}

ruby already has %\<s\>() syntax for special literals, and it seems pretty clean
and feasible

i did something like that in one of my projects:
[sith](https://github.com/alehander42/sith) however it's way more limited
and unnatural, if i have time, i'd like to try that one here
