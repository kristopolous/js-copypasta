/*
# Sometimes you just want one isolated function.

Sometimes I find myself writing the same helper functions in situations where I don't want to pull in a library, just put in some really small function.

This is a *different* kind of toolkit ... there's no source to include.

Instead, there's a description of each function, what it does, and then a multi and one line version of it for you to put into your code.

> Note 1: The one-line version is intentionally *not* "minified" or obfuscated ... it's intended to be a readable variation of the multi-line that fits in one line.

> Note 2: This README is valid javascript.  Really, if you want to you can just include it as a script ... that's why the "stray" comments are peppered around.

**Every helper is self-contained. No function depends on any other function in the 'library.'**

Let's get started

  * <a href='#addtrigger'>addTrigger</a> - a module that can add ".ready" to an object
  * <a href='#when'>when</a> - runs a block of code when a global is defined
  * <a href='#req'>req</a> - loads a script and then runs a when 
  * <a href='#multi'>multi</a> - runs multiple functions through a single assignment
  * <a href='#chain'>chain</a> - cascades the return values of multiple functions with a single assignment
  * <a href='#once'>once</a> - makes sure that a function is run only once

<a name='addtrigger'></a>
# addTrigger

## Purpose
This is similar to a `$(document).ready` in jQuery.  This author has seen many DIY implementations with subtle timing bugs.

For instance, here is one recently (Feb 2016) found in Volusion:

      function volusionDocReady(callback) {
        volusionDocReadyMethods.push(callback);
      }

      $(function() {
        $.each(volusionDocReadyMethods, function (index, volusionDocReadyMethod) {
          volusionDocReadyMethod();
        });
      });

      return {
        ready: volusionDocReady
      };

This code has a crucial error that leads to things sometimes not working.  

To understand the bug, let's look at a usage of this library where this invocation exists (called CB henceforth):

      $(document).ready(function(){
        volusion.ready(function () { <<-- CB
          ...
        });
      });

To understand the bug you have to understand the timelines
of possible execution.

Let's state what this code does:

 * CB is always added to `volusionDocReadyMethods`
 * The members of `volusionDocReadyMethods` always get called
 * BUT CB is NOT always added BEFORE the methods get called.

Therefore, CB can register AFTER the methods are called, and
thus is never run.

This is where honest and well-intended implementations of `.ready()` can break as seen for a client of mine.

Some could blame the implementer of CB as the culprit here.

However, wouldn't it be better if the `.ready()` implementation was robust enough to not take issue with this? Fragile foundations are hardly worth using at all.  Let's do better.

## Example Usage

`addTrigger` can be placed upon existing singletons (or global objects -- however you want to call them):

    addTrigger('ready', object);

This can be composed pretty easily to our `.ready()` implementation above.

    function addReady(object) { return addTrigger('ready', object); }

The genericity of this, allows multiple triggers to exist.  So you could have `.onLogin` and `.ready` or something else.

When you want to execute the code, call the named function without a callback function argument.

For instance, in our above example we could have turned the first block of code into this:

    addTrigger('ready', volusion);
    $(volusion.ready);

Then

    volusion.ready( ... );

While avoiding the timing bugs.

Here it is, with no dependencies or demands, frameworks or funk in a single line of code.

## Implementation

**Multi-line**
*/

    function addTrigger(name, object) {
      var stack = [];

      object[name] = function(arg) {
        if(arg && arg.call && arg.apply) { 
          return stack === null ? arg() : stack.push(arg);
        } 
        while(stack.length) {
          stack.shift()();
        }
        stack = null;
      }
    }
/*
**Single-line**

    function addTrigger(name, object){ var stack=[]; object[name]=function(arg){ if(arg && arg.call && arg.apply){ return stack===null ? arg():stack.push(arg); } while(stack.length){ stack.shift()(); } stack=null; } }

<a name='when'></a>
# when

## Purpose 
Similar to an [AMD Require](http://requirejs.org/docs/whyamd.html) except designed to delay execution until a library that is included through say an HTML `<script>` tag is loaded.

## Example Usage

This will wait until JQuery is loaded ( presuming it's being included somewhere else on the page ):

Sometimes, given a not-so-hotly designed system, this can lead to a race-condition.  

    <script>
    $(function() { ... });
    </script>

You could either be an "architect" and spend endless hours trying to fix it, or use when:

    <script>
    when('$').run(function() { ... });
    </script>

## Implementation

**Multi-line**

*/

    function when(lib) {
      var _cb, _ival = setInterval(function(){

        if(self[lib]) {
          _cb();
          clearInterval(_ival);
        }
      }, 20);

      return {
        run: function(cb) { _cb = cb; }
      }
    }
