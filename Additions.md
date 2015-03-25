Look at lifetimes
-----------------

* Who initializes this?
* When does it change?
* Who changes it?
* Is this information somewhere else in the mean time?
* Is it ever inconsistent?

----

Somewhere, someone typed the value you see into a keyboard, generated it from a random number generator, or computed it and saved it.

Somewhere else, some time else, that value will affect some human or humans. Who are these people?

What or who chooses who they are?

Is that value ever going to change?

Who changes it?

Now look at your code, and back to me, now back to your code, now back at me. Your code itself has turned into values, typed into a keyboard, by a human, for a human.

----


Look for hidden state machines
------------------------------

Sometimes boolean variables get used together as a decomposed state machine

For example, the variables `isReadied` and `isFinished` might show a state machine like so:

`START -> READY -> FINISHED`

isReadied | isFinished | state
----------|------------|------------
`false`   | `false`    | START
`false`   | `true`     | _invalid_ 
`true`    | `false`    | READY
`true`    | `true`     | FINISHED

^ Note that they can also express the state `!isReadied && isFinished` -- which might be an interesting source of bugs, if something can end up at the finished state without first being ready.

------


Say something about classes

Say something about access control

-----

Language power
--------------

Pretend you don't know what language you're reading at all. Your job is to figure out what features the language written has.

Does it have variables? Can they be reassigned?
Mutable values?
Does it have functions?
Is there some sort of inheritance going on?
Mixins? Merged values or unions of some sort?

What sort of types can you spot?
Strings? Numbers? Arrays? Dictionaries?

How many named things are there?

Humans can't really track more relationships than about Dunbar's number. That's about 150 or so, with a lot of give or take.

-----

Reading isn't linear.
---------------------

^ We think we can read source code like a book. Crack the introduction or README, then read through from chapter one to chapter two, on toward the conclusion.

^ It's not like that. We can't even prove that a great many programs have conclusions.

^ We skip back and forth from chapter to chapter, module to module. We can read the module straight through but we won't have the definitions of things from other modules. We can read in execution order, but we won't know where we're going more than one call site down.

^ Even parsers reading the source code don't work quite linearly. Either they're LL parsers, and they chug along, roughly aware of what they're expecting to be able to find at any given point, but at they don't really know if the details further on will make sense with the whole until the end, or they're LR parsers, and they see all the details, but don't really know what the parts make until the end, when they finish collapsing everything they've read into a tree and can make it into actual structure.


Reading Order
-------------

Do start at the entry point of a package? 

how about in a browser?

Try setting a breakpoint early and tracing down through functions in a debugger.

Try setting a breakpoint deep in the code, and reading each function in the call stack.

Examine a callback by logging `callback.toString()`. Search for where that's defined.


-----

