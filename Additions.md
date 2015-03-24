Look at lifetimes
-----------------

* Who initializes this?
* When does it change?
* Who changes it?
* Is this information somewhere else in the mean time?
* Is it ever inconsistent?

Look for hidden state machines
------------------------------

Sometimes boolean variables get used together as a decomposed state machine

For example, the variables `isReadied` and `isFinished` might show a state machine like so:

`INITIALIZED -> READY -> FINISHED`

| isReadied | isFinished | state       |
| `false`   | `false`    | INITIALIZED |
| `false`   | `true`     | _invalid_   |
| `true`    | `false`    | READY       |
| `true`    | `true`     | FINISHED    |

Note that they can also express the state `!isReadied && isFinished` -- which might be an interesting source of bugs, if something can end up at the finished state without first being ready.

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
