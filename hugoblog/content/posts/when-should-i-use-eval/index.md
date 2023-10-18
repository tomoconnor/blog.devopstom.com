---
title: "When should I use eval()?"
date: "2013-10-07T19:59:58.000Z"
description: "Never."

---

NEVER.
======
 
That's got that off my chest.

`eval()`

is possibly the most dangerous thing ever.  It's basically a way to execute arbitrary code from a string or variable.  

Here's a few reasons why it's dangerous.

It leaves you open to injection attacks. 

In Javascript, `eval()` forces the engine to drop into Interpreter mode, which slows down your application, and it will remain slow, as there's no opportunity for optimisation-level caching to take place.

It's a bugger to debug, because there's no line numbers.

In Javascript (client-side), `eval() `is dangerous because it exposes you to cross site scripting attacks.  

In server-side code, `eval()` is downright lethal, because it exposes the entire server to anything that the user wants to run. 

Python has a "safer" eval, called `literal_eval` in the `ast` module, which allows for parsing of user-provided data, without having to write a parser to sanitise it yourself.  I'd still avoid it like the plague, given a choice.
 

This is all fairly fresh in my mind, because I discovered a snippet of code somewhere (not disclosing where, as I'm doing the responsible thing and doing the disclosure properly), that was along the lines of 

`var jsonData = eval ("(" + string + ")");`

Apparently `JSON.parse()` isn't good enough for them. 

Horrifying.