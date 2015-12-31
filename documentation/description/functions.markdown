#### Unbound Functions

Let's look closer at one of those last examples from the section on type inheritance: `doge: filter [ &: ends-with “er” ? ]`. You'll recall that `doge` is defined as

    doge => LinkedList: with-element “wagger” next Nil:
        push “rump”:
        push “belly”:
        push “lungs”:
        push “shoulders”:
        push “neck”:
        push “head”:
        push “sniffer”.

If you've dealt with similar constructs in other languages (in Ruby, it's *very* similar: `doge.select {|part| part.end_with? "er" }`) you should suspect that the `[...]` expression is a function of some sort.

In Sandpiper, square brackets denote a function literal; at runtime, when it comes into scope, it forms a first-class function—which, of course, is also a first-class object. We've seen functions before—types take functions in their definitions, and refer to them by name in their own method namespaces. But unbound functions like that in the `filter` example are common and quite flexible.

A function normally takes an argument or more, and returns one value. Here's a function that takes a number:

    ;; adds one, returns a string representation
    [ omnom -> omnom: inc: to-s ]

The terms before the pipe rocket `->` operator are the parameters to the function. Multiple arguments are separated by spaces (commas are never used to separate items in Sandpiper):

    ;; true if the index is less than the item's i-value
    [ item  index ->
        index < (item at :i-val) ]

Functions without parameters omit the pipe rocket:

    [ Random: from (1 .. 10) ]

For brevity, functions automatically have arguments named with ampersands, starting from `&0`, through `&1`, `&2`, `&3`, etc. If your function uses these, it needn't otherwise name its parameters. Additionally, a lone ampersand `&` is equivalent to `&0`. The following functions are all identical:

    [ omnom -> omnom: inc: to-s ]
    [ &0: inc: to-s ]
    [ &: inc: to-s ]

The part of the function after the rocket is the "function body," which consists of one or more declarations. The final declaration's value is the function's return value. So far, the examples in this section have all been single-declaration functions. As before, we need to end declarations with a period if we have more than one in a scope.

    [ upper-limit ->
        real-limit => upper-limit: sqrt.
        Random: from (0 .. real-limit).  ]

In certain circumstances, you may want to make absolutely sure your little function is accepting the type of arguments you expect. For instance, we want to make sure in this function that we're getting a number before we try to take its square root:

    [ upper-limit (Number) ->
        real-limit => upper-limit: sqrt.
        Random: from (0 .. real-limit).  ]

Multiple-argument functions can also have type specs:

    [ value (Int)  period (Int)  -> value: mod period ]

It's entirely up to you when you specify the argument types in a small function. In small scripts especially, you needn't burden yourself with specifying them. Or, if a lot of your code is strongly typed already, the types may be perfectly inferred already, and specifying them again would be redundant.

Being objects, you can assign functions to variables.

    humaniser => [ &: inc: to-s ]

A function captures the state of variables surrounding it in its lexical scope, at the time it's created; thus, multiple copies of the "same function" may actually be distinct in state, if surrounding variables they use are different.

There are two different ways to call an unbound function, depending on how you want to use it and how it's named. Let's look at the two faces of `max`: It's a standard function that's always available, but it has two different names. One name is `max-of(){and()}`, and the other is just `max`. The first form uses curly brackets to indicate that the `and()` clause of the name may repeat (this is available for method names as well). Since it has particles and argument placeholders, we can call it just like any other method, using the single colon.

    : max-of 5 and 3   ;; 5
    : max-of 5 and 3 and 7 and -10   ;; 7

The official name of the colon in this usage is the "Function Incantation Call Operator." When there is only an empty expression to its left, it assumes the incantation names an unbound function, instead of a method. (Remember that the operator requires some amount of whitespace to its right—`:max-of` is interpreted as a symbol!)

To call a function this way, the variable name that refers to it must be formatted in an incantation-compatible way (with parentheses for arguments) that matches the parameters of the function. Sometimes this isn't the most convenient, or you need to call a function that's named too generically. In these cases we can use the "Direct Function Call Operator," or `::`.

The `max` variation is designed to be called this way.

    5  3  ::max
    5  3  7  -10  ::max

Its arguments are space-separated, and come to the left. This is unusual for most languages. One of Sandpiper's guiding philosophies is that data should flow from left to right, through the machinery of your code. As such, the arguments flow from the left, through the `max` function to the right.

There is actually a third way to call a function, though it is less of an intrinsic language feature. Functions are objects, and they have methods. One of those methods is `apply()`, that takes a List of arguments. This is how we'd use it with the `max` function:

    max: apply { 5  3  7  -10 }

Or, if we have an arbitrary list called `args`,

    [ args -> max: apply args ]

Functions that have side effects should be called using the sister method `apply()!`:

    [ args -> max: apply args ! ]
