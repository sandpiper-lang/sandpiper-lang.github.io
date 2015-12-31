#### Declaring New Classes

A class is a specific kind of type, but it's the most tangible kind, so we'll start there. For this example we'll build a simple linked list.

    LinkedList => class
        is-an Object.

        there-is element.
        there-is next-node (LinkedList?).
    
    end.

The class-literal is `class...end`, and the class it creates is bound to the `LinkedList` variable. There is no global namespace in Sandpiper, not even for class names. If we put other code around this class in the same file, that code can refer to it by name because the variable `LinkedList` is in the same lexical scope. If we refer to a “LinkedList” from another file (another scope), we'll encounter an error; we'd have to `import` that file to get access to the names it declares. (We only get access to names beginning with uppercase letters.)

The `is-an Object.` declaration is another keyword declaration, that defines the class's formal inheritance. We also say that for each instance, `there-is` an `element` and a `next-node`. The `next-node` is another `LinkedList` or `Nil`, and the `element` is implicitly type `Any`.

We need to give it an initializer to fill in those variables.

    LinkedList => class
        is-an Object.

        there-is element.
        there-is next-node (LinkedList?).

        init with-element e (Any) next l (LinkedList?) => [
            element => e.
            next-node => l.

            self.
        ].
    
    end.

This is a fairly standard initializer, declared with the `init` keyword. It returns the object itself (it could return anything to create a different kind of object); functions in Sandpiper always return the value of their final declaration. The function becomes a private instance method, paired with a public class method by the same name, `with-element()next()`, that opaquely creates an instance and calls the private initializer. Now we can do

    LinkedList: with-element “hoge” next Nil

which creates our first list. As a terrible example, we can build a three-element list, in the way you'd expect:

    hoge => LinkedList: with-element “hoge” next fuga.
    fuga => LinkedList: with-element “fuga” next piyo.
    piyo => LinkedList: with-element “piyo” next Nil.

Obviously is this is awfully verbose. We'll clean it up later. You should also notice that the declarations need not come in a particular order—they're considered to be simultaneously true, so as long as there are no circular dependencies, you can order them however it makes the most sense to you.

The next thing we want is to be able to inspect the list. Instance variables are always private, so we'll add two methods, which, for reasons that will become clear, we'll call `head` and `rest`:

    LinkedList => class
        is-an Object.

        there-is element.
        there-is next-node (LinkedList?).

        init with-element e (Any) next l (LinkedList?) => [
            element => e.
            next-node => l.

            self.
        ].
    
        instance-func head => [ element ].
        instance-func rest => [ next-node ].

    end.

Methods are formally called "bound functions" in Sandpiper—in this case, bound to instances of LinkedList. The keyword declaration `instance-func <declaration>` is aliased `- <declaration>`, for simplicity and clarity; so we could instead say:

        - head => [ element ].
        - rest => [ next-node ].

We can also have class methods, or "type functions," which are functions bound to the type itself. The declaration looks exactly the same, but uses the keyword `type-func`, aliased `+`.

Before going on, let's add methods for adding elements at the head (`push`) and tail (`tack`). Being immutable, we can't actually change an existing list. This is good, and by design, as it frees us from the concerns of making private copies and threadsafing our data. Instead, when we want to modify a structure, we just return a new, altered version.

Push is easy:

        - push e (Any) => [
            instance-type: with-element e next self.
        ].

`self` is the object we're writing as. `instance-type` is the formal type of that object—it's probably `LinkedList`, but could be a subclass, if we define one later. We simply use our constructor to make a new node, with the function argument `e` as the item and `self` as the rest of the list. The return value is the new head node, the new list.

Tack is more interesting:

        - tack e (Any) => [
            new-child => next-node: when LinkedList use [ next-node: tack e ]
                                    otherwise [ instance-type: with-element e next Nil ].
        
            instance-type: with-element element next new-child.
        
        ].

