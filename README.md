# Sometimes you just want one isolated function.

Sometimes I find myself writing the same helper functions in situations where I don't want to pull in a library, just put in some really small function.

This is a *different* kind of toolkit ... there's no source to include.

Instead, there's a description of each function, what it does, and then a multi and one line version of it for you to put into your code.

> Note: The one-line version is intentionally *not* "minified" or obfuscated ... it's intended to be a readable variation of the multi-line that fits in one line.

**Every helper is self-contained. No function depends on any other function in the 'library.'**

Let's get started

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

### Multi-line

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

### Single-line

    function when(lib){ var _cb, _ival=setInterval(function(){ if(self[lib]) { _cb(); clearInterval(_ival); } }, 20); return{ run: function(cb) { _cb=cb; } } }

 
# multi

## Purpose
Some libraries don't allow an easy way to specify an `Array` of callbacks as opposed to just one.  `multi` allows any number of functions to be glued together and executed in sequence

## Example Usage

### Style 1

Pretend you have some old crufty code that wants you to specify things like this:

    flashObject.onclick = fn0;

But you also want it to run fn1, fn2, and fn3.

With multi you can do this like so:

    var fn_wrap = multi();

    fn_wrap.add(fn0).add(fn1).add(fn2).add(fn3);

    flashObject.onclick = fn_wrap;

The `arguments` and `this` pointer are of course maintained.

### Style 2

There's a short-hand style here too, given the fact that the initial list comes from the arguments
passed into the function.  This means that you can also invoke it like so:

    flashObject.onclick = multi(fn0, fn1, nf2, fn3);

And get the same outcome.

## Implementation

### Multi-line

    function multi() {
      var _list = Array.prototype.slice.call(arguments),
        _invoke = function() {
        var _args = arguments, _this = this;

        _list.forEach(function(cb) {
          cb.apply(this, _args);
        }, _this);
      }

      _invoke.add = function(cb) {
        _list.push(cb);
        return _invoke;
      }

      return _invoke;
    }

### Single-line

    function multi(){var list=Array.prototype.slice.call(arguments), invoke=function(){var _args=arguments, _this=this; _list.forEach( function(cb){cb.apply(this, _args);}, _this );}  _invoke.add=function(cb){_list.push(cb); return _invoke;}  return _invoke;}


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

### Multi-line

    function once(fn) {
      var _fn = self[fn];

      return self[fn] = function() {
        self[fn] = function() { return _fn; };
        return _fn = _fn.apply(this, arguments);
      };
    }


### Single-line

    function once(fn){var _fn=self[fn]; return self[fn]=function(){self[fn]=function(){return _fn;}; return _fn=_fn.apply(this, arguments);}; }
