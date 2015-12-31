#### Direct Recursive Calls and Early Returns

Within a bound function (a method), the variable `self` always refers to the object the function is bound to. Some languages use `this` for that purpose. Sandpiper also has a keyword `this`, but it refers to the function in which it appears. The most common usage is to perform recursive calls:

    ;; calls itself with ever-larger arguments
    infinity => [ n -> (n inc) ::this ]

We can make a standard recursive Fibonacci generator without `this`

    fibonacci => [ n ->
        n: <= 1:
            if-true  [ n ]
            if-false [
                ((n - 1) ::fibonacci) + ((n - 2) ::fibonacci) ] ].

but the function has to be named in order for it to work. We'd like to use `this` instead, to make it more portable. A good attempt might be

    fibonacci => [ n ->
        n: <= 1:
            if-true  [ n ]
            if-false [
                ((n - 1) ::this) + ((n - 2) ::this) ] ].

but there's a big problem: `this` refers to the body of the `if-false` clause, which is not what we want to recurse into. (And, since that function doesn't take any arguments, and we're trying to pass a number in, we'll immediately get an error.) Let's start building this again, bit by bit.

A Fibonacci number is the sum of the two prior Fibonacci numbers in the sequence. This is easy to represent:

    fibonacci => [ n ->
        ((n - 1) ::this) + ((n - 2) ::this) ].

We're calling the right function with `this`, because its scope is still the outer function. We'll leave this as the usual case, the "happy path," but add an additional early return if `n` is at a base case:

    fibonacci => [ n ->
        ^^ n: <= 1:
            if-true [ ^n ].
        
        ((n - 1) ::this) + ((n - 2) ::this).

    ].

There's some new syntax here, the caret and double caret. We call them the "Return-Throw" `^` and "Catch-Bounce-Return" `^^` operators. The return-throw takes the expression that comes after it (in this case `n`) and immediately throws it up the call stack. It's similar to throwing exceptions in other languages, but it takes any object, and is meant primarily for early returns, instead of error conditions. There is no "try-catch block" mechanism for handling these throws in Sandpiper; instead, we use the catch-bounce-return operator `^^`. It prefixes a declaration, and catches any early return that gets thrown from inside anywhere in that declaration. Instead of catching it for the surrounding code to use, however, it returns the thrown value one last time out of the function it immediately appears in (out of `this` where it appears).

Our function does what it's supposed to at this point, but it's inefficient. As a final touch, we should ask the compiler to memoize this function for us, so that the Fibonacci numbers at certain indices `n` are remembered across branches of recursion. We'll do that by putting the letter 'm' right after the closing bracket. We also want to make sure negative numbers are not given by making `n` an Unsigned integer:

    fibonacci => [ n (Unsigned) ->
        ^^ n: <= 1:
            if-true [ ^n ].
        
        ((n - 1) ::this) + ((n - 2) ::this).

    ]m.

There must be no space between the bracket and the 'm.'



#### A brief note on boolean logic

The primary mechanism for logical branching is the `if-true()if-false()` method that's available on every object. It takes two functions as arguments, and is implemented by default to return the value of the first function, the `if-true` function. The method is overridden by `Nil` and `False` to return the value of the second function instead. Thus, all objects except `Nil` and `False` are logically truthy.

There are a number of similar methods like the solo `if-true()` we used above.
