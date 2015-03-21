How To Read Source Code
=======================

---

What do we mean when we say "reading source code"?

Reading for comprehension

Reading to find bugs

Reading for interactions

---

Kinds of source code

* C++
* Javascript
* ES6
* Coffee Script
* Shell Script
* SNOBOL

---

Another way to think of kinds of source code

* Algorithmic
* Interface-defining
* Implementation
* Glue
* Configuring
* Tasking

^ There's a lot of study of algorithmic source code out there because that's what academia produces: algorithms, the little, often mathematical ways of doing things. All the others are a lot more nebulous but I think a lot more interesting. They're certainly the bulk of what most people write when they're programming computers.

^ Of course, sometimes, a bit of code does more than one of these things.

---

What's 'Glue'?
--------------

^ Not all the interfaces we want to use play nice together. Glue is the plumbing connecting things together. Middleware, promises vs callbacks binding code, inflating arguments into objects or breaking objects apart. All that's glue.

---

How do you read glue?

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

^ Interface-defining is the opposite of that.

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

^ Like glue code, look for how errors are handled and exposed. Is that consistent?

---

Have you ever looked at deeply algorithmic code?

```javascript
function Grammar(rules) {
  // Processing The Grammar
  // ======================
  //
  // Here we begin defining a grammar given the raw rules, terminal
  // symbols, and symbolic references to rules
  //
  // The input is a list of rules.
  //
  // Add the accept rule
  // -------------------
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

---

What part's the implementation then?

^ So algorithmic code is a kind of special case of implementation code. It's not so exposed to the outside world, it's the meat of a program. Quite often it's business logic or the core processes of the software.

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

^ It's pretty easy to see what functions call what other functions. However, the reverse is more interesting! So one of the things you can look for when reading source code, and in particular the implementation bits is look at what things have to happen or be called to set up the state the process needs to be in to start.

^ Is that state explicit, passed in via parameters? Is it assumed to be there, as an instance variable or property? Is there a single path to get there, with an obvious place that state is set up? Or is it diffuse?

---

Configuration
-------------

^ The line between source code and configuration file is super thin. There's a constant tension between having a configuration be expressive and readable and direct.

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

A `kraken` config file.

^ Kraken took a 'low power language' approach to configuration and chose json. 

----

Unfinished:

Here's a trick for reading what the author is letting through:

Ask what a filter doesn't pass.

if (foo === null) {
}

"null is special, everything else is treated differently."

"even undefined. And false."

Maybe the author meant "== null"