/*
**Single-line**

    function when(lib){ var _cb, _ival=setInterval(function(){ if(self[lib]) { _cb(); clearInterval(_ival); } }, 20); return{ run: function(cb) { _cb=cb; } } }

 
<a name='req'></a>
# req

## Purpose
Similar to an amd require, this is a really light library that will load some js, given that it's not loaded before.

## Example Usage

Pretend you have a debug library you want to pull in before running some code.  You know that when it's loaded
it will define `dbg` in the global namespace.  Pretend the code is at `//example.com/dbg.js`.

    req('//example.com/dbg.js', 'dbg', function() { ... });

There are very cathedral ways of doing this ... but let's not do that.

Also note that this method won't load a script if the object you are looking for is defined.  If the functionality was
brought in elsewhere, this thing isn't stupid enough that it needs its own copy to consider it "loaded".

## Implementation

**Multi-line**

*/

    function req(url, obj, cb) {
      req[url] = self[obj] || req[url] || (document.body.appendChild(document.createElement('script')).src = url);

      var _ival = setInterval(function(){
        if(self[obj]) {
          cb(obj);
          clearInterval(_ival);
        }
      }, 20);
    }
/*
**Single-line**

    function req(url, obj, cb){ req[url]=self[obj] || req[url] || (document.body.appendChild(document.createElement('script')).src=url); var _ival=setInterval(function() { if(self[obj]){ cb(obj); clearInterval(_ival); } }, 20); }


<a name='multi'></a>
# multi

## Purpose
Some libraries don't allow an easy way to specify an `Array` of callbacks as opposed to just one.  `multi` allows any number of functions to be glued together and executed in sequence

## Example Usage

Pretend you have some old crufty code that wants you to specify things like this:

    flashObject.onclick = fn0;

But you also want it to run fn1, fn2, and fn3.

With multi you can do this like so:

    flashObject.onclick = multi(
      fn0, fn1, fn2, fn3);

The `arguments` and `this` pointer are of course maintained.

## Implementation

**Multi-line**
*/

    function multi() {
      var list = Array.prototype.slice.call(arguments);
      return function() {
        var args = arguments;

        list.forEach(function(cb) {
          cb.apply(this, args);
        }, this);
      };
    }
/*
**Single-line**

    function multi(){var list=Array.prototype.slice.call(arguments); return function(){var args=arguments; list.forEach( function(cb){cb.apply(this, args);}, this);} }

<a name='chain'></a>
# chain

## Purpose

Chain is like multi only the return value of each function gets passed through to the next ... this is similar to a middleware in ruby. It's worth noting that there's just a slight change from the multi() above.

## Example Usage

Pretend you needed to do some processing on something before passing it on to the next function.  For instance, say you are listening to a dom node and then want the value to be passed through to the lower function.  We'll start with a wrapper.

    function on_change(dom_node, user_defined_cb) {
      $(dom_node).change(chain(
        function() { return this.value; },
        user_defined_cb
      ));
    }

Now we'll listen on that

    on_change('#my-input', function(what) { 
      console.log("the new value of my-input is " + what);
    });

**Multi-line**
*/
    function chain() {
      var list = Array.prototype.slice.call(arguments);
      return function() {
        var args = arguments;

        list.forEach(function(cb) {
          args = cb.apply(this, args);
        }, this);
      };
    }
/*
**Single-line**

    function chain(){var list=Array.prototype.slice.call(arguments); return function(){var args=arguments; list.forEach( function(cb){args=cb.apply(this, args);}, this);} }


<a name='once'></a>
# once

## Purpose
Sometimes there's bad code that runs a function many times and you want some bandaid that makes it only run once.  There's
too many other things you'll break if you try to "refactor" the "masterpiece" in front of you.

`once()` permits that function to only run one time. It has the assumption that 

 * `fn.once` is not defined
 * The function's scope is `self[fn]`

In practice, these are generally pretty safe assumptions in instances where this functionality is necessary.


## Example Usage
Pretend you have an ajax call that doesn't have any immediately UI feedback (there's a very fancy name for this that the
front-end 'frameworks' like to use to make themselves look like they are super smart...).  

Anyway, so you have a button that the user will click like 4 or 5 times getting no UI feedback and thus posts a bunch
of times ... yeah, we've all been there.  Here's how you fix that here:

Run this:

    once("email_ten_thousand_people");

And then it will "wrap" the actual implementation in a function that makes sure it doesn't get run more than once. You'll
see in the multi-line below.  There's a [magical regex](http://stackoverflow.com/questions/2648293/javascript-get-function-name) that once *could* use to remove the requirement of the quotations, but that makes the implementation harder to understand
for future you and in violation of the principle of this library.  However, feel free to put that check in if you want.

**Multi-line**
*/

    function once(fn) {
      var _fn = self[fn];

      return self[fn] = function() {
        self[fn] = function() { return _fn; };
        return _fn = _fn.apply(this, arguments);
      };
    }

/*
**Single-line**

    function once(fn){var _fn=self[fn]; return self[fn]=function(){self[fn]=function(){return _fn;}; return _fn=_fn.apply(this, arguments);}; }
*/
