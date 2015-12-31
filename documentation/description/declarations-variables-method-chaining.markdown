#### Declarations, Variables, and Method Chaining

Variables in Sandpiper may be set exactly once, making them immutable. You use the hash-rocket `=>` to assign them a value.

    doge-body => my-doge: rest

Traditional variable assignment is like putting a value into a bucket. In sandpiper, it's a declaration—a declaration that some name (`doge-body`) is equivalent to the value of some expression (`my-doge: rest`).

To declare that variable without assigning anything to it yet, use the syntax

    there-is doge-body.

This is a kind of “keyword declaration,” which we’ll discuss later. Assignment-free variable declarations won’t be immediately useful, but they’re important to know about.

As an example, we can re-do our earlier string replace by splitting it into pieces, and joining them back together. One way to accomplish this is through a pair of declarations:

    pieces => “hoge, fuga, piyo”: split “, ”.
    replaced => pieces: join “ ”.

Each declaration ends in a period—this is necessary when you have more than one declaration in a row.

We can simplify this code by chaining the method calls together:

    replaced => “hoge, fuga, piyo”: split “, ”: join “ ”.



##### Implicit Method Calls

You can omit the colon after the receiver in a method call.

    “foo”: replace “o” with “!”.
    “foo” replace “o” with “!”.  ;; same

    5: * 3.
    5 * 3.  ;; same

Doing so comes with a number of intrinsic limitations—particularly, you can only omit the colon where the syntax doesn't become ambiguous; and call chaining is not possible. For instance, `5 * 3 - 10` won't give you 5. It'll look for the `*()-()` method, and not be able to find it. Any of the following *will* work, however:

    5: * 3: - 10.  ;; explicit
    5 * 3: - 10.   ;; the first method call in a chain can be implicit
    (5 * 3) - 10.  ;; otherwise disambiguated

It should be clear, at this point, why the simple arithmetic operators don't have precedence over one another. They're chained linearly, left-to-right, like any other methods.




#### A Brief Introduction to Types

So far, we’ve been writing ducktyped code, that looks and feels like ducktyping in Ruby, Python, Smalltalk, etc. In fact, all Sandpiper code is strongly typed, but it allows you to completely ignore type information in most circumstances, if it's not helpful for your code. We'll just briefly touch on Types here.

A type is a set of methods, and can be a class of objects. For instance, the literal `“doge”` is an object of type `Text`, which tells you how you can interact with it—what methods you can call on it.

In a variable declaration, the variable's “declaration point,” to this point, has just been the variable's name. It can also contain the name of the variable's type in parentheses after the name. For instance, you can re-write

    there-is doge.
    doge-body => doge: rest.

as

    there-is doge (List).
    doge-body (List) => doge: rest.

The value `Nil` cannot be the value of a variable with a specific, simple type like `List`. `Nil` has its own class, `NilClass`, which doesn't inherit from `Object`. To allow such a variable to refer to Nil as well, you can suffix it with a question mark, to make it Optional: `List?`. (While you can't use an `Object`-type variable to refer to Nil, you *can* use one typed `Any`.)

The compiler will infer the types of variables as much as possible. Specifying them explicitly tells Sandpiper to make sure that your variables hold the exact type you're expecting when the program runs. To the extent that it can do this at compile time, the compiler will make sure all your types line up, and warn you when they don't. When it can't do this, it will insert runtime checks. Specifying a variable's type isn't usually useful at inline declarations, but can be essential when declaring new classes.
