How To Read Source Code
=======================

Aria Stewart

@aredridel

^ I work on the krakenjs team at PayPal. I work on internationalization and localization.

^ More specifically, what I do is wrangle the increasingly complicated ways that strings are concatenated in our web apps.

^ So why do we concatenate strings in complicated ways instead of simple ones?

^ Different responsibilities. I'll come back to this later.

---

I almost didn't give this talk.

^ Like there's any programmers out there who don't read source code.

^ Then I met a bunch of programmers who don't read source code. And I talked to some more who wouldn't read anything but the examples. And I met a _lot_ of beginning programmers who have a hard time figuring out where to start.

^ When they were maintaining software, they'd do almost anything to avoid reading the source code.

---

What do we mean when we say "reading source code"?
--------------------------------------------------

Reading for comprehension

Reading to find bugs

Reading to find interactions

Reading to review

Reading to see the interface

Reading to learn!

---

Reading isn't linear
--------------------

^ We think we can read source code like a book. Crack the introduction or README, then read through from chapter one to chapter two, on toward the conclusion.

^ It's not like that. We can't even prove that a great many programs have conclusions.

^ We skip back and forth from chapter to chapter, module to module. We can read the module straight through but we won't have the definitions of things from other modules. We can read in execution order, but we won't know where we're going more than one call site down.

-----

Reading Order
-------------

Do you start at the entry point of a package?

How about in a browser?

Maybe find the biggest source code file and read that first.

Try setting a breakpoint early and tracing down through functions in a debugger.

Try setting a breakpoint deep in the code, and reading each function in the call stack.

----

Kinds of source code

* Javascript
* C++
* ES6
* Coffee Script
* Shell Script

^ Who reads any of these kinds of source code?

---

* SNOBOL
* APL
* Befunge
* Forth
* Perl
* Other people's javascript

^ How about these?

---

Another way to think of kinds of source code

* Glue
* Interface-defining
* Implementation
* Algorithmic
* Configuring
* Tasking

^ There's a lot of study of algorithmic source code out there because that's what academia produces: algorithms, the little, often mathematical ways of doing things. All the others are a lot more nebulous but I think a lot more interesting. They're certainly the bulk of what most people write when they're programming computers.

^ Of course, sometimes, something does more than one of these things. Many times, in fact. Figuring out what it's trying to be can be one of the first tasks.

---

What's 'Glue'?
--------------

^ Not all the interfaces we want to use play nice together. Glue is the plumbing connecting things together. Middleware, promises vs callbacks binding code, inflating arguments into objects or breaking objects apart. All that's glue.

---

How do you read glue?
---------------------

^ We're looking for how two interfaces are shaped differently, and what's common between them.

---

This is from Ben Drucker's `stream-to-promise`

```javascript
internals.writable = function (stream) {
  return new Promise(function (resolve, reject) {
    stream.once('finish', resolve);
    stream.once('error', reject);
  });
};
```

^ The two interfaces we're talking about here are node streams and promises.

^ The things they have in common are that they do work until they finish, in node with an event, in promises by calling the resolution functions.

^ Now the thing you can notice while you're reading is that promises really can only be resolved once, but streams can actually emit the same event multiple times. They don't usually, but as programmers we usually know the difference between can't and shouldn't.

---

More glue!

```javascript
var record = {
    name: (input.name || '').trim(),
    age: isNaN(Number(input.age)) ? null : Number(input.age),
    email: validateEmail(input.email.trim())
}
```

^ More things to think about in glue: How are errors handled?

^ It's worth asking if any of these things throw? Do they delete bad values? Maybe they coerce them into being okay-enough values? Are these the right choices for the context they're operating in? Are conversions lossy?

---

What's interface-defining code?
-------------------------------

^ You know how you have some modules that are really only used one or two places, they're kinda internal, and you hope nobody looks at them too hard?

^ Interface-defining is the opposite of that. It's the hard boundaries of your module.

---

From node's `events.js`

```javascript
exports.usingDomains = false;

function EventEmitter() { }
exports.EventEmitter = EventEmitter;

EventEmitter.prototype.setMaxListeners = function setMaxListeners(n) { };
EventEmitter.prototype.emit = function emit(type) { };
EventEmitter.prototype.addListener = function addListener(type, listener) { };
EventEmitter.prototype.on = EventEmitter.prototype.addListener;
EventEmitter.prototype.once = function once(type, listener) { };
EventEmitter.prototype.removeListener = function removeListener(type, listener) { };
EventEmitter.prototype.removeAllListeners = function removeAllListeners(type) {};
EventEmitter.prototype.listeners = function listeners(type) { };
EventEmitter.listenerCount = function(emitter, type) { };
```