We're building up the new list by creating a node for the new element `e`, and then referring to it with a brand-new chain of nodes that duplicate the existing list, all the way back up to the root (head) node. We're asking our own `next-node` object whether it's a LinkedList itself, or Nil. If it's a sub-list, we're delegating creating the rest of the duplicate sub-list to that node, using a recursive call. If instead it's Nil, then `self` is the last node, so `next-node` is going to be a new one-element list containing `e`. Finally, whatever that sub-list is, we create a duplicate copy of `self` with the same `element` we had before, but with the new sub-list in place of the old `next-node`.

It should be clear that for singly-linked lists, it's natural and efficient to prepend an item, where the entire existing list can be re-used; but much less efficient to append an item. (If we're using a tree, it's not terribly inefficient to add a leaf, where the tree depth is more like *lg(n),* and only that many elements need to be recreated.)



#### Using Type Inheritance to Make Great Things

Right now our humble linked list provides no really easy traversal mechanism. We could write one, but, as it turns out, we don't need to. We just need to declare that it `is FiniteSequence.`

    LinkedList => class
        is-an Object.
        is FiniteSequence.

        there-is element.
        there-is next-node (LinkedList?).

        init with-element e (Any) next l (LinkedList?) => [
            element => e.
            next-node => l.

            self.
        ].
    
        - head => [ element ].
        - rest => [ next-node ].
        ...

    end.

A type declares methods and can implement the ones it wants to. A class is a type from which actual objects can be made. Particularly, a class is a type that can have instance variables. When a class formally inherits from another class using `is-a` or `is-an`, it works basically like subclassing in any other objective language—including that it can only formally inherit from one class. In addition, it can inherit from any number of non-class types using the `is` declaration.

A non-class type can declare methods that subclasses promise to implement, like interfaces in Java; and it can declare and implement methods that subclasses and subtypes inherit, much like imported modules in Ruby.

A good Sandpiper type should have a narrowly-focused collection of methods that are all strongly related to the purpose of the type; and it should import other types, as they make sense, to gain their functionality. There is no performance penalty for having lots of small types, as they make sense. FiniteSequence, for instance, defines a handful of really useful methods, like `tail`, `reversed`, and `shuffled`; but it imports both Sequence and FiniteCollection, which each import Collection. A Collection provides the functionality of an object that generically contains some items, like `map`, `reduce`, and `empty?`. A FiniteCollection adds more specific functionality like `count`, `sample`, and `each()!`, that don't work on an infinite collection. A Sequence adds methods like `first()`, `with-indices`, and `at()`, which only make sense for contents that have an ordering. And a FiniteSequence is all of these things, adding functionality based on the end of a sequence like `reversed` and `shuffled`. By importing FiniteSequence, we get all of this in our LinkedList for free.

Well, it's almost free. There are two methods on Collection that are declared but not implemented—`head` and `rest`—that we need to implement. `head` is expected to return any element in the collection (the first in a sequence, or Nil if it's empty), and `rest` is expected to return a collection of the same type that's almost a copy of the original, except that omits the element that would be returned by `head`. Fortunately, we have written these already!

The other methods in Collection, and its subtypes, will use our implementation of `head` and `rest` as the basis for enumerating our list.

Suppose we have the following list (built back-to-front):

    doge => LinkedList: with-element “wagger” next Nil:
        push “rump”:
        push “belly”:
        push “lungs”:
        push “shoulders”:
        push “neck”:
        push “head”:
        push “sniffer”.

Having imported FiniteSequence (which is also called List), we can do:

    doge: reversed.  ;; starts with wagger and goes up to sniffer
    doge: shuffled.  ;; you crazy mixed up bunch of pup
    doge: filter [ &: ends-with “er” ? ].  ;; wagger and sniffer
    doge: count.  ;; 8
    doge: sample.  ;; a random element, like neck
    doge: at 2.  ;; belly

We'll take a look at that `filter()` clause in the next section.
