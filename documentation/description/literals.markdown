#### Array and Dictionary Literals

Array and Dictionary Literals are enclosed in `{`curly brackets`}`.

In Arrays, elements are separated by whitespace: `{ “fish” “dog” “fankhauser” }`. Dictionaries consist of key-value pairs, separated also by whitespace. A rocket, `->` or `=>`, pairs the key to its value. The pipe rocket `->` associates by value:

    { “food” -> “fish”  “pet” -> “dog”  “hooman” -> “fankhauser” }

or

    fid => “food”.  pid => “pet”.  hid => “hooman”.
    { fid -> “fish”  pid -> “dog”  hid -> “fankhauser” }

The terms `fid`, `pid`, and `hid` are evaluated; their string values `“food”`, `“pet”`, and `“hooman”` are used as keys in the dictionary. It's quite common to use symbols instead of strings in dictionaries, though:

    { :food -> “fish”  :pet -> “dog”  :hooman -> “fankhauser” }

If you're doing this, you can use the hash-rocket instead. See if you can spot what's going on here:

    { food => “fish”  pet => “dog”  hooman => “fankhauser” }

Where the pipe-rocket `->` associates by value—it *evaluates* what's on its left-hand side—the hash-rocket `=>` associates by symbol. In a dictionary, it takes the *bare word* to its left and interprets it as a symbol. This matches the behavior of the hash-rocket for variable declarations inside a function: It takes the bare word to the left as a variable name for the association.