^ We're defining the interface for `EventEmitter` here.

^ Some things to ask here are is this complete?  What guarantees does this make? Are internal details exposed?

^ If you have strong interface contracts, this is where you should expect to find them.

^ Like glue code, look for how errors are handled and exposed. Is that consistent? Does it distinguish errors due to internal inconsistency from errors because the user of the interface didn't understand it?

---

Implementation
--------------

---

```javascript
  startRouting: function() {
    this.router = this.router || this.constructor.map(K);

    var router = this.router;
    var location = get(this, 'location');
    var container = this.container;
    var self = this;
    var initialURL = get(this, 'initialURL');
    var initialTransition;

    // Allow the Location class to cancel the router setup while it refreshes
    // the page
    if (get(location, 'cancelRouterSetup')) {
      return;
    }

    this._setupRouter(router, location);

    container.register('view:default', _MetamorphView);
    container.register('view:toplevel', EmberView.extend());

    location.onUpdateURL(function(url) {
      self.handleURL(url);
    });

    if (typeof initialURL === "undefined") {
      initialURL = location.getURL();
    }
    initialTransition = this.handleURL(initialURL);
    if (initialTransition && initialTransition.error) {
      throw initialTransition.error;
    }
  },
```

From `Ember.Router`

^ This is what that always needs more documentation about why and not so much about what.

^ Things to look for when reading here are how it fits into the larger whole.

^ What's coming from the public interface to this module? What needs validation? Are these variables going to contain what we think they contain? What other parts does this touch? What's the risk to changing it when adding new features? What might break? Do we have tests for that?

^ What's the lifetime of these variables? (this is an easy one: This looks really well designed and doesn't store needless state with a long lifetime.) Though maybe we should look at `_setupRouter`...

---

Process entailment
------------------

Or, looking forward: _How much is required to use this thing correctly?_

and backward: _If we're here, what got us to this point?_

^ It's pretty easy to see what functions call what other functions. However, the reverse is more interesting! So one of the things you can look for when reading source code, and in particular the implementation bits is look at what things have to happen to set up the state the process needs to be in to begin.

^ Is that state explicit, passed in via parameters? Is it assumed to be there, as an instance variable or property? Is there a single path to get there, with an obvious place that state is set up? Or is it diffuse?

---

Algorithms
-----------

^ So algorithmic code is a kind of special case of implementation code. It's not so exposed to the outside world, it's the meat of a program. Quite often it's business logic or the core processes of the software.

```javascript
function Grammar(rules) {
  // Processing The Grammar
  //
  // Here we begin defining a grammar given the raw rules, terminal
  // symbols, and symbolic references to rules
  //
  // The input is a list of rules.
  //
  // The input grammar is amended with a final rule, the 'accept' rule,
  // which if it spans the parse chart, means the entire grammar was
  // accepted. This is needed in the case of a nulling start symbol.
  rules.push(Rule('_accept', [Ref('start')]));
  rules.acceptRule = rules.length - 1;
```

This bit is from a parser engine I've been working on called `lotsawa`. More on the next slide.

^ It's been said a lot that good comments say why something is done or done that way, rather than what it's doing. Algorithms usually need more explanation of what's going on since if they were trivial, they'd probably be built into our standard library. Quite often to get good performance out of something, the exactly what-and-how matters a lot.

^ This is the stuff that those of you with CS degrees get to drool over (or maybe have traumatic memories of).

---

```javascript
  // Build a list of all the symbols used in the grammar so they can be numbered instead of referred to
  // by name, and therefore their presence can be represented by a single bit in a set.
  function censusSymbols() {
    var out = [];
    rules.forEach(function(r) {
      if (!~out.indexOf(r.name)) out.push(r.name);

      r.symbols.forEach(function(s, i) {
        var symNo = out.indexOf(s.name);
        if (!~out.indexOf(s.name)) {
          symNo = out.length;
          out.push(s.name);
        }

        r.symbols[i] = symNo;
      });

      r.sym = out.indexOf(r.name);
    });

    return out;
  }

  rules.symbols = censusSymbols();

```

Reads like a math paper, doesn't it?

^ One of the things that you usually need to see in algorithmic code is the actual data structures. This one is building a list of symbols and making sure there's no duplicates.

