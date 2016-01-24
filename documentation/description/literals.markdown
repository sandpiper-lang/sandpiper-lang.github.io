#### Text Literals

What other languages call a "String," we officially call Text. (For convenience, and to accommodate the force of lifelong habit, the type Text is also called String.) Text literals are, unsurprisingly, held between `"`double quotes`"`. Here we'll only discuss strings enclosed in typewriter quotes, because they're right there on your keyboard; however, Sandpiper supports and recommends using real `“`quotation marks.`”`

Sandpiper uses Smilef-style text literals. To interpolate one text in another, we embed the subtext in parentheses, within the outer string:

    [ mood -> "(mood) with the snakes and spiders" ]
        ;; -> "partying with the snakes and spiders"

The parenthetical expression can be any expression you'd use anywhere else in Sandpiper, and it gets evaluated at runtime.

    [ mood -> "(mood: capitalized) with the snakes and spiders" ]
        ;; -> "Partying with the snakes and spiders"

##### Why parentheses, and not square brackets?

Parentheses enclose expressions; square brackets enclose functions. Only expressions can be interpolated into strings.

##### Escaping

There is one guiding rule for escaping characters in things like Text literals:

1.  If an opening character would open an unwanted sub-scope, or
2.  if a closing character would prematurely close the scope,

make that character a literal of itself by using the open and close characters immediately adjacent. So—

If you want to include a literal open-parenthesis in the text, use `()` instead:

    "Your fingers will be warmer if you ski harder ()up to a point)."
        ;; will not try to evaluate `up: to a point`.

Along the same lines, if you want to use a quote mark in your text, simply use a pair of them adjacent.

    "Press the ""back"" button several times"

##### Formats and Special Characters

Sandpiper treats all code input as UTF-8, without exception. All unicode characters are allowed in text literals, so long as they aren't syntactically meaningful (and those that are, have escapes). There are no "escape sequences" for special characters like NUL bytes and newlines. Except for the examples above, to put a character in a string, just put it in the string. No fuss.

##### Comment-Block Multiline Texts

If you want to include multiline text in your code using the above kind of quotation marks, the text is likely to interfere with your code's indentation, or vice-versa. You can solve this by using comment-block text. This is a fun, strange hybrid of comments and text literals. As you may have gleaned from the rest of this guide, comments are any text from a semicolon to the end of the line. Normally, comment text is just disregarded. But if the first character after the semicolon is a quotation mark, it begins a text block over subsequent comment lines. Let's look at an example. Suppose we want to print the start of the Homebrew help string:

    Example usage:
      brew [info | home | options ] [FORMULA...]
      brew install FORMULA...

Instead of writing

        Stdout: puts "Example usage:
      brew [info | home | options ] [FORMULA...]
      brew install FORMULA..."

you can use comment-block text to make the context more clear:

        Stdout: puts ;"Example usage:
                     ;   brew [info | home | options ] [FORMULA...]
                     ;   brew install FORMULA..."

All whitespace up to the semicolon, including the semicolon, is trimmed off of the string. (An additional single space after the semicolon is trimmed off, to make the alignment consistent.)

It's worth noting that, while this looks like a regular magic comment that holds a string, it is in fact not a real comment. You can't put more code before the semicolons (that wouldn't make sense anyhow) and after the final quote mark, you can include more non-commented code on the same line (though you ought not use anything but an end-of-declaration period).

##### Special Character Functions; or how, Okay, Sometimes You Actually Want Escape Characters

Sometimes you want to represent a number of invisible characters that, if included literally, would make the surrounding code gross-looking, even with indent-eased comment-block text. For instance, say we want to replace Windows-style line endings with Unix-style ones. This code objectively sucks:

    headers: replace "
    " with "
    ".

It replaces CRLF sequences with just LFs. And you can totally tell that by glancing at it, right? There are a number of invisible characters, defined in variables such that you can refer to them in any scope. The relevant ones in this case are `Cr` for a carriage return and `Lf` for a line-feed. (In C, these are `\r` and `\n`, respectively.) So instead of the ambiguous, awful example above, you could simply interpolate them:

    headers: replace "(Cr)(Lf)" with "(Lf)".





#### Array and Dictionary Literals

Array and Dictionary Literals are enclosed in `{`curly brackets`}`.

In Arrays, elements are separated by whitespace: `{ "fish" "dog" "fankhauser" }`. Dictionaries consist of key-value pairs, separated also by whitespace. A rocket, `->` or `=>`, pairs the key to its value. The pipe rocket `->` associates by value:

    { "food" -> "fish"  "pet" -> "dog"  "hooman" -> "fankhauser" }

or

    fid => "food".  pid => "pet".  hid => "hooman".
    { fid -> "fish"  pid -> "dog"  hid -> "fankhauser" }

The terms `fid`, `pid`, and `hid` are evaluated; their string values `"food"`, `"pet"`, and `"hooman"` are used as keys in the dictionary. It's quite common to use symbols instead of strings in dictionaries, though:

    { :food -> "fish"  :pet -> "dog"  :hooman -> "fankhauser" }

If you're doing this, you can use the hash-rocket instead. See if you can spot what's going on here:

    { food => "fish"  pet => "dog"  hooman => "fankhauser" }

Where the pipe-rocket `->` associates by value—it *evaluates* what's on its left-hand side—the hash-rocket `=>` associates by symbol. In a dictionary, it takes the *bare word* to its left and interprets it as a symbol. This matches the behavior of the hash-rocket for variable declarations inside a function: It takes the bare word to the left as a variable name for the association.



#### Function Literals

We've talked about function literals before. They're just code wrapped in `[` square brackets `]`.
