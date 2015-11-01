---
layout: post
title: "visual representation of code layers"
date: "2015-11-01 03:18:22 +0200"
---

source code is just textual these days

there are some exceptions, like visual browsers or graph-like dataflow languages, but most source code is just text. which is really ok, because text is universal and easy to manipulate.

however we highlight it in different colors, use bold, why don't we try something more radical with our editors?

* use different font sizes.
  i am looking at a piece of code. I want to see it "zoomed in" but I still want the perspective. Show me the current focused method with bigger font size and the others with smaller.

* represent types and exceptions with background colors
  we use front colors so much, but why can't we use the background? instead of showing boring try/catch boilerplate, we can fill backgrounds of code depending on the possible exceptions it can throw

those are random ideas, but I may try to implement prototypes of them as editor plugins, atom seems good for that(it's easy to style with css)