^ Look also for hints as to the running time of the algorithm. You can see in this part, I've got two loops. In Big-O notation, that's O(n * m), then you can see that there's an `indexOf` inside that. That's another loop in Javascript, so that actually adds another factor to the running time. (twice -- looks like I could make this more optimal by re-using one of the values here) Good thing this isn't the main part of the algorithm!

Configuration
-------------

^ The line between source code and configuration file is super thin. There's a constant tension between having a configuration be expressive and readable and direct.

----

```javascript
app.configure('production', 'staging', function() {
  app.enable('emails');
});

app.configure('test', function() {
  app.disable('emails');
});
```

An example using Javascript for configuration.

^ What we can run into here is combinatorial explosion of options. How many environments do we configure? Then, how many things do we configure for a specific instance of that environment. It's really easy to go overboard and end up with all the possible permutations, and to have bugs that only show up in one of them. Keeping an eye out for how many degrees of freedom the configuration allows is super useful.

---

```javascript
    "express": {
        "env": "", // NOTE: `env` is managed by the framework. This value will be overwritten.
        "x-powered-by": false,
        "views": "path:./views",
        "mountpath": "/"
    },

    "middleware": {

        "compress": {
            "enabled": false,
            "priority": 10,
            "module": "compression"
        },

        "favicon": {
            "enabled": false,
            "priority": 30,
            "module": {
                "name": "serve-favicon",
                "arguments": [ "resolve:kraken-js/public/favicon.ico" ]
            }
        },

```

A bit of `kraken` config file.

^ Kraken took a 'low power language' approach to configuration and chose JSON. A little more "configuration" and a little less "source code". One of the goals was keeping that combinatorial explosion under control. There's a reason a lot of tools use simple key-value pairs or ini-style files for configuration, even though they're not terribly expressive.

---

Configuration source code has some interesting and unique constraints that are worth looking for.

* Lifetime
* Machine writability
* Responsible people vary a lot more
* Have to fit in weird places like environment variables
* Often store security-sensitive information

---

Tasking
-------

---

What do charging 50 credit cards, building a complex piece of software with a compiler and build tools, and sending 100 emails have in common?

^ Transactionality. Often, some piece of the system needs to happen exactly once, and not at all if there's an error. A compiler that leaves bad build products around is a great source of bugs. Double charging customers is bad. Flooding someone's inbox because of a retry cycle is terrible.

^ Resumabilty. A need to continue where they left off given the state of the system.

^ Sequentiality. If they're not strictly linear processes, there's usually a very directed flow through the code. Loops tend to be big ones, around the whole shebang.

---

Reading Messy Code
------------------

---

So how do you deal with this?

```javascript
      DuplexCombination.prototype.on = function(ev, fn) {
    switch (ev) {
  case 'data':
  case 'end':
  case 'readable':
this.reader.on(ev, fn);
return this
  case 'drain':
  case 'finish':
this.writer.on(ev, fn);
return this
  default:
return Duplex.prototype.on.call(this, ev, fn);
    }
      };
```

You are seeing that right. That's reverse indendation. Blame Isaac.

---

Rose tinted glasses!
--------------------

`standard -F dc.js`

```javascript
DuplexCombination.prototype.on = function (ev, fn) {
  switch (ev) {
    case 'data':
    case 'end':
    case 'readable':
      this.reader.on(ev, fn)
      return this
    case 'drain':
    case 'finish':
      this.writer.on(ev, fn)
      return this
    default:
      return Duplex.prototype.on.call(this, ev, fn)
  }
}
```

It's okay to use tools while reading! 

---

How do you read this?

