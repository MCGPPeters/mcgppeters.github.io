# The Either type in C\#

As of C# 7 we gained a few more new functional language constructs. Among them (value) tuples, patterns matching and deconstruction. Now, I'm not going into functional programming theory for multiple reasons. For one, I think I'm not qualified :). Second, it is not the purpose of this (series of) post(s).

It will however show how certain functional constructs combined with a specific data type (Either) can be used to approach and simplify very common, even repetitive, programming tasks. Programming tasks that are related to something we all complete on a daily basis, solving everyday business problems. Things like validation, exception handling and offering alternatives for absent values (not returning NULL). Nothing esoteric, math related and it will be in C#.

As I mentioned, in C# we have tuple types. And these are very useful. One of the useful purposes in C# is that they can be used to return multiple values thus from a function (method etc...), without having to declare a type (probably only used once). This is one of the 'selling' points of is being highlighted in C# 7.

Now if you think about it, a tuple basically expresses a type that for each instance it MUST contain a value for all its members. for instance, given the following tuple type:

```C#
(int, string, double)
```

Each instance MUST contain an integer AND a string AND a double. With an emphasis on the word AND.

So imagine a type that expresses something similar, but having an emphasis on the word OR. How would something like that look like? Maybe something like:

```C#
Either<int, string>
```

For each instance of this type, it's value MUST *either* be an integer OR a string.

Now, why would that be useful. And why might this be at least as valuable a type as a tuple? Well think about it. How many times would you like to express explicitly that a function might:

* return a value OR raise an exception?
* return a (list of) value(s) OR nothing (or worse, the wicked NULL value) (like a query)?
* return a result OR an error (like a validation error or declined authorization)?
* ... ?

Lots of times, huh? This is where Either comes in handy. In functional jargon, Either could be seen as a so called 'discriminating union', which is a so called 'algebraic datatype', as is a tuple.

C# doesn't have discriminating unions (yet). Discriminating unions in functional languages can express more cases then 2. But while you can do this as well in C# by expressing multiple variants of Either (e.g. ```Either<T, T1, T3, ...>```, like Func delegates) this isn't really needed. First, two cases for the purposes I describe in this post are enough. Second, it actually is a new type and third, when really needed, ```Either< T, TOtherwise>``` can be nested (e.g. ```Either<int, Either<int, int>>``` etc), just as tuples can (```(int, (int, int))``` etc).

Consistently using this type, I might argue, significantly reduces the number of common bugs. Not the least of which is... the dreaded, and most of the times unexpected and very hard / expensive to debug, ```NullReferenceException```.

In the follow up post I will show how using this type can be simplified and can help you the be very clear in expressing your intent ...

