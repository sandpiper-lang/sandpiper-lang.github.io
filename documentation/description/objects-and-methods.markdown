#### Objects

Everything you can refer to in Sandpiper is an Object. Objects in Sandpiper are very similar to objects in other languages—they have state, they have methods, and they participate in a class structure—except that they are almost always immutable.

#### Simple Method Calls

Most of the work in Sandpiper is done by calling methods on objects, and usually those methods simply return values and don’t cause side-effects. Let’s take a look at something like that:

    “foo”: reversed

This will return `“oof”`, the result of calling (using `:`) the method `reversed` on the object `“foo”`. We’d call `“foo”` the *receiver* in this case, because it receives the “reversed” message.

Method calls can, of course, take arguments:

    “hoge, fuga, piyo”: split “, ”

This returns a 3-item list `{“hoge” “fuga” “piyo”}` by separating out the parts of the receiving string `“hoge, fuga, piyo”`. The method is `split`, and the argument is `“, ”`. Methods with multiple arguments divide the method name into pieces, and take the arguments in between:

    “hoge, fuga, piyo”: replace “, ” with “ ”

Again, we’re calling (using the `:` operator) the method `replace with` on the object `“hoge, fuga, piyo”`, with the arguments `“, ”` and `“ ”`. This syntax is similar to Smalltalk or Objective-C, except that it changes the role of the colon.

Where the object to the left of the colon is the “receiver,” the phrase `replace “, ” with “ ”` we call the “method incantation.” It consists of a series of alternating “particles” (`replace`, `with`) and “arguments” (`“, ”, “ ”`). If you take all the particles together with `()` as a placeholder for the actual arguments, you get the “method name,” as in `replace()with()`.

You'll commonly see method names expressed as Symbols (the same as in Ruby and other languages; we’ll discuss them later), which look like `:replace()with()`. Note that the colon as a method call operator requires a space (or any whitespace) after it, where the colon indicating a symbol requires that there *not* be any space after it.


#### Side Effects

"Method calls" and "function calls" don't actually direct the machine to execute a call. They describe how to calculate a value. Sandpiper may actually execute your method calls in any order it sees fit (or more than once, or not at all), so long as it maintains the logical consistency of what certain functions are expected to return.

Out here in the real world, though, we expect our programs to do something. Certain methods are marked as having a side-effect, which causes the calls to be compiled more like traditional procedural code.

So far, we haven’t made a program that actually *does* anything. We’ve just described how to evaluate certain values. Let’s put one of these values back out onto the Terminal. We’ll use the Stdout object, which is always available, and call its `puts()!` method:

    Stdout: puts “Hello, world!” !

The method ends in a bang (`!`) to indicate it performs side-effects, in addition to just returning a value. In fact, this code snippet is a complete program which you’ll be able to run when the compiler becomes available.

You can nest expressions, just as you’d expect:

    Stdout: puts (“hoge, fuga, piyo”: replace “,” with “.”) !

You're not likely to "forget" to put the bang after the methods that need it: The bang is literally part of the method name itself.