```
(function(t,e){if(typeof define==="function"&&define.amd){define(["underscore","
jquery","exports"],function(i,r,s){t.Backbone=e(t,s,i,r)})}else if(typeof export
s!=="undefined"){var i=require("underscore");e(t,exports,i)}else{t.Backbone=e(t,
{},t._,t.jQuery||t.Zepto||t.ender||t.$)}})(this,function(t,e,i,r){var s=t.Backbo
ne;var n=[];var a=n.push;var o=n.slice;var h=n.splice;e.VERSION="1.1.2";e.$=r;e.
noConflict=function(){t.Backbone=s;return this};e.emulateHTTP=false;e.emulateJSO
N=false;var u=e.Events={on:function(t,e,i){if(!c(this,"on",t,[e,i])||!e)return t
his;this._events||(this._events={});var r=this._events[t]||(this._events[t]=[]);
r.push({callback:e,context:i,ctx:i||this});return this},once:function(t,e,r){if(
!c(this,"once",t,[e,r])||!e)return this;var s=this;var n=i.once(function(){s.off
(t,n);e.apply(this,arguments)});n._callback=e;return this.on(t,n,r)},off:functio
n(t,e,r){var s,n,a,o,h,u,l,f;if(!this._events||!c(this,"off",t,[e,r]))return thi
s;if(!t&&!e&&!r){this._events=void 0;return this}o=t?[t]:i.keys(this._events);fo
r(h=0,u=o.length;h<u;h++){t=o[h];if(a=this._events[t]){this._events[t]=s=[];if(e
||r){for(l=0,f=a.length;l<f;l++){n=a[l];if(e&&e!==n.callback&&e!==n.callback._ca
llback||r&&r!==n.context){s.push(n)}}}if(!s.length)delete this._events[t]}}retur
n this},trigger:function(t){if(!this._events)return this;var e=o.call(arguments,
1);if(!c(this,"trigger",t,e))return this;var i=this._events[t];var r=this._event
s.all;if(i)f(i,e);if(r)f(r,arguments);return this},stopListening:function(t,e,r)
{var s=this._listeningTo;if(!s)return this;var n=!e&&!r;if(!r&&typeof e==="objec
```

---

`uglifyjs -b < backbone-min.js`

```javascript
(function(t, e) {
    if (typeof define === "function" && define.amd) {
        define([ "underscore", "jquery", "exports" ], function(i, r, s) {
            t.Backbone = e(t, s, i, r);
        });
    } else if (typeof exports !== "undefined") {
        var i = require("underscore");
        e(t, exports, i);
    } else {
        t.Backbone = e(t, {}, t._, t.jQuery || t.Zepto || t.ender || t.$);
    }
})(this, function(t, e, i, r) {
    var s = t.Backbone;
    var n = [];
    var a = n.push;
    var o = n.slice;
    var h = n.splice;
    e.VERSION = "1.1.2";
    e.$ = r;
    e.noConflict = function() {
```

---

Human Parts
-----------

Or: _guessing intent is dangerous but you learn so much_

----

There's a lot of tricks for figuring out what the author of something meant.

----

Look for guards and coercions
-------------------------------------------

```javascript
if (typeof arg != 'number') throw new TypeError("arg must be a number");
```

Looks like the domain of whatever function we're in is 'numbers'.

```javascript
arg = Number(arg)
```

Coerce things to numeric. Same domain as above, but doesn't reject errors via exceptions. There might be `NaN`s though. Probably smart to read and check if there's comparisons that will be false against those.

----

Look for defaults
-----------------

```javascript
arg = arg || {}
```

Default to an empty object.

```javascript
arg = (arg == null ? true : arg)
```

Default to true only if a value wasn't explicitly passed.

---

Look for layers
---------------

`req` and `res` from Express are tied to the web; how deep do they go?

Is there an interface boundary you can find?

---

Look for tracing
----------------

Are there inspection points?

Debug logs?

Do those form a complete narrative? Or are they ad-hoc leftovers from the last few bugs?

---

Look for reflexivity
--------------------

Are identifiers being dynamically generated?

Is there `eval`? Metaprogramming? New function creation?

`func.toString()` is your friend!

----

Look at lifetimes
-----------------

* Who initializes this?
* When does it change?
* Who changes it?
* Is this information somewhere else in the mean time?
* Is it ever inconsistent?

^ Somewhere, someone typed the value you see into a keyboard, generated it from a random number generator, or computed it and saved it.

^ Somewhere else, some time else, that value will affect some human or humans. Who are these people?

^ What or who chooses who they are?

^ Is that value ever going to change?

^ Who changes it?

----


Look for hidden state machines
------------------------------

^ Sometimes boolean variables get used together as a decomposed state machine

^ For example, the variables `isReadied` and `isFinished` might show a state machine like so:

```
var isReadied = false;
var isFinished = false;
```

Or

`START -> READY -> FINISHED`

----

Look for hidden state machines
------------------------------

```
isReadied | isFinished | state
----------|------------|------------
false     | false      | START
false     | true       | invalid
true      | false      | READY
true      | true       | FINISHED
```

^ Note that they can also express the state `!isReadied && isFinished` -- which might be an interesting source of bugs, if something can end up at the finished state without first being ready.

------

Look for composition and inheritance
------------------------------------

Is this made of parts I can recognize? Do those parts have names?

------

Look for common operations
--------------------------

`map`.

`reduce`.

cross-join.

-----

It's time to go read some source code.

Enjoy!
