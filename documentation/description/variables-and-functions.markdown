
#### Variables

We ended the previous chapter with this example demonstrating nested expressions:

    Stdout: puts (“hoge, fuga, piyo”: replace “, ” with “ ”) !

Relying on nested expressions, in more complex programs, pretty quickly turns into a lousy, unreadable mess. To avoid doing that, we can assign nested values to variables using the `=>` operator. So we can re-write the above example:

    de-commafied-list => “hoge, fuga, piyo”: replace “, ” with “ ”.
    Stdout: puts de-commafied-list !

If you’re used to C-like languages, you should notice three important things:

1. Variable names can contain hyphens. In fact, they can contain just about any character except whitespace, brackets, or other characters that could be mis-interpreted for other syntactic features.
2. Declarations end in a period. All “code body” in Sandpiper (like this program scope) consists of declarations, and each declaration ends in a period. **But** not all declarations need to end in a period. In this case, the `puts/!` call is relieved of its period because the last particle of the method incantation (`!`) ends in an exclamation mark. You *could* still write it `Stdout: puts de-commafied-list !.` but that would be silly.
3. Variables are assigned exactly once—they are immutable. You can’t change them, and it doesn’t cause any problems for well-structured code. The first time I encountered a language with immutable variables, my hackles went up, and I said “How can you possibly represent a state changing over time with immutable variables!?” Actually, I didn’t say it so articulately, but Sandpiper has a stellar solution in the form of Pipes, which we’ll discuss later.

Variables in Sandpiper are strongly typed; but since the types are handled automatically in this case, we’ll ignore this detail until later chapters.

We use the `=>` operator to assign variables because (a) it contains an equals sign that’s traditionally used for assignment, and (b) it looks like a forward-pointing arrow. In traditional assignment, values are taken from the right side, and slid into the variable on the left as though it were a slot. Some languages even use a left-pointing arrow to represent this. In Sandpiper, it’s the opposite: A variable is just a name that refers to a value; in fact, you could say that the variable simply points to the evaluated result of the expression on the right-hand side of the `=>`. **In short, it’s a left-to-right *association,* not a right-to-left *storage*.**



#### Function Literals

Functions are declarations inside [square brackets]. Let’s define one.

    [ 2 ]

That’s a function that always returns the number 2. Functions are objects, so we can refer to them by variable:

    two => [ 2 ].

OK, so that’s a boring function. We can give a function parameters

    lower-back/ => [ name -> name: downcase: reversed ].
    : lower-back “Delano”.  ;; Calls the function and evaluates to “onaled”.
                            ;; (Text after a semicolon forms a comment, by the way.)

In this case, `name` is a parameter, and the `->` symbol separates its declaration from the function body. Some cool things are going on here! First, we’re calling a local function by using the same method call operator we’ve been using, `:`, with no receiver on the left. By putting a slash in the variable name `lower-back/`, we tell the compiler where the argument goes, which allows us to call the function in the same way we’d call a method. There are other ways to call functions, which we’ll see later.

And the second cool thing is the function body (e.g. `2` or `name: downcase: reversed`) forms the return value of the function, without silly `return` keywords! So what happens if we put multiple declarations in a function?

    lower-back/ => [ name ->
        lower-name => name: downcase.
        lower-name: reversed.
    ].
    : lower-back “Delano”.

This function does *exactly* the same thing. **In fact, it’s the *last* declaration in a function body that forms the return value.** You may notice that we omitted the period from those first single-declaration functions; the last declaration in a scope can have its period left off.

A function can take more than one parameter; each parameter is given before the `->`, with spaces separating them.

    subtract/from/ => [ subtrahend minuend ->
        minuend: - subtrahend
    ].
    : subtract 17 from 2.  ;; evaluates to -15.

A function can also refer to variables in its enclosing scopes, in effect becoming a closure:

    two => 2.
    subtract/from-two => [ subtrahend ->  two: - subtrahend ].
    : subtract 17 from-two.  ;; evaluates to -15.

One would expect as much from a language.


#### Functions in Boolean Logic

Before going further, we need to step over to something a little more rudimentary: Boolean logic. Say we have a couple of variables,

    hoge => (something).
    fuga => (something else).

We might naturally want to see if they’re equal. Simple enough:

    hoge: == fuga

literally just calling the `==` method on *hoge* with *fuga* as the argument. This returns a boolean, either the singleton `True` or the singleton `False`. These are just objects; in fact, members of `TrueClass` and `FalseClass`, respectively. And these offer a number of logic-oriented methods, such as `if-true/if-false/`. This method takes two functions, and finds the value of one of them (ignoring the other), depending on the receiver. The return value of the `if-true/if-false/` method itself is that value of the chosen function.

    hoge: == fuga: if-true [ hoge: * fuga ]  if-false [ hoge: - fuga ]

Here we have two method calls chained together: `==/` and `if-true/if-false/`. The equality test is done first, and so the `if-true/if-false/` method is called on either `True` or `False`. So, if *hoge* and *fuga* are equal, the whole expression is equal to the product of the two; if not, it’s the difference of the two.

Conceptually, the `if-true/if-false/` method is implemented by default to evaluate the first (true) function and return its value, and is overridden in `False` and `Nil` to evaluate and return the second (false) function. (The compiler reserves the right to cheat a bit about this in practice, for the sake of speed.) This is essentially how it works in Smalltalk as well. It also means that all objects except for `Nil` and `False`, such as `0`, `“”`, and `[ False ]`, are all logically truthy.


#### Terse Function Parameters

It’s pretty common to see a small, inline function like

    [ doge -> doge: is-happy? ]

Notice how the parameter name `doge` gets repeated immediately? As a helpful shortcut, you can use the percent symbol `%` to refer to parameters, as in `%0`, `%1`, `%2`, etc., as many as you need.

    [ %0: is-happy? ]  ;; equivalent

For further simplicity when you have only one parameter, just `%` is interchangeable for `%0`.

    [ %: is-happy? ]  ;; also equivalent

Terse parameters are only available for the immediate function where they appear. For instance, this is invalid:

    [
        %: is-happy?:  if-true [ %: name: append “ is wagging” ]
                      if-false [ %: name: append “ is sullen” ].
    ]

The `%` at the beginning of the outer function is valid, assuming the caller passes in a doge. But the `%`s *inside* the if/else clauses refer to arguments passed into those inner functions themselves, by the `if-true/if-false/` method, not the doge from the outer function—and since the `if` doesn’t pass in any arguments, the inner `%`s refer to nothing.



#### A Note on Typography

If you’re using a contemporary browser, you may be wondering how to type the `->` and `=>` characters in your code. These are, in fact, just a hyphen or equals sign followed by a greater-than. On this site, we use the [Hasklig](https://github.com/i-tu/Hasklig) typeface for code, which joins these character pairs into single, double-width glyphs. It’s rather nifty, and makes Sandpiper code more beautiful to read. You should try it out for your own editor!
