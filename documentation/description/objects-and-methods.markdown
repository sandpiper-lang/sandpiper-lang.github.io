
<article>
#### Objects

Everything you can refer to in Sandpiper is an object. Objects in Sandpiper are very similar to objects in other languages—they have state, they have methods, and they participate in a class structrue—except that they are almost always immutable.

</article>

<article>
#### Simple Method Calls

Most of the work in Sandpiper is done by calling methods on objects, and usually those methods simply return values and don’t cause side-effects. Let’s take a look at something like that:

    “foo”: reversed

This will return `“oof”`, the result of calling (`:`) the method `reversed` on the object `“foo”`. We’d call `“foo”` the *receiver* in this case, because it receives the “reversed” message.

Method calls can, of course, take arguments:

    “hoge, fuga, piyo”: split “, ”

This returns a 3-item list by separating out the parts of the receiving string `“hoge, fuga, piyo”`. The method is `split`, and the argument is `“, ”`. Methods that take multiple arguments divide the method name into pieces, and insert the arguments in between:

    “hoge, fuga, piyo”: replace “, ” with “ ”

Again, we’re calling (using the `:` operator) the method `replace with` on the object `“hoge, fuga, piyo”`, with the arguments `“, ”` and `“ ”`. This syntax is similar to Smalltalk or Objective-C, except that it changes the role of the colon.

As a point of terminology, where the object to the left of the colon is the “receiver,” the terms `replace “, ” with “ ”` after the colon we refer to as the “method incantation.” This is separate from the “method name,” which is `replace/with/`, each `/` indicating the place for an argument in the incantation.

</article>

<article>
#### Side Effects

So far, we haven’t made a program that actually *does* anything. Let’s—um—*do* that right now.

There’s an object that’s always available called `Stdout`, short for “standard out,” and it’s how we put text out to a terminal. It has a method called `puts/!` which takes a piece of text for its sole argument, prints it out, and follows it with a newline character.

    Stdout: puts “Hello, world!” !

The method ends in a bang (`!`) to indicate it performs side-effects, in addition to just returning a value. (This is required by Sandpiper; otherwise, it might never get run!)

In fact, this code is a complete program which you can run. (well, erm, when the compiler’s available.)

You can nest expressions, just as you’d expect:

    Stdout: puts (“hoge, fuga, piyo”: replace “, ” with “ ”) !

which outputs “hoge fuga piyo”. Natch!

</article>


<article>
#### Method Chaining

You can fairly intuitively chain method calls together. Instead of doing `replace/with/`, we could do

    “hoge, fuga, piyo”: split “, ”: join “ ”

This splits the “hoge, fuga, piyo” string, which returns a list of the components; on that list we then call `join` which puts the elements back together into one string, with a single space separating them.

</article>
